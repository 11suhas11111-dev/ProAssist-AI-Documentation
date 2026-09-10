# New Account Handoff — Developer Onboarding Guide

> **CRITICAL NOTICE FOR NEW DEVELOPERS & AI AGENTS**:  
> Read this document completely BEFORE inspecting or editing code. This is an existing, mature, 12-phase project with **367 passing tests** on Windows 11. Do NOT refactor the architecture, do NOT rewrite completed phases, and do NOT install unapproved dependencies.

---

## 1. What is ProAssist AI / Friday?

ProAssist AI is an intelligent, secure, local-first personal voice assistant engineered for Windows 11. Inspired by the Stark Industries FRIDAY HUD aesthetic, it provides seamless desktop automation, personal information management (contacts, tasks, reminders, notes, weather), and conversational intelligence.

### Core Distinctions
- **Local-First**: Does not need an internet connection for core desktop actions, file management, contacts, tasks, or notes.
- **Privacy-First**: No background audio recording, no silent location tracking, zero credential exposure, one-way voice biometric embeddings.
- **Deterministic & Safe**: LLMs are planners only; high-risk actions require interactive user confirmation.

---

## 2. Where is the Project?

- **Application Repository**: `c:\Users\DELL\OneDrive\Documents\ProAssist AI`
- **Documentation Repository**: `c:\Users\DELL\OneDrive\Documents\ProAssist-AI-Documentation`
- **GitHub Documentation URL**: `https://github.com/11suhas11111-dev/ProAssist-AI-Documentation`
- **Active Virtual Environment / Python**: `C:\Users\DELL\AppData\Local\Programs\Python\Python311\python.exe`
- **Application Database**: Local SQLite database at `~/.proassist/proassist.db`

---

## 3. Current Project Baseline

| Metric | Verified Value |
|---|---|
| **Python Version** | Python 3.11.9 (64-bit) |
| **Operating System** | Windows 11 Professional / Home |
| **Total Automated Tests** | **367 passed** |
| **Test Failures / Errors** | **0 failed, 0 errors, 0 regressions** |
| **Current Completed Phase**| **Phase 6.4 — Weather & Information** |
| **Next Target Phase** | **Phase 6.5 — Web Search** |

---

## 4. Completed Phases Summary

1. **Phase 1 (Core Architecture)**: PySide6 application, rule-based `TaskRouter`, `ExecutionEngine`, `PermissionManager`, `ActionConfirmation`, `AuditLogger`, `SystemAgent` (10 tools), `FileAgent` (10 tools).
2. **Phase 2 (Voice Pipeline)**: Audio capture (16kHz PCM), WebRTC VAD, Faster-Whisper local STT, language detector (EN, HI, KN), Edge-TTS fallback.
3. **Phase 3 (Voice Authentication)**: Local voice biometrics, spectral centroid extraction, user enrollment, encrypted voice profiles, OpenWakeWord architecture.
4. **Phase 3.1 (Security Hardening)**: Truthful wake-word readiness states (`MODEL NOT READY`), session timeouts (300s), lockout policies (3 failed attempts), constant-time comparisons.
5. **Phase 4 (Structured Planning)**: Google Gemini 2.5 Flash provider, structured JSON `ExecutionPlanSchema`, strict `PlanValidator` (tool allowlist, max 10 steps), local/cloud badges.
6. **Phase 5 (FRIDAY HUD Frontend)**: Holographic AI Visualizer with audio-reactive radial pulses, top telemetry bar, conversation stream, confirmation cards, settings panel.
7. **Phase 5.1 (Floating Desktop Refinement)**: Floating ORB mode, semi-expanded PANEL mode, EXPANDED window, system tray integration, fixed confirmation state consistency.
8. **Phase 6.0 (Integration Foundation)**: `SafeHttpClient`, `ProviderStatus` enum, `IdempotencyManager`, `WindowsCredentialStore` (DPAPI), `DevFallbackStore`, `oauth_accounts` table.
9. **Phase 6.1 (Contacts & Entity Resolution)**: `ContactService`, `ContactRepository`, `ContactResolver` (alias & relationship resolution: "call mom"), `ContactAgent` (6 tools).
10. **Phase 6.2 (Tasks & Reminders)**: `TaskService`, `TaskRepository`, `TaskResolver`, natural language `TimeParser`, `RecurrenceManager`, active `ReminderScheduler`, `TaskAgent` (11 tools).
11. **Phase 6.3 (Notes & Personal Knowledge)**: `NotesService`, `NotesRepository`, `NoteResolver`, SQLite FTS5 BM25 ranked search, zero ambient recording invariant, minimal LLM snippets (<= 3), `NotesAgent` (8 tools).
12. **Phase 6.4 (Weather & Information)**: `WeatherService`, `WeatherCacheManager`, `LocationResolver`, Open-Meteo REST integration via `SafeHttpClient`, zero silent geolocation, zero ambient networking, `WeatherAgent` (5 tools).

---

## 5. Architectural Contracts (Non-Negotiable)

1. **Never Bypass the Security Chain**:
   $$\text{TaskRouter} \longrightarrow \text{PlanValidator} \longrightarrow \text{PermissionManager} \longrightarrow \text{ConfirmationManager} \longrightarrow \text{ExecutionEngine}$$
2. **LLMs Produce JSON Plans, Never Code**:
   The LLM is strictly prohibited from running shell commands, python scripts, or arbitrary code. It only outputs validated JSON containing approved tools from `ToolRegistry`.
3. **Preserve Database Integrity**:
   Never reset or wipe `proassist.db`. Use idempotent migrations. All connections must enforce `WAL` mode and `foreign_keys=ON`.
4. **Zero Ambient Networking & Zero Silent Geolocation**:
   Never add background threads polling external APIs or checking device GPS/IP location.
5. **Preserve Test Baseline**:
   All 367 existing tests must pass before and after any new feature is merged.

---

## 6. How a New Agent Must Start

When assigned a new task on this project:

```powershell
# 1. Open project root
cd "c:\Users\DELL\OneDrive\Documents\ProAssist AI"

# 2. Run test suite to verify baseline
$PYTHON = "C:\Users\DELL\AppData\Local\Programs\Python\Python311\python.exe"
& $PYTHON -m pytest -q

# Expected output: 367 passed in ~75s

# 3. Read mandatory architecture docs:
# - ARCHITECTURE_LOCK.md
# - SECURITY_INVARIANTS.md
# - PROJECT_STATUS.md
# - docs/phase6_4_weather_information.md

# 4. Review the target specification for Phase 6.5 (Web Search) in DEVELOPMENT_ROADMAP.md
```

---

## 7. Prohibited Actions for New Agents

- **DO NOT** rewrite or replace the PySide6 user interface.
- **DO NOT** replace SQLite with PostgreSQL, MongoDB, or ChromaDB.
- **DO NOT** add background microservices, Redis, or Celery.
- **DO NOT** bypass `PermissionManager` or `ConfirmationManager`.
- **DO NOT** allow the LLM to execute shell commands or arbitrary python.
- **DO NOT** introduce silent background geolocation or ambient network polling.
- **DO NOT** commit secrets, API keys, or raw personal data.
- **DO NOT** claim tests passed without actually executing `pytest`.
