# ProAssist AI / Friday — Documentation & Development Handoff

**ProAssist AI** (internally codenamed **Friday**) is an intelligent, secure, multilingual, local-first personal desktop voice assistant built for **Windows 11** in **Python 3.11**. It combines a floating, HUD-style desktop interface (inspired by Stark Industries / FRIDAY) with a deterministic, security-gated multi-agent execution architecture, local speech and biometric processing, and cloud LLM planning.

> **Current Status**: **Phase 6.4 Complete** (Weather & Information)  
> **Test Baseline**: **367 passed, 0 failed, 0 errors, 0 regressions**  
> **Next Phase**: **Phase 6.5 — Web Search**

---

## 1. Core Philosophy & Architectural Principles

1. **Local-First Execution**:
   Deterministic operating system automation, task management, contacts, personal notes, and local system queries execute completely offline without cloud dependencies.
2. **Zero Ambient Recording & Absolute Privacy**:
   Microphone streams are never recorded or written to disk. WebRTC VAD audio buffers are processed in memory and discarded immediately after transcription and biometric verification. Audio profiles contain only one-way mathematical voice embeddings.
3. **Zero Silent Geolocation & Zero Ambient Networking**:
   The assistant never uses Windows Location APIs, background GPS, or IP geolocation. Weather data requires an explicit location query or user-configured default. No ambient background network polling occurs.
4. **LLM as Planner, Never Arbitrary Code Executor**:
   Cloud LLMs (Google Gemini 2.5 Flash) act solely as structured JSON reasoning engines (`ExecutionPlanSchema`). The LLM has zero direct shell execution authority, zero database access, zero direct network access, and cannot bypass permission or confirmation gates.
5. **Defense-in-Depth Security Chain**:
   Every action passes through a strict 5-stage pipeline:
   $$\text{TaskRouter} \longrightarrow \text{PlanValidator} \longrightarrow \text{PermissionManager} \longrightarrow \text{ConfirmationManager} \longrightarrow \text{ExecutionEngine}$$
   Destructive actions (`delete_files`, `delete_note`, `shutdown`, etc.) require mandatory explicit interactive user confirmation.
6. **Single Windows Desktop Process**:
   PySide6 provides an interactive floating HUD with three distinct presentation states: `ORB` (small desktop widget), `PANEL` (semi-expanded HUD), and `EXPANDED` (detailed command/settings window).

---

## 2. Capabilities Overview

| Capability Area | Status | Implementation Details |
|---|---|---|
| **System & Desktop Control** | **IMPLEMENTED** | App launching/killing, volume control, screenshots, process listing, hardware telemetry. |
| **File Automation** | **IMPLEMENTED** | Copy, move, rename, delete (gated), compress (ZIP), extract, duplicate file detection. |
| **Voice Interaction Pipeline**| **IMPLEMENTED** | PyAudio/sounddevice 16kHz capture, WebRTC VAD, Faster-Whisper local STT, Edge-TTS fallback. |
| **Voice Biometrics** | **IMPLEMENTED** | Spectral centroid speaker verification, enrollment, session lockout, encrypted profiles. |
| **Wake Word Detection** | **PARTIAL** | OpenWakeWord integration ready; custom `"Hey ProAssist"` acoustic model **NOT READY**. |
| **Floating HUD UI** | **IMPLEMENTED** | PySide6 FRIDAY floating HUD with ORB/PANEL/EXPANDED modes, radial visualizer, status indicators. |
| **Contacts & Entity Resolution**| **IMPLEMENTED** | Canonical name normalization, alias matching, relationship queries ("call mom"), SQLite storage. |
| **Local Tasks & Reminders** | **IMPLEMENTED** | Natural-language time parsing, recurring rules (daily, weekly), SQLite persistence, active scheduler. |
| **Notes & Personal Knowledge** | **IMPLEMENTED** | SQLite FTS5 BM25 ranked full-text search, explicit intent only, privacy-bounded LLM snippets (<= 3). |
| **Weather & Information** | **IMPLEMENTED** | Open-Meteo REST API via `SafeHttpClient`, SQLite cache (30m/3h TTL), truthful freshness states (`LIVE`/`CACHED`/`STALE`/`UNAVAILABLE`). |
| **Web Search & Browsing** | **PLANNED** | Scheduled for Phase 6.5. Strictly unbuilt currently. |
| **Calendar Integration** | **PLANNED** | Scheduled for Phase 6.6. |
| **Email Integration** | **PLANNED** | Scheduled for Phase 6.7. |
| **WhatsApp / Messaging** | **PLANNED** | Scheduled for Phase 6.8. |
| **Cross-Service Workflows** | **PLANNED** | Scheduled for Phase 6.9. |

---

## 3. Technology Stack

- **Operating System**: Windows 11 (AMD64)
- **Runtime**: Python 3.11.9
- **Desktop UI**: PySide6 (Qt 6.7+)
- **Local Storage**: SQLite 3 with Write-Ahead Logging (WAL) and strict foreign keys via `aiosqlite`
- **Security & Crypto**: Windows DPAPI, Windows Credential Manager, `cryptography` (Fernet)
- **Speech-to-Text**: `faster-whisper` (Base model, int8 quantization on CPU)
- **Text-to-Speech**: `edge-tts` (English, Hindi, Kannada neural voices), `piper-tts` architecture
- **Voice Activity Detection**: `webrtcvad`
- **System Telemetry**: `psutil`, `pyautogui`, `pyperclip`
- **HTTP Client**: `httpx` wrapped in custom `SafeHttpClient` with timeout and credential stripping
- **Testing**: `pytest`, `pytest-asyncio`, `pytest-cov`

