# Known Limitations & Technical Boundaries — ProAssist AI / Friday

This document provides a **truthful, candid assessment** of the current technical boundaries and unfinished components of ProAssist AI as of Phase 6.5.

---

## 1. Voice & Wake Word Subsystem

1. **"Hey ProAssist" Custom Wake Word Untrained**:
   - The neural acoustic model (`hey_proassist.onnx`) has **NOT been trained**.
   - The engine correctly reports `WakeWordStatus.MODEL_MISSING` and displays `[Hey ProAssist MODEL NOT READY]` on the UI.
   - Users must trigger interactions via Push-to-Talk (PTT) or UI button.
2. **Speaker Verification Algorithm**:
   - The current biometric engine uses local spectral centroid and acoustic feature vectors rather than deep neural d-vectors (e.g. SpeechBrain ECAPA-TDNN).
   - It reliably rejects different voices under quiet conditions, but is sensitive to significant ambient background noise.
   - It is not formally certified for Presentation Attack Detection (PAD).

---

## 2. Platform & OS Boundaries

1. **Windows 11 Exclusive**:
   - The application relies directly on Windows-specific mechanisms:
     - Windows DPAPI (`CryptProtectData`) and Windows Credential Manager.
     - PySide6 Windows frameless window flags (`Qt.FramelessWindowHint`).
     - Windows process and volume controls (`psutil`, `pyautogui`, Windows media keys).
   - Running on Linux or macOS is not supported in the current architecture.

---

## 3. Intelligence & Networking Boundaries

1. **Cloud LLM Required for Conversational Chat**:
   - Free-form, multi-step natural language reasoning requires an active internet connection and a valid `GEMINI_API_KEY`.
   - If offline or unconfigured, the assistant continues executing all 52 deterministic direct tools locally, but cannot answer open-ended conversational questions.
2. **Strict Non-Browser Web Search**:
   - Web search is powered by DuckDuckGo and Brave APIs returning factual text snippets and URLs.
   - Arbitrary webpage scraping, clicking elements, submitting forms, running JavaScript, or downloading arbitrary binaries are strictly forbidden by architectural design.
