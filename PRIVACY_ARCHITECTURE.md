# Privacy Architecture — ProAssist AI

Privacy is the foundational design constraint of ProAssist AI. Unlike commercial smart speakers and cloud assistants that continuously stream ambient audio or upload complete personal databases to cloud servers, ProAssist AI is engineered around **strict data minimization and local edge isolation**.

---

## 1. Core Privacy Invariants

| Privacy Invariant | System Guarantee | Enforcement Mechanism |
|---|---|---|
| **Zero Ambient Audio Recording** | The microphone is never open continuously. Audio frames are never recorded to disk. | Audio capture activates strictly upon Push-to-Talk or hardware wake-word trigger. In-memory buffer is flushed immediately after STT. |
| **Zero Silent Geolocation** | ProAssist AI never queries Windows location services, GPS hardware, or IP-lookup APIs. | Weather and location queries require explicit city names or user-configured default location stored in SQLite. |
| **Zero Ambient Networking** | No background threads poll external APIs or check online servers. | Network requests occur exclusively during an explicit user turn. |
| **Local Personal Knowledge** | Personal notes, contacts, tasks, and reminders reside 100% locally in SQLite. | Stored in `proassist.db` at `~/.proassist/`. Zero cloud syncing without explicit configuration. |
| **Cloud LLM Data Minimization** | The user's personal database is NEVER uploaded in bulk to cloud LLMs. | `NotesService` extracts a maximum of 3 minimal relevant snippets via local FTS5 search before prompting Gemini. |
| **One-Way Voice Embeddings** | Voice biometric models do not preserve voice recordings. | Converts audio into irreversible mathematical feature vectors. Enrolled templates cannot be reconstructed into speech. |

---

## 2. Audio Privacy & Lifecycle

```
[ Microphone Stream ]
         │
         ▼
[ WebRTC VAD in RAM ] ──(No speech detected)──► Memory overwritten immediately
         │
         ▼ (Speech detected)
[ AudioBuffer in RAM ]
         │
         ├──► Local STT (Faster-Whisper on CPU) ──► Transcribed string
         │
         └──► Local Biometrics (SpeakerVerifier) ──► Feature vector
         │
         ▼
[ Buffer Purged ] ────► RAM explicitly released; ZERO WAV/MP3 files written to disk
```

---

## 3. Personal Notes & Knowledge Privacy Boundary

The `NotesService` (`memory/notes/service.py`) strictly enforces an information boundary between local SQLite and cloud LLMs:

1. **Explicit Memory Intent**: ProAssist AI never silently records conversational turns as notes. A note is created ONLY upon an explicit user directive (e.g. `"remember that my WiFi password is ..."` or `"take a note: ..."`).
2. **Local FTS5 Ranking**: Search queries execute locally using SQLite Full-Text Search (FTS5) and BM25 relevance ranking.
3. **Snippet Truncation for Cloud LLM**: When a conversational query requires Gemini to answer a question based on personal notes:
   - FTS5 selects the top relevant notes.
   - Truncates content to short snippets ($\le 250$ characters each).
   - Injects at most **3 snippets** into the prompt.
   - Completely prevents dumping the user's personal note repository to cloud infrastructure.

---

## 4. Weather & Location Privacy

Weather intelligence (`weather/`) respects user location boundaries:
- Querying `"what's the weather"` without specifying a location halts execution and prompts:
  `"Which location should I check? You can also set a default location by saying 'set default location to Bengaluru'."`
- Resolving `"Bangalore"` maps to `"Bengaluru"` using a local dictionary of common aliases without making network geocoding requests.
- Cache entries in `weather_cache` store only meteorological data (temperature, wind, humidity) and zero user identity or device metadata.

---

## 5. Telemetry & Local Diagnostics

- **No Remote Telemetry**: ProAssist AI contains zero telemetry SDKs (no Google Analytics, no Sentry, no Mixpanel).
- **Local Diagnostics**: System health, CPU usage, and RAM consumption monitored via `psutil` are displayed only on the local PySide6 HUD top bar and written to local rotating logs (`logs/proassist.log`).

## Email Privacy Controls (Phase 6.7)
- Local SQLite storage for email accounts, drafts, and metadata (`proassist.db`).
- Zero ambient networking: no background email fetchers, polling daemons, or telemetry.
- Network access occurs strictly upon explicit user command and through `SafeHttpClient`.
- Outgoing communication requires explicit user confirmation.
