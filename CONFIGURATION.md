# Configuration Guide — ProAssist AI / Friday

ProAssist AI follows a **hierarchical configuration architecture**:
1. Defaults are defined in `config/config.yaml`.
2. Secrets and local overrides are specified in `.env`.
3. System environment variables take top priority.
All values are validated at startup by Pydantic in `config/settings.py`.

---

## 1. Configuration File: `config/config.yaml`

```yaml
app:
  name: ProAssist AI
  version: "0.1.0"
  env: development          # development | production | testing
  log_level: DEBUG          # DEBUG | INFO | WARNING | ERROR | CRITICAL
  data_dir: null            # null defaults to ~/.proassist
  log_dir: null             # null defaults to <project_root>/logs

database:
  filename: proassist.db

logging:
  rotation: "10 MB"
  retention: "30 days"

languages:
  supported:
    - code: en
      name: English
      whisper_code: en
      tts_voice_edge: en-IN-NeerjaNeural
    - code: hi
      name: Hindi
      whisper_code: hi
      tts_voice_edge: hi-IN-SwaraNeural
    - code: kn
      name: Kannada
      whisper_code: kn
      tts_voice_edge: kn-IN-SapnaNeural
  default: en

wake_word:
  enabled: false
  phrase: hey_proassist
  threshold: 0.5
  cooldown_seconds: 2.0

voice_auth:
  enabled: true
  provider: local_spectral_centroid
  samples_required: 3
  min_duration_seconds: 1.5
  verification_threshold: 0.70
  max_failed_attempts: 3
  lockout_seconds: 30
  session_timeout_seconds: 300
  store_raw_audio: false

voice:
  sample_rate: 16000
  channels: 1
  chunk_ms: 30
  max_recording_seconds: 15.0
  min_speech_seconds: 0.3
  silence_timeout_seconds: 1.5
  tts_engine: local_first

stt:
  provider: faster_whisper
  model_size: base          # tiny | base | small
  device: cpu               # cpu | cuda
  compute_type: int8

llm:
  provider: gemini          # gemini | claude | mock
  gemini_model: gemini-2.5-flash
  temperature: 0.1
  timeout_seconds: 15.0
  max_tokens: 1024

weather:
  provider: open-meteo
  temperature_unit: celsius # celsius | fahrenheit
  current_cache_ttl_seconds: 1800  # 30 min
  forecast_cache_ttl_seconds: 10800 # 3 hours
  request_timeout_seconds: 10.0
```

---

## 2. Environment Variables Template: `.env.example`

```bash
# ProAssist AI — Environment Variables
# Copy to .env and configure local values. NEVER commit .env to Git!

PROASSIST_ENV=development
PROASSIST_LOG_LEVEL=DEBUG

# Symmetric Master Key (Auto-generated on first run if empty)
PROASSIST_ENCRYPTION_KEY=<FERNET_KEY_HERE>

# Cloud LLM Key (Google Gemini 2.5 Flash)
GEMINI_API_KEY=<YOUR_GEMINI_API_KEY_HERE>

# Alternative Cloud Provider
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY_HERE>
```
