# System Architecture — ProAssist AI / Friday

ProAssist AI is structured as a modular, single-process, multi-agent assistant application running natively on **Windows 11**. It enforces a strict **Hybrid Edge-Cloud** computing topology: all deterministic system tasks, file operations, personal contacts, reminders, and notes execute locally at the edge, while unstructured reasoning and natural-language planning escalate to cloud LLMs (Google Gemini).

---

## 1. High-Level Architecture Diagram

```
+─────────────────────────────────────────────────────────────────────────────+
|                         PRESENTATION LAYER (PySide6)                        |
|                                                                             |
|   [ Floating ORB Widget ] ◄──► [ Semi-Expanded PANEL ] ◄──► [ EXPANDED Window ]|
|   - Holographic Visualizer (RMS Reactive)  - Top Telemetry Bar               |
|   - Conversation Panel Stream              - Interactive Confirmation Card   |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ (UI Events / Commands)
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                       VOICE & AUDIO PIPELINE (Local)                        |
|                                                                             |
|   AudioCapture ──► WebRTC VAD ──► AudioBuffer ──► Faster-Whisper Local STT  |
|         │                                │                                   |
|         ▼                                ▼                                   |
|   OpenWakeWord Engine            SpeakerVerifier (Voice Biometrics)         |
|   (Neural Detection)             (128-d Acoustic Centroid Embeddings)       |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ (Transcribed Text + Auth Level)
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                          INTELLIGENCE CORE LAYER                            |
|                                                                             |
|   [ ContextManager ] ──► Injects conversation turns & user preferences      |
|           │                                                                 |
|           ▼                                                                 |
|     [ TaskRouter ]                                                          |
|     ├── TIER 1: Direct Deterministic Regex ──► Local Plan (0 LLM overhead)  |
|     └── TIER 2: Unstructured Request ───────► Gemini 2.5 Flash Cloud LLM   |
|                                                     │ (JSON Plan)           |
|                                                     ▼                       |
|                                            [ PlanValidator ]                |
|                                            - Tool Allowlist Check           |
|                                            - Max 10 Steps Check             |
|                                            - Risk Level Auto-Elevation      |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ (Validated ExecutionPlan)
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                    SECURITY & GOVERNANCE GATEWAY                            |
|                                                                             |
|   [ PermissionManager ] ──► Validates AuthLevel (PUBLIC / AUTH / CONFIRM)   |
|           │                                                                 |
|           ▼                                                                 |
|   [ ConfirmationManager ] ─► Gated interactive approval for HIGH-risk actions|
|           │                                                                 |
|           ▼                                                                 |
|   [ IdempotencyManager ] ──► Leased execution tokens & state transitions     |
|           │                                                                 |
|           ▼                                                                 |
|   [ AuditLogger ] ─────────► Immutable audit record in SQLite audit_logs    |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ (Authorized Step Dispatch)
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                     MULTI-AGENT EXECUTION ENGINE                            |
|                                                                             |
|   ExecutionEngine dispatches execution tasks to registered agents:          |
|                                                                             |
|   ├── SystemAgent   ──► System tools (app control, screenshots, volume, ps) |
|   ├── FileAgent     ──► File tools (copy, move, delete, compress, search)   |
|   ├── ContactAgent  ──► Contact tools (resolve aliases, lookup, CRUD)       |
|   ├── TaskAgent     ──► Task & Reminder tools (natural time, scheduler)     |
|   ├── NotesAgent    ──► Personal Knowledge tools (SQLite FTS5 BM25 search)  |
|   └── WeatherAgent  ──► Weather tools (Open-Meteo REST via SafeHttpClient)  |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ (Local I/O, SQLite, Network)
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                    PERSISTENCE & SERVICE INFRASTRUCTURE                     |
|                                                                             |
|   - aiosqlite (WAL Mode, Foreign Keys ON, proassist.db)                     |
|   - Windows DPAPI & Credential Manager (Secure credential store)            |
|   - SafeHttpClient (httpx client with timeout, retry & secret redaction)    |
|   - Windows Native APIs (psutil, pyautogui, pywin32)                        |
+─────────────────────────────────────────────────────────────────────────────+
```

---

## 2. Detailed Subsystem Breakdown

### 2.1 Presentation Layer (`ui/`)
- **Framework**: PySide6 (Qt 6.7+).
- **Core Modes**:
  - `ORB`: Small floating circular desktop widget with rotating gyroscopic orbital rings and breathing core.
  - `PANEL`: Semi-expanded floating HUD widget showing active assistant state, audio-reactive waveform, and quick action bar.
  - `EXPANDED`: Full dashboard featuring conversation stream history, settings configuration, and execution cards.
