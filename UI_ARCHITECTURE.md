# UI Architecture — ProAssist AI / Friday

The user interface of ProAssist AI is a **futuristic, floating desktop HUD** developed natively in **PySide6 (Qt 6.7+)**. Inspired by the Stark Industries FRIDAY / JARVIS aesthetic, it emphasizes unobtrusive desktop presence, dark neon telemetry, dynamic radial waveforms, and strict state reflection.

---

## 1. Floating Desktop Interaction Modes

ProAssist AI operates in three dynamically switchable display modes (`ui/main_window.py`):

```
+───────────────────────+        +───────────────────────────────────+
|     [ ORB MODE ]      |        |           [ PANEL MODE ]          |
|  Small floating widget|        |  Semi-expanded HUD                |
|  - Breathing AI Core  | ◄────► |  - Top Telemetry Bar              |
|  - Orbital Rings      |        |  - Holographic AI Visualizer      |
|  - Desktop Pinning    |        |  - Quick Action Dock              |
+───────────────────────+        +─────────────────┬─────────────────+
                                                   │
                                                   ▼
                                 +───────────────────────────────────+
                                 |         [ EXPANDED MODE ]         |
                                 |  Full Assistant Window            |
                                 |  - Full Conversation History      |
                                 |  - Interactive Confirmation Cards |
                                 |  - Detailed Settings Panel        |
                                 |  - Execution Step Progress        |
                                 +───────────────────────────────────+
```

1. **`ORB` Mode**: Unobtrusive floating circular HUD (100x100px) that sits persistently on top of other Windows desktop applications. Features rotating gyroscopic orbital rings and a pulsating core indicating current status.
2. **`PANEL` Mode**: Expands to reveal the top telemetry bar, audio-reactive waveform visualizer, and quick voice/text input dock.
3. **`EXPANDED` Mode**: Displays the multi-turn conversational scroll area, execution step cards, and full application settings.
4. **System Tray Integration**: Provides quick menu access to toggle modes, trigger Push-to-Talk, view status, or exit cleanly.

---

## 2. The 8 Assistant UI States

UI state transitions are driven strictly by real backend events emitted from `Orchestrator`, `VoicePipeline`, and `ExecutionEngine`:

| State | Primary Accent Color | Pulse Rate | Telemetry Identifier | Description |
|---|---|:---:|:---:|---|
| **`IDLE`** | Cyan (`#00f0ff`) | 0.7 Hz | `SYS_IDLE_READY` | Calm breathing pulse; assistant ready for voice or text. |
| **`LISTENING`** | Deep Cyan (`#00c8ff`) | 1.5 Hz | `MIC_STREAM_ACTIVE` | Active microphone stream; radial bars react to real audio RMS. |
| **`AUTHENTICATING`** | Amber (`#ffb703`) | 2.2 Hz | `VOICE_BIO_SCAN` | Biometric speaker verification centroid comparison. |
| **`THINKING`** | Violet (`#bf5af2`) | 2.8 Hz | `NEURAL_REASONING` | Gemini task planning and schema validation. |
| **`EXECUTING`** | Orange (`#ff5e00`) | 2.0 Hz | `TACTICAL_EXECUTION` | Step-by-step tool execution via domain agents. |
| **`SPEAKING`** | Soft Cyan (`#58a6ff`) | 1.2 Hz | `TTS_AUDIO_OUT` | Synthesizing and streaming speech output. |
| **`CONFIRMATION_REQUIRED`**| Crimson (`#ff3b30`) | 3.5 Hz | `SECURITY_HALT` | High-risk action gated; interactive modal active. |
| **`ERROR`** | Red (`#ff2a55`) | 0.0 Hz | `SYSTEM_FAULT` | Action failed, timeout, or subsystem error. |

---

## 3. Core UI Components

### 3.1 Top Telemetry Bar (`ui/widgets/top_bar.py`)
Provides constant visibility into system state without opening settings:
- **Title Badge**: `PROASSIST AI // FRIDAY`
- **Online Indicator**: `[● ONLINE]`
- **Authentication Badge**: `[AUTH: PUBLIC]` or `[AUTH: OWNER]`
- **Processing Location**: `[LOCAL]` (0ms cloud latency) or `[CLOUD]` (Gemini engaged)
- **Wake Word Telemetry**: `[Hey ProAssist MODEL NOT READY]` (truthful status display)

### 3.2 Holographic AI Visualizer (`ui/widgets/ai_visualizer.py`)
- Radial gradient central core that pulses smoothly according to state frequency.
- Three concentric gyroscopic orbital rings rotating at mathematical phase offsets.
- Dynamic radial waveform ticks around the perimeter that react in real-time to the microphone's Root-Mean-Square (RMS) audio energy.

### 3.3 Interactive Confirmation Card (`ui/widgets/confirmation_card.py`)
- Intercepts `HIGH`-risk actions before execution.
- Emphasized crimson border (`#ff3b30`) and warning iconography.
- Displays target tool name and exact parameters (e.g. `delete_files: report.pdf`).
- **Focus Safety**: The `CANCEL` button receives default focus to prevent accidental enter-key executions.
- Includes a 30-second countdown timer; if unattended, defaults to `DENY`.

---

## 4. Live Application vs Screenshot/Demo Behavior

- **Live Behavior**: The visualizer pulses based on actual QTimer paint events, and radial tick heights are calculated from real audio buffers captured by `AudioCaptureManager`.
- **Telemetry Truthfulness**: If `GEMINI_API_KEY` is absent, the HUD displays `[LOCAL]` and warns if an ambiguous request cannot be planned. If `hey_proassist.onnx` is missing, the HUD explicitly shows `[Hey ProAssist MODEL NOT READY]`. The UI never displays fake simulated readiness.
