# Testing Strategy & Test Suite Reference — ProAssist AI

ProAssist AI enforces a rigorous automated testing discipline. Every architectural boundary, security policy, database migration, and agent tool is covered by comprehensive pytest test suites.

---

## 1. Test Suite Metrics

- **Current Verified Baseline**: **416 passed, 0 failed, 0 errors, 0 regressions**
- **Execution Time**: ~48.84 seconds on Windows 11 (Python 3.11.9)
- **Framework**: `pytest 9.1+` with `pytest-asyncio 0.24+` and `pytest-cov 5.0+`
- **Configuration**: `pytest.ini` with `asyncio_mode = auto`

---

## 2. Test File Breakdown

| Test File | Test Count | Scope & Verification Coverage |
|---|:---:|---|
| `tests/test_audio_capture.py` | 4 | Audio frame slicing, sample rate compliance, buffer management. |
| `tests/test_audio_devices.py` | 4 | Sounddevice device enumeration, default mic detection. |
| `tests/test_config.py` | 10 | Pydantic configuration validation, YAML loading, environment overrides. |
| `tests/test_conversational_planning.py`| 5 | Gemini plan generation, multi-step conversation turn handling. |
| `tests/test_file_agent.py` | 7 | FileAgent dispatch, execution isolation. |
| `tests/test_file_tools.py` | 12 | Concrete file tools: copy, move, rename, delete, search, zip. |
| `tests/test_language_detector.py` | 8 | Language detection across English, Hindi, and Kannada. |
| `tests/test_llm_provider.py` | 9 | Gemini provider formatting, mock provider, timeout recovery. |
| `tests/test_local_first_routing.py` | 18 | TaskRouter regex routing, direct tool execution without LLM. |
| `tests/test_orchestrator.py` | 12 | End-to-end orchestrator pipeline and state transitions. |
| `tests/test_phase4_security.py` | 5 | PlanValidator allowlists, forbidden operation rejection. |
| `tests/test_phase6_contacts.py` | 19 | Contact CRUD, alias resolution, relationship matching ("call mom"). |
| `tests/test_phase6_credentials.py` | 9 | WindowsCredentialStore, DPAPI encryption, fallback store. |
| `tests/test_phase6_database.py` | 3 | SQLite WAL mode, foreign keys, schema migrations. |
| `tests/test_phase6_idempotency.py` | 8 | Idempotency leasing, deduplication of mutating tool steps. |
| `tests/test_phase6_notes.py` | 19 | Notes CRUD, FTS5 BM25 ranked search, privacy snippet boundary. |
| `tests/test_phase6_provider_status.py` | 5 | ProviderStatus enum error mapping in SafeHttpClient. |
| `tests/test_phase6_search.py` | 24 | Search providers, query validation, secret redacting, citations, anti-browser. |
| `tests/test_phase6_security.py` | 8 | PermissionManager AuthLevels, SecretRedactor sanitization. |
| `tests/test_phase6_tasks_reminders.py` | 21 | Task CRUD, natural time parsing, recurring rules, scheduler. |
| `tests/test_phase6_weather.py` | 25 | Open-Meteo REST, SQLite cache (LIVE/CACHED/STALE/UNAVAIL), zero silent geolocation. |
| `tests/test_plan_validator.py` | 20 | Edge cases in plan validation: parameter types, max 10 steps. |
| `tests/test_security_chain_hardening.py`| 5 | Defense-in-depth pipeline execution without bypasses. |
| `tests/test_system_agent.py` | 7 | SystemAgent tool dispatch and OS telemetry. |
| `tests/test_system_tools.py` | 9 | Concrete system tools: volume, clipboard, process listing. |
| `tests/test_task_router.py` | 49 | 49 distinct voice/text routing patterns (EN, HI, KN). |
| `tests/test_text_to_speech.py` | 6 | Edge-TTS neural voice synthesis, language selection. |
| `tests/test_transcription_result.py` | 5 | STT transcription data models and confidence scores. |
| `tests/test_ui.py` | 15 | PySide6 widget instantiation, 8 assistant states, event bus. |
| `tests/test_voice_auth.py` | 6 | Voice biometrics, enrollment, cosine similarity threshold. |
| `tests/test_voice_auth_hardening.py` | 5 | Lockout policies (3 failures), session timeouts (300s). |
| `tests/test_voice_auth_security.py` | 5 | Rejection of un-enrolled speakers, profile encryption. |
| `tests/test_voice_pipeline.py` | 4 | Complete voice capture to STT transcription pipeline. |
| `tests/test_voice_pipeline_phase3.py` | 3 | Voice pipeline integration with speaker verification. |
| `tests/test_wake_word.py` | 3 | Wake-word engine interface and event triggering. |
| `tests/test_wake_word_engine.py` | 5 | OpenWakeWord frame processing and sensitivity threshold. |
| `tests/test_wake_word_hardening.py` | 9 | Truthful status reporting (`MODEL_MISSING`, `NOT_READY`). |
| `tests/test_phase6_calendar.py` | 25 | Calendar CRUD, conflict warnings, ambiguity resolution, local SQLite, Google/mock providers, idempotency leasing. |
| **Total** | **416** | **100% Passing** |

---

## 3. How to Run the Tests

Execute full test suite:
```powershell
python -m pytest -q
```

Execute only Phase 6.5 Web Search tests:
```powershell
python -m pytest -v tests/test_phase6_search.py
```
