# Voice Architecture & Speech Processing — ProAssist AI

ProAssist AI features a **local-first, privacy-preserving voice pipeline** supporting multi-language automatic speech recognition (ASR), voice activity detection (VAD), speaker biometric verification, and neural text-to-speech (TTS).

---

## 1. Complete Voice Pipeline Diagram

```
[ Microphone Input (16kHz 16-bit Mono PCM) ]
                       │
                       ▼
            [ AudioCaptureManager ] (sounddevice stream)
                       │
                       ▼
             [ WebRTCVADManager ] (30ms frames, aggressiveness=2)
                       │
             Speech Detected? ──NO──► Discard frame immediately
                       │
                      YES
                       ▼
                 [ AudioBuffer ] (held in RAM only)
                       │
       ┌───────────────┴───────────────┐
       ▼                               ▼
[ SpeakerVerifier ]          [ FasterWhisperSTT ]
(128-d Acoustic Centroid)    (Base model, int8 CPU)
       │                               │
Cosine Sim >= 0.70?            Transcribed String
├── YES ──► Session = AUTH             │
└── NO  ──► Session = PUBLIC           ▼
       │                     [ LanguageNormalizer ]
       └───────────────┬───── (EN / HI / KN Unicode normalization)
                       │               │
                       ▼               ▼
                 [ Orchestrator & TaskRouter ]
                       │
                       ▼
            [ TextToSpeechService ]
            (Edge-TTS Indian Neural Voices)
                       │
                       ▼
            [ Audio Output Device ]
```

---

## 2. Speech-to-Text (STT)

- **Engine**: `faster-whisper` based on OpenAI Whisper with CTranslate2 optimization.
- **Model Size**: `base` (optimized for local CPU inference, ~75MB footprint).
- **Quantization**: `int8` (fast inference with sub-second transcription on standard laptops).
- **Multilingual Support**:
  - **English (`en`)**: Native transcription.
  - **Hindi (`hi`)**: Transcribes in Devanagari script; normalized via `LanguageNormalizer`.
  - **Kannada (`kn`)**: Transcribes in Kannada script; normalized via `LanguageNormalizer`.

---

## 3. Text-to-Speech (TTS)

- **Primary Local-First Engine**: `EdgeTTSService` using Microsoft Edge neural speech endpoints (free, zero API key requirement).
- **Voices Configured**:
  - English: `en-IN-NeerjaNeural`
  - Hindi: `hi-IN-SwaraNeural`
  - Kannada: `kn-IN-SapnaNeural`
- **Fallback / Local Edge**: Architecture supports local `piper-tts` ONNX voices when offline models are downloaded.

---

## 4. Voice Biometrics & Authentication

- **Module**: `LocalVoiceAuthenticator` (`security/voice_auth/`).
- **Feature Extraction**: Generates a 128-dimensional acoustic feature vector combining spectral centroid, spectral rolloff, zero-crossing rate, and energy distributions.
- **Enrollment**:
  - Requires 3 clear spoken samples of at least 1.5 seconds each.
  - Verifies acoustic consistency between samples ($\ge 0.50$).
  - Encrypts the template vector and stores it in the `voice_profiles` table.
- **Verification**:
  - Computes cosine similarity between incoming utterance and enrolled owner profile.
  - **Threshold**: $\ge 0.70$ cosine similarity required to elevate session to `AUTHENTICATED`.
  - **Lockout**: 3 failed attempts lock authentication for 30 seconds.
  - **Timeout**: Verified session reverts to `PUBLIC` after 300 seconds of inactivity.

---

## 5. Wake Word Status & Truthful Engineering

> **CRITICAL ARCHITECTURAL FACT**:  
> The custom neural acoustic model for **"Hey ProAssist"** (`hey_proassist.onnx`) is **NOT READY** (untrained/missing from repository).

- ProAssist AI refuses to simulate fake wake-word readiness.
- The `OpenWakeWordEngine` explicitly returns:
  - `status_code = WakeWordStatus.MODEL_MISSING`
  - `is_ready = False`
- The PySide6 HUD displays: `[Hey ProAssist MODEL NOT READY]`.
- **Supported Interaction**: Users interact reliably via the **Push-to-Talk (PTT)** button or keyboard shortcuts.
