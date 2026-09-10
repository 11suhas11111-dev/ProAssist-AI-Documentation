# Security Architecture — ProAssist AI

ProAssist AI implements a **defense-in-depth, least-privilege security model** designed to protect the local Windows environment, sensitive user credentials, and personal data from unauthorized physical access, malicious prompt injection, or rogue autonomous execution.

> **Disclaimer**: The security architecture described herein represents a robust engineering implementation designed for local personal assistant environments. It has not been formally certified under Common Criteria or commercial biometric evaluation schemes (e.g., FIDO Biometrics / ISO 30107-3 PAD).

---

## 1. Security Foundations & Threat Model

ProAssist AI addresses five primary threats:
1. **Unauthorized Voice Spoofing**: Impostors issuing commands via microphone when the owner is absent.
2. **Accidental / Malicious Destruction**: Commands that delete critical files, overwrite data, or shut down the machine.
3. **Prompt Injection / Jailbreaking**: Malicious inputs manipulating the LLM into generating destructive tool calls.
4. **Credential Exfiltration**: Leaking passwords, API tokens, or OAuth keys to logs, LLMs, or unauthorized processes.
5. **Rogue Agent Drift**: Multi-step workflows entering unbounded execution loops.

---

## 2. Authentication Hierarchy

The system defines three strict authentication levels (`AuthLevel` in `security/permission_manager.py`):

| AuthLevel | Numeric Value | Description | Permitted Tools |
|---|:---:|---|---|
| **`PUBLIC`** | `0` | Default state on startup or after session timeout. No voice authentication required. | Read-only hardware telemetry (`get_system_info`), time/date queries, basic conversational small talk. |
| **`AUTHENTICATED`** | `1` | Session verified via local voice biometric scan matching the enrolled owner profile. | App launching, non-destructive file operations (`copy`, `search`), contacts, tasks, notes, weather. |
| **`CONFIRMATION`** | `2` | Voice authentication PLUS explicit, interactive user approval via the PySide6 UI dialog. | Destructive actions (`delete_files`, `delete_note`, `delete_task`), system shutdown, restart, future external messaging. |

---

## 3. Voice Biometrics & Authentication Architecture

Voice authentication operates entirely on-device without cloud API dependencies (`security/voice_auth/`):

```
                       [ Incoming Audio Frame ]
                                  │
                                  ▼
                     [ SpeakerEnrollmentManager ]
                                  │
                  Is User Enrolled? (3 samples required)
                                  ├── NO  ──► Prompts user to complete enrollment
                                  └── YES
                                          │
                                          ▼
                             [ SpeakerVerifier ]
                     Extracts 128-d acoustic feature vector
                                          │
                        Cosine Similarity >= 0.70 Threshold?
                                          ├── NO
                                          │    ├── Increment failed attempt counter
                                          │    ├── If failures >= 3 -> LOCKOUT for 30 seconds
                                          │    └── Return "Access Denied"
                                          └── YES
                                               ├── Reset failure counter
                                               ├── Start 300-second session timer
                                               └── Elevate session to AUTHENTICATED
```

### Security Hardening Measures:
- **Lockout Mechanism**: After 3 consecutive failed verification attempts, authentication is locked out for 30 seconds (`lockout_seconds: 30`).
- **Session Expiry**: An authenticated session automatically reverts to `PUBLIC` after 300 seconds of inactivity (`session_timeout_seconds: 300`).
- **Encrypted Voice Profiles**: Voice embedding vectors are stored encrypted in the SQLite `voice_profiles` table using AES-GCM / Fernet keys protected via Windows DPAPI.
- **Zero Raw Audio Storage**: Raw audio buffers are held in volatile RAM only during VAD processing and are immediately overwritten.

---

## 4. Gated Interactive Confirmation

For all `HIGH`-risk tools (`delete_files`, `delete_note`, `shutdown`, `restart`):
- Execution is strictly suspended.
- A high-visibility modal card is rendered in the PySide6 HUD.
- The UI defaults keyboard focus to `CANCEL` / `DENY`.
- The confirmation dialog presents the exact target parameters (e.g. full path to file being deleted).
- If denied or timed out (30s), the operation aborts with a `DENIED` status and is logged to `audit_logs`.

---

## 5. Credential Store Architecture

External integration secrets (Google Gemini keys, future OAuth refresh tokens) are managed via an abstracted credential vault (`security/credentials/`):

```
                        [ BaseCredentialStore ]
                                   │
              ┌────────────────────┴────────────────────┐
              ▼                                         ▼
   [ WindowsCredentialStore ]                [ DevFallbackStore ]
   - Production Default                      - Local development / testing only
   - Windows Credential Manager              - Fernet encrypted file
   - Windows DPAPI (CryptProtectData)        - Requires master key in .env
```

- **Zero Plaintext Storage**: Credentials are never stored as plaintext in configuration files or SQLite tables.
- **OAuth Metadata Separation**: The `oauth_accounts` SQLite table contains only non-sensitive account metadata (`account_email`, `scopes`, `expires_at`, `credential_reference`). Actual tokens reside in the Windows Credential Vault.

---

## 6. Secret Redaction & Sanitization (`SecretRedactor`)

Before text is written to log files, persisted in database fields, or transmitted to cloud LLMs (Gemini), it passes through `SecretRedactor`:
- Detects and strips:
  - Google API keys (`AIzaSy...`)
  - OpenAI / Anthropic API keys (`sk-proj-...`, `sk-ant-...`)
  - Bearer tokens (`Bearer ...`)
  - Passwords and connection strings
- Replaces matches with `[REDACTED]` tokens.

---

## 7. Audit Logging (`AuditLogger`)

Every action executed by ProAssist AI produces an immutable record in the `audit_logs` SQLite table:
- `timestamp`: UTC ISO-8601 string.
- `authenticated`: 1 or 0 indicating biometric status at execution time.
- `user_command`: Redacted user utterance.
- `agent` & `tool`: Subsystem and method invoked.
- `parameters`: JSON dictionary of sanitized parameters.
- `result_status`: `SUCCESS`, `FAILURE`, `DENIED`, or `CANCELLED`.
- `result_detail`: Redacted human-readable execution summary.
- `error_message`: Stack trace or failure reason if unsuccessful.
