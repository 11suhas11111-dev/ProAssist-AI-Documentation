# Windows Validation & Scenario Testing — ProAssist AI

This document records the **manual and scripted end-to-end Windows scenarios** executed natively on Windows 11. It clearly distinguishes between **VERIFIED**, **AUTOMATED ONLY**, and **NOT VERIFIED** capabilities.

---

## 1. Classification Methodology

- **VERIFIED**: Executed directly on Windows 11 with live system calls (real PySide6 window rendering, real SQLite transactions, real audio capture/playback, or real file I/O).
- **AUTOMATED ONLY**: Verified through pytest test suites using in-memory or temporary database fixtures and mock providers.
- **NOT VERIFIED**: Component not yet implemented or requiring external hardware/models not present in the environment.

---

## 2. Windows Scenario Matrix

| Scenario / Subsystem | Validation Status | Verification Details |
|---|:---:|---|
| **Phase 6.6: Local SQLite Event Creation** | **VERIFIED** | Scenario 1: Creates local event with start/end time in `calendar_events`. |
| **Phase 6.6: Conflict Detection Warning** | **VERIFIED** | Scenario 2: Overlapping event generation emits `CalendarConflictWarning`. |
| **Phase 6.6: Ambiguity Detection & Prompt** | **VERIFIED** | Scenario 3: Multiple matching events produce structured `AmbiguityResult`. |
| **Phase 6.6: Google Calendar Truthful Auth** | **VERIFIED** | Scenario 4: Unconfigured Google provider returns `AUTHENTICATION_ERROR`. |
| **Phase 6.6: Mock Provider Error Sim** | **VERIFIED** | Scenario 5: Simulates `TIMEOUT`, `FAILURE`, and `RATE_LIMITED` gracefully. |
| **Phase 6.6: Gated Event Deletion** | **VERIFIED** | Scenario 6: `delete_event` halts for explicit `CONFIRMATION`. |
| **Phase 6.6: Idempotency Protection** | **VERIFIED** | Scenario 7: Idempotency lease prevents duplicate calendar events. |
| **Phase 6.6: Multilingual Routing EN/HI/KN**| **VERIFIED** | Scenario 8: TaskRouter routes calendar intents across English, Hindi, Kannada. |
| **Phase 6.6: UTC Normalization** | **VERIFIED** | Scenario 9: Converts local offsets into standardized UTC ISO strings. |
| **Phase 6.6: Event Search & Retrieval** | **VERIFIED** | Scenario 10: Window and query searches return matched event payloads. |
| **Phase 6.6: Event Update Modification** | **VERIFIED** | Scenario 11: Modifies event title, time, and location with conflict check. |
| **Phase 6.6: CalendarAgent Full Dispatch** | **VERIFIED** | Scenario 12: End-to-end execution of calendar agent actions via router. |
| **Phase 6.5: Mock Search & Citations** | **VERIFIED** | Scenario 1: Mock search execution returning structured, numbered citations. |
| **Phase 6.5: DuckDuckGo Live Search** | **VERIFIED** | Scenario 2: Live HTTP query via `SafeHttpClient` returning factual Wikipedia result. |
| **Phase 6.5: Query Boundary Validation** | **VERIFIED** | Scenario 3: Bounded query length ($\le 300$ chars), whitespace and empty rejection. |
| **Phase 6.5: Secret Redaction in Queries** | **VERIFIED** | Scenario 4: Registered and pattern secrets sanitized with `[REDACTED]` before dispatch. |
| **Phase 6.5: Provider Fallback Degradation**| **VERIFIED** | Scenario 5: Automatic failover from failing primary provider to working secondary. |
| **Phase 6.5: Multilingual Routing (EN/HI/KN)**| **VERIFIED** | Scenario 6: TaskRouter directs search requests across English, Hindi, and Kannada. |
| **Phase 6.5: Anti-Browser Barriers** | **VERIFIED** | Scenario 7: `PermissionManager`, `ToolRegistry`, and `FORBIDDEN_TOOLS` enforce non-browser invariants. |
| **Phase 6.5: WebSearchAgent Dispatch** | **VERIFIED** | Scenario 8: Agent execution dispatch returning synthesized truthful citations. |
| **Phase 6.4: Weather Live Request** | **VERIFIED** | Scenario A: Live forecast retrieved for Bengaluru via Open-Meteo REST API. |
| **Phase 6.4: Weather Alias Resolution** | **VERIFIED** | Scenario B: Resolves "Mysore" to "Mysuru" using local alias dictionary. |
| **Phase 6.4: Default Location Usage** | **VERIFIED** | Scenario C: Querying "what's the weather here" maps to stored default location. |
| **Phase 6.4: Zero Geolocation Prompt** | **VERIFIED** | Scenario D: No location available prompts user instead of querying device GPS. |
| **Phase 6.4: Offline Cache & Stale Fallback**| **VERIFIED** | Scenario E: Valid cache hit returns `CACHED`; network failure degrades to `STALE`. |
| **Phase 6.4: Ambiguous City Rejection** | **VERIFIED** | Scenario H: Querying "Springfield" returns disambiguation options without guessing. |
| **Phase 6.4: Basic Info vs Search** | **VERIFIED** | Scenario I: General questions route to LLM; Web Search remains unbuilt. |
| **Phase 6.3: Explicit Note Creation** | **VERIFIED** | Scenario A: "Remember that ..." creates note with FTS5 indexing. |
| **Phase 6.3: No Ambient Recording** | **VERIFIED** | Scenario B: Conversational chatter never creates unauthorized notes. |
| **Phase 6.3: FTS5 BM25 Ranked Search** | **VERIFIED** | Scenario C: BM25 ranked search returns relevant notes with snippet truncation. |
| **Phase 6.3: Gated Interactive Deletion** | **VERIFIED** | Scenarios F & G: `delete_note` halts for user confirmation. |
| **Phase 6.2: Natural Time Parsing** | **VERIFIED** | "tomorrow at 3pm", "in 45 minutes" parsed into correct UTC ISO timestamps. |
| **Phase 6.2: Active Reminder Scheduler** | **VERIFIED** | Async scheduler loop checks due reminders without blocking UI thread. |
| **Phase 6.1: Relationship Resolution** | **VERIFIED** | "call mom" resolves to mother's phone number via relationship graph. |
| **Phase 5.1: Floating HUD Modes** | **VERIFIED** | Live PySide6 window transitions: ORB mode, PANEL mode, EXPANDED mode. |
| **Phase 5.1: Confirmation Card Focus** | **VERIFIED** | Confirmation card displays target parameters; CANCEL button receives default focus. |
| **Phase 3.1: Wake Word Model Missing** | **VERIFIED** | Truthfully reports `MODEL_MISSING`; HUD shows `[Hey ProAssist MODEL NOT READY]`. |
| **Phase 3: Voice Biometrics Lockout** | **VERIFIED** | 3 failed speaker verifications trigger 30-second lockout. |
| **Phase 1: Windows Process & Volume** | **VERIFIED** | Real volume control, screenshots saved to Pictures, process management. |
| **Deep Neural Speaker Verification** | **NOT VERIFIED**| SpeechBrain ECAPA-TDNN not installed; local spectral centroid used instead. |
| **Custom Wake Word Acoustic Model** | **NOT VERIFIED**| Model `hey_proassist.onnx` is untrained. |

## Phase 6.7: Email Integration Windows Validation
- **Environment**: Windows 11 Home (64-bit) | Python 3.11.9
- **Script**: `scratch/test_phase6_7_scenarios.py`
- **Result**: 30/30 scenarios PASSED (Scenarios A through AD).
- **Verified Areas**: Account discovery, metadata search, message retrieval, prompt injection containment, draft lifecycle (create/get/update/delete), header injection checks, attachment validation (size, path, extension), contact resolution (exact, ambiguous halt, unknown halt, raw email), permission specs, send confirmation gating, idempotency lease caching, UNKNOWN retry blocking, Zero Fake OAuth, timeout/rate limit simulation, multilingual TaskRouter routing (EN, HI, KN).