---

## 4. Documentation Map

This repository serves as the definitive architecture and handoff knowledge base. Navigate the full specification using the links below:

### Core Architecture & Governance
- [**NEW_ACCOUNT_HANDOFF.md**](NEW_ACCOUNT_HANDOFF.md): **Start Here!** Complete guide for any new agent or developer taking over the codebase.
- [**ARCHITECTURE_LOCK.md**](ARCHITECTURE_LOCK.md): Non-negotiable architectural invariants that must never be altered or regressed.
- [**PROJECT_STATUS.md**](PROJECT_STATUS.md): Authoritative tracking of completed, in-progress, and planned milestones.
- [**SECURITY_INVARIANTS.md**](SECURITY_INVARIANTS.md): Formal security rules, risk levels, and authentication checks.
- [**DEVELOPMENT_WORKFLOW.md**](DEVELOPMENT_WORKFLOW.md): Rules for branch management, test verification, and phase transitions.

### Technical Deep Dives
- [**SYSTEM_ARCHITECTURE.md**](SYSTEM_ARCHITECTURE.md): Comprehensive end-to-end subsystem breakdown and request lifecycle.
- [**SECURITY_ARCHITECTURE.md**](SECURITY_ARCHITECTURE.md): Authentication levels, credential management, DPAPI, and audit trails.
- [**PRIVACY_ARCHITECTURE.md**](PRIVACY_ARCHITECTURE.md): Local-first data isolation, secret redaction, and cloud data minimization.
- [**DATA_ARCHITECTURE.md**](DATA_ARCHITECTURE.md): SQLite storage patterns, connection lifecycle, and transactional integrity.
- [**DATABASE_SCHEMA.md**](DATABASE_SCHEMA.md): Complete DDL, column types, relationships, and indexes for all 19 database tables.
- [**AGENT_TOOL_ARCHITECTURE.md**](AGENT_TOOL_ARCHITECTURE.md): Complete catalog of all 6 agents and 50 registered tools.
- [**LLM_ARCHITECTURE.md**](LLM_ARCHITECTURE.md): Gemini provider integration, structured JSON planning, and safety guardrails.
- [**VOICE_ARCHITECTURE.md**](VOICE_ARCHITECTURE.md): Audio capture, VAD, STT, TTS, and biometrics pipeline.
- [**UI_ARCHITECTURE.md**](UI_ARCHITECTURE.md): PySide6 FRIDAY floating HUD, orbital visualizer, and telemetry cards.
- [**NETWORK_PROVIDER_ARCHITECTURE.md**](NETWORK_PROVIDER_ARCHITECTURE.md): `SafeHttpClient`, error mapping, and external service contracts.

### Operations & Planning
- [**TESTING_STRATEGY.md**](TESTING_STRATEGY.md): Test hierarchy, mocking conventions, and regression barriers.
- [**DEPENDENCIES.md**](DEPENDENCIES.md): Full dependency inventory, versions, and rationale.
- [**CONFIGURATION.md**](CONFIGURATION.md): `config.yaml` options and `.env` template guidelines.
- [**KNOWN_LIMITATIONS.md**](KNOWN_LIMITATIONS.md): Candid documentation of current technical boundaries.
- [**DEVELOPMENT_ROADMAP.md**](DEVELOPMENT_ROADMAP.md): Detailed blueprints for Phases 6.5 through 6.10.

### Historical Phase Reports
- [**Phase 1: Core System & File Automation**](phases/PHASE_1.md)
- [**Phase 2: Voice Pipeline & Multilingual Support**](phases/PHASE_2.md)
- [**Phase 3: Voice Authentication & Wake Word**](phases/PHASE_3.md)
- [**Phase 3.1: Security Hardening**](phases/PHASE_3_1.md)
- [**Phase 4: Structured Task Planning & Intelligence**](phases/PHASE_4.md)
- [**Phase 5: FRIDAY Desktop HUD**](phases/PHASE_5.md)
- [**Phase 5.1: Floating Desktop UI Refinement**](phases/PHASE_5_1.md)
- [**Phase 6.0: Integration Foundation & Credential Store**](phases/PHASE_6_0.md)
- [**Phase 6.1: Contacts & Entity Resolution**](phases/PHASE_6_1.md)
- [**Phase 6.2: Tasks & Reminders Subsystem**](phases/PHASE_6_2.md)
- [**Phase 6.3: Notes & Personal Knowledge**](phases/PHASE_6_3.md)
- [**Phase 6.4: Weather & Information Service**](phases/PHASE_6_4.md)

### Validation & Verification
- [**TEST_RESULTS.md**](validation/TEST_RESULTS.md): Full pytest run logs and test suite metrics.
- [**WINDOWS_VALIDATION.md**](validation/WINDOWS_VALIDATION.md): Execution records of manual and simulated Windows end-to-end test scenarios.
- [**RELEASE_READINESS.md**](validation/RELEASE_READINESS.md): Production checklist and verification gates.