- **State Machine**: Visual states derive strictly from backend events: `IDLE`, `LISTENING`, `AUTHENTICATING`, `THINKING`, `EXECUTING`, `SPEAKING`, `CONFIRMATION_REQUIRED`, `ERROR`.

### 2.2 Voice & Audio Pipeline (`voice/`, `security/voice_auth/`)
- **Audio Capture**: 16,000 Hz, 16-bit mono PCM stream via `sounddevice`.
- **Speech Detection**: `webrtcvad` operating in 30ms frames. Captures until 1.5s of consecutive silence is detected.
- **Biometric Authentication**: Voice embeddings computed using local spectral centroid and acoustic feature vectors. Compared against stored enrollment profile using cosine similarity ($\ge 0.70$).
- **Local STT**: `faster-whisper` (Base model, int8 quantization) running locally on CPU.
- **Local-First TTS**: Synthesizes responses via `edge-tts` (Indian neural voices `en-IN-NeerjaNeural`, `hi-IN-SwaraNeural`, `kn-IN-SapnaNeural`) with local playback.

### 2.3 Context & Planning (`core/`, `core/llm/`)
- **ContextManager**: Maintains rolling memory of the last 8 conversation turns and injects relevant user facts.
- **TaskRouter**: Evaluates input against high-performance regex patterns across English, Hindi, and Kannada. Routes matching deterministic commands directly to agents without LLM latency.
- **Cloud LLM (Gemini 2.5 Flash)**: Receives unroutable or ambiguous commands. Generates a strictly validated JSON structure conforming to `ExecutionPlanSchema`.
- **PlanValidator**: Enforces safety constraints: plan cannot exceed 10 steps, only approved tools in `ToolRegistry` are accepted, and forbidden operations (`run_python`, `execute_shell`, `eval`) trigger immediate plan rejection.

### 2.4 Governance & Security Gateway (`security/`, `core/idempotency.py`)
- **PermissionManager**: Maps tools to `RiskLevel` (`LOW`, `MEDIUM`, `HIGH`) and enforces minimum `AuthLevel` (`PUBLIC`, `AUTHENTICATED`, `CONFIRMATION`).
- **ConfirmationManager**: Intercepts `HIGH`-risk actions (e.g. `delete_files`, `shutdown`, `delete_note`). Prompts user via interactive PySide6 dialog with default focus on `CANCEL`.
- **IdempotencyManager**: Prevents duplicate executions of mutating actions using leased operation IDs (`PENDING` -> `IN_PROGRESS` -> `SUCCEEDED`/`FAILED`).
- **AuditLogger**: Commits every execution attempt, parameters, calling user status, and result to the SQLite `audit_logs` table.

### 2.5 Multi-Agent Execution Layer (`agents/`, `tools/`)
The system employs 6 specialized domain agents inheriting from `BaseAgent`:
1. `SystemAgent`: Desktop process and hardware management via Windows APIs.
2. `FileAgent`: Safe filesystem operations with destination boundary checks.
3. `ContactAgent`: Contact address book and multi-criteria entity resolution.
4. `TaskAgent`: Todo task management, natural time parsing, and active reminder scheduling.
5. `NotesAgent`: Full-text search personal knowledge base backed by SQLite FTS5.
6. `WeatherAgent`: Weather intelligence and caching backed by Open-Meteo REST API.

---

## 3. End-to-End Request Lifecycle

```
[ User speaks: "Delete the old report.pdf in documents" ]
                           │
                           ▼
  1. AudioCapture streams 16kHz audio to WebRTC VAD
  2. VAD detects speech termination -> passes buffer
  3. SpeakerVerifier verifies voice biometrics against owner profile
     - Elevates session state to AuthLevel.AUTHENTICATED
  4. Faster-Whisper transcribes speech: "delete report.pdf"
  5. ContextManager assembles context turn
  6. TaskRouter matches FileAgent delete pattern:
     -> tool: delete_files, params: {"filename": "report.pdf"}
  7. PlanValidator verifies tool allowlist and parameter types
  8. PermissionManager detects delete_files is HIGH risk
     -> Demands AuthLevel.CONFIRMATION
  9. ConfirmationManager halts execution:
     -> Displays Crimson Confirmation Card in PySide6 UI:
        "Confirm: Delete file report.pdf?"
     -> User clicks "CONFIRM"
 10. ExecutionEngine dispatches to FileAgent.execute()
 11. FileAgent executes concrete tool -> file deleted from disk
 12. AuditLogger records SUCCESS in audit_logs
 13. TTS speaks confirmation: "Report.pdf has been deleted."
```
