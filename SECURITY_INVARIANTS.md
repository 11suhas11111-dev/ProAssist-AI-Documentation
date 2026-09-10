# Security Invariants — Verified Checklist

This document specifies the **non-negotiable security invariants** that govern the ProAssist AI codebase. Every invariant in this list is enforced by source code and covered by automated test suites.

---

## 1. Authentication & Voice Biometrics

- [x] **Invariant 1.1**: Wake word detection alone CANNOT grant elevated authentication. Wake word triggers audio recording; speaker verification must independently verify voice biometrics.
- [x] **Invariant 1.2**: Failed speaker verification blocks access to protected tools and speaks an "Access Denied" notice.
- [x] **Invariant 1.3**: After 3 consecutive failed verification attempts, voice authentication is locked out for 30 seconds.
- [x] **Invariant 1.4**: Authenticated sessions automatically revert to `PUBLIC` after 300 seconds of inactivity.
- [x] **Invariant 1.5**: Speaker enrollment requires at least 3 distinct audio samples of $\ge 1.5$ seconds each with acoustic consistency $\ge 0.50$.
- [x] **Invariant 1.6**: Raw user audio is never written to disk or stored in SQLite; only mathematical feature vectors are persisted.

---

## 2. Permissions & Gated Execution

- [x] **Invariant 2.1**: Authentication cannot bypass permissions. Even an authenticated owner cannot execute HIGH-risk tools without interactive confirmation.
- [x] **Invariant 2.2**: All destructive tools (`delete_files`, `delete_note`, `delete_task`, `shutdown`, `restart`) require explicit `CONFIRMATION`.
- [x] **Invariant 2.3**: Interactive confirmation dialogs must default focus to `CANCEL` / `DENY`.
- [x] **Invariant 2.4**: Denied, cancelled, or timed-out confirmations completely abort execution and trigger an audit entry.
- [x] **Invariant 2.5**: Unknown tools are treated as `HIGH` risk by default and require confirmation.

---

## 3. LLM Governance & Planning Constraints

- [x] **Invariant 3.1**: The LLM is an untrusted planner. It NEVER executes code, runs shell scripts, or interacts directly with the OS.
- [x] **Invariant 3.2**: Execution plans produced by LLMs must strictly conform to `ExecutionPlanSchema` and pass `PlanValidator`.
- [x] **Invariant 3.3**: The tool allowlist strictly forbids and rejects:
  - `run_python`
  - `execute_shell`
  - `eval` / `exec`
  - `powershell` / `cmd`
  - Arbitrary URL fetching / scraping
- [x] **Invariant 3.4**: An execution plan cannot exceed 10 steps under any circumstance.
- [x] **Invariant 3.5**: The LLM has zero direct database access (cannot execute raw SQL against `proassist.db`).
- [x] **Invariant 3.6**: The LLM has zero direct credential access (cannot read Windows Credential Vault or raw tokens).

---

## 4. Credential & Data Security

- [x] **Invariant 4.1**: API keys, passwords, and OAuth tokens are NEVER stored in plaintext in SQLite or configuration files.
- [x] **Invariant 4.2**: Windows DPAPI (`CryptProtectData`) and Windows Credential Manager are used for production credential storage.
- [x] **Invariant 4.3**: All application logs and cloud LLM prompts pass through `SecretRedactor` to strip API keys and bearer tokens.
- [x] **Invariant 4.4**: SQLite database connections must unconditionally enforce `PRAGMA journal_mode=WAL` and `PRAGMA foreign_keys=ON`.
- [x] **Invariant 4.5**: Mutating operations must use `IdempotencyManager` to prevent duplicate step execution.

---

## 5. Network & Sensor Safety

- [x] **Invariant 5.1**: All external HTTP requests must route through `SafeHttpClient` with strict connection/read timeouts (8–10s).
- [x] **Invariant 5.2**: Outbound HTTP headers are sanitized to prevent token leakage into debug logs.
- [x] **Invariant 5.3**: Zero Silent Geolocation: ProAssist AI never queries Windows location services, GPS hardware, or IP lookup APIs.
- [x] **Invariant 5.4**: Zero Ambient Networking: No background polling timers exist for weather or external integrations.
- [x] **Invariant 5.5**: Weather cache data is never falsely presented as fresh live data; stale cache degradation requires explicit notice.

---

## 6. Web Search & Anti-Browser Safety

- [x] **Invariant 6.1**: Strictly zero browser automation (Playwright, Selenium, Puppeteer, headless browsers, JS execution) are permitted.
- [x] **Invariant 6.2**: Anti-browser tools are permanently forbidden in `ToolRegistry.FORBIDDEN_TOOLS`.
- [x] **Invariant 6.3**: Search queries must be sanitized by `SecretRedactor` to prevent accidental credential leakage.
- [x] **Invariant 6.4**: Truthful Citations: The assistant must never fabricate URLs, domain names, or source citations.
- [x] **Invariant 6.5**: Search queries are strictly bounded to 300 characters and non-empty strings.

---

## 7. Calendar Integration Safety

- [x] **Invariant 7.1**: Zero Fake OAuth: Unconfigured external calendar integration must truthfully report `ProviderStatus.AUTHENTICATION_ERROR` without fabricating credentials.
- [x] **Invariant 7.2**: Offline-First Local Store: Local calendar operations execute 100% locally against SQLite with zero outbound network calls.
- [x] **Invariant 7.3**: Conflict Warnings as Collaborative Prompts: Overlapping events generate explicit `CalendarConflictWarning` notifications rather than silently overriding or auto-rejecting.
- [x] **Invariant 7.4**: Gated Destruction: `delete_event` is strictly classified as `RiskLevel.HIGH` and requires `AuthLevel.CONFIRMATION`.
- [x] **Invariant 7.5**: Timezone Normalization: All event timestamps are parsed, converted, and stored in normalized UTC ISO-8601 strings.

## Email Security Invariants (Phase 6.7)
1. **Header Injection Defense**: Carriage returns and newlines (`\r`, `\n`) are strictly prohibited in email addresses, CC/BCC, and subjects to prevent SMTP header injection.
2. **Prompt Injection Containment**: Email bodies are enclosed in `<untrusted_email_content>` tags with explicit instructions forbidding execution of commands inside the email content.
3. **Attachment Validation**: Attachments must be existing regular files <= 25 MB. Executable extensions (`.exe`, `.bat`, `.cmd`, `.ps1`, `.vbs`, `.js`, etc.) are unconditionally rejected.
4. **HIGH-Risk Send Confirmation**: `send_email` requires explicit confirmation via `ConfirmationManager`. Default button focus is CANCEL.
5. **Idempotency Lease & UNKNOWN Retry Prohibition**: Send operations acquire an atomic idempotency lease. If an operation yields `UNKNOWN` status, automatic retries are strictly prohibited to prevent duplicate emails.
6. **Zero Fake OAuth**: Google Mail provider never simulates successful authentication when credentials are missing.
