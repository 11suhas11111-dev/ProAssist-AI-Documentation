# Dependency Inventory — ProAssist AI / Friday

All dependencies are pinned and tracked in `requirements.txt`. The application runs on Python 3.11.9 (64-bit) on Windows 11.

---

## 1. Production Runtime Dependencies

| Package | Minimum Version | Purpose | Phase Introduced |
|---|---|---|:---:|
| **`fastapi`** | `>=0.115.0` | In-process API data modeling and route validation. | Phase 1 |
| **`uvicorn[standard]`** | `>=0.30.0` | ASGI server runtime support. | Phase 1 |
| **`websockets`** | `>=13.0` | Asynchronous duplex streaming (used by Edge-TTS). | Phase 1 |
| **`pydantic`** | `>=2.9.0` | Strongly-typed data validation and plan schemas. | Phase 1 |
| **`pydantic-settings`** | `>=2.5.0` | Validated environment and application configuration. | Phase 1 |
| **`pyyaml`** | `>=6.0.2` | Loading and parsing `config/config.yaml`. | Phase 1 |
| **`python-dotenv`** | `>=1.0.1` | Loading `.env` secrets and overrides. | Phase 1 |
| **`loguru`** | `>=0.7.2` | Structured, colored, rotating audit and debug logging. | Phase 1 |
| **`PySide6`** | `>=6.7.0` | Native Qt 6 GUI framework for FRIDAY HUD. | Phase 1 / 5 |
| **`aiosqlite`** | `>=0.20.0` | Asynchronous SQLite driver for `proassist.db`. | Phase 1 |
| **`cryptography`** | `>=43.0.0` | Fernet symmetric encryption for credentials and profiles. | Phase 1 |
| **`psutil`** | `>=6.1.0` | Windows CPU, RAM, disk, and process telemetry. | Phase 1 |
| **`pyautogui`** | `>=0.9.54` | Desktop automation and screenshot capture. | Phase 1 |
| **`pyperclip`** | `>=1.9.0` | Windows clipboard read and write tools. | Phase 1 |
| **`Pillow`** | `>=10.0.0` | Screenshot image processing and UI asset rendering. | Phase 1 |
| **`httpx`** | `>=0.27.0` | Asynchronous HTTP client wrapped in `SafeHttpClient`. | Phase 1 / 6.0 |
| **`sounddevice`** | `>=0.5.0` | Low-latency 16kHz microphone audio streaming. | Phase 2 |
| **`soundfile`** | `>=0.14.0` | Audio file I/O and buffer decoding. | Phase 2 |
| **`webrtcvad-wheels`** | `>=2.0.10` | Real-time Voice Activity Detection (30ms frames). | Phase 2 |
| **`faster-whisper`** | `>=1.2.0` | Edge CPU speech-to-text transcription (CTranslate2). | Phase 2 |
| **`edge-tts`** | `>=7.2.0` | Free Microsoft neural TTS voice synthesis. | Phase 2 |
| **`piper-tts`** | `>=1.8.0` | Architecture support for local offline ONNX TTS voices. | Phase 2 |

---

## 2. Development & Testing Dependencies

| Package | Minimum Version | Purpose |
|---|---|---|
| **`pytest`** | `>=8.3.0` | Core test runner and assertion framework. |
| **`pytest-asyncio`** | `>=0.24.0` | Async event loop test fixtures and execution. |
| **`pytest-cov`** | `>=5.0.0` | Code coverage reporting. |

---

## 3. Optional & Future Phase Dependencies (Commented in requirements.txt)

- `# openwakeword>=0.6.0`: Reserved for Phase 3 custom acoustic model training.
- `# speechbrain>=1.0.0`: Reserved for future deep neural speaker d-vectors.
- `# ollama>=0.4.0`: Reserved for local LLM inference fallback.
- `# playwright>=1.48.0`: Reserved for Phase 8 / future browser automation.
- `# chromadb>=0.5.0`: Reserved for vector embeddings (if ever required).
