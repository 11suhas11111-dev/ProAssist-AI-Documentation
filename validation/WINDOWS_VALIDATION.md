# Windows Validation & Scenario Testing — ProAssist AI / Friday

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
