# Architecture Lock — Invariant Specifications

This document defines the **frozen architectural contracts** of ProAssist AI. Future developers and autonomous coding agents are strictly prohibited from refactoring, removing, or bypassing these patterns.

---

## 1. Core Operating Constraints

### 1.1 Single-Process Windows Desktop Application
- ProAssist AI runs as a single Python 3.11 desktop application on Windows 11.
- **NO Microservices**: Never introduce Docker containers, Redis, separate background daemons, or Celery workers. Inter-subsystem communication occurs via asynchronous in-memory method dispatch and SQLite.

### 1.2 Local-First Execution Barrier
- All core desktop automation (app launch, system telemetry, file handling, contacts, tasks, reminders, notes) must execute completely offline without cloud API roundtrips.
- Cloud calls are restricted exclusively to:
  1. Gemini LLM planning for unstructured queries.
  2. Edge-TTS neural voice synthesis when local Piper voices are absent.
  3. Open-Meteo weather REST API.

---

## 2. The 5-Stage Execution Pipeline

Every user turn, whether received via voice audio or text entry, MUST traverse the following linear defense-in-depth pipeline. No component may bypass intermediate stages:

```
Voice / Text Input
       │
       ▼
[ 1. ContextManager ] ─────► Assembles recent turns + user preference facts
       │
       ▼
[ 2. TaskRouter ] ─────────► Deterministic regex / keyword match (O(1) local dispatch)
       │                    ├── DIRECT MATCH ──► Generates ExecutionPlan directly
       │                    └── AMBIGUOUS ────► Invokes Cloud LLM Planner
       │
       ▼
[ 3. PlanValidator ] ──────► Validates ExecutionPlan against strict JSON schema:
       │                    - Max 10 steps
       │                    - Tool must exist in ToolRegistry allowlist
       │                    - Parameters must match required types
       │                    - Automatically elevates HIGH-risk tools
       │
       ▼
[ 4. PermissionManager ] ──► Validates caller session against AuthLevel:
       │                    - PUBLIC (0): Read-only info (system_info, time)
       │                    - AUTHENTICATED (1): Owner voice verified
       │                    - CONFIRMATION (2): Voice auth + explicit user prompt
       │
       ▼
[ 5. ConfirmationManager ] ► If tool requires CONFIRMATION:
       │                    - Blocks execution
       │                    - Renders interactive Confirmation Card in UI
       │                    - Default focus is CANCEL / DENY
       │                    - Proceeds ONLY if explicitly approved
       │
       ▼
[ 6. ExecutionEngine ] ────► Dispatches to target Agent -> Tool -> Service
       │                    - Logs record to SQLite AuditLogger
       │                    - Never allows tools to execute arbitrary shell code
```

---

## 3. LLM Governance: Planner-Only Isolation

The Large Language Model (Google Gemini) is isolated as an untrusted reasoning engine:

1. **Zero Execution Authority**: The LLM NEVER executes code, runs shell scripts, or directly triggers OS actions. It produces a declarative JSON plan (`ExecutionPlanSchema`).
2. **Forbidden Commands**: The system actively rejects and contains zero tools for:
   - `run_python`
   - `execute_shell`
   - `eval` / `exec`
   - `powershell` / `cmd`
   - Arbitrary URL fetching / web scraping
3. **No Direct Database Access**: The LLM cannot execute SQL or query SQLite tables directly. It must invoke high-level domain tools (`search_notes`, `list_tasks`).
4. **No Direct Credential Access**: The LLM never sees raw passwords, API keys, or private tokens. Secret inputs are sanitized by `SecretRedactor` before sending prompts to the cloud.

---

## 4. Database Invariants (SQLite)

- **Database Path**: Stored in `~/.proassist/proassist.db` (or project-configured data directory).
- **Driver**: `aiosqlite` exclusively with connection pooling via async context managers.
- **WAL Mode**: `PRAGMA journal_mode=WAL` and `PRAGMA foreign_keys=ON` are mandatory on every connection.
- **Migrations & Data Preservation**: Never reset, wipe, or drop tables in `proassist.db`. Schema evolutions must use idempotent `CREATE TABLE IF NOT EXISTS` and `CREATE INDEX IF NOT EXISTS`.

---

## 5. Network Safety Contract (`SafeHttpClient`)

All outbound network requests must use `SafeHttpClient`:
1. Strict connection and read timeouts (default 8–10s).
2. Automatic redaction of sensitive HTTP headers (`Authorization`, `X-API-Key`) from application logs.
3. Deterministic error mapping to the `ProviderStatus` enum (`SUCCESS`, `TIMEOUT`, `NETWORK_ERROR`, `RATE_LIMITED`, `SERVICE_ERROR`, `PAYLOAD_ERROR`).
4. Zero arbitrary HTTP requests: outbound URLs must target pre-configured provider endpoints.

---

## 6. Privacy & Sensor Invariants

1. **Zero Ambient Recording**: The microphone is opened strictly during active Push-to-Talk or after verified wake-word triggering. Audio is processed in memory and discarded. Raw audio is never persisted to disk.
2. **Zero Silent Geolocation**: ProAssist AI never queries Windows location services, GPS sensors, or background IP geolocation. Weather queries require explicit location strings or an explicitly configured default location.
3. **Zero Ambient Networking**: Weather and external integrations never poll in the background. Network traffic occurs only in response to explicit user turns.
