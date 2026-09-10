# Network & External Provider Architecture — ProAssist AI / Friday

ProAssist AI strictly limits external network communication. All outbound HTTP calls are wrapped inside an enforced safety client that enforces timeouts, redacts secrets from logs, and maps network errors into deterministic provider states.

---

## 1. The Safe HTTP Client (`SafeHttpClient`)

Located in `core/network_client.py`, `SafeHttpClient` wraps `httpx.AsyncClient`:

```python
class SafeHttpClient:
    def __init__(self, timeout: float = 10.0, max_retries: int = 2):
        self._timeout = httpx.Timeout(timeout, connect=8.0, read=timeout)
        self._max_retries = max_retries
        self._redactor = SecretRedactor()

    async def get(self, url: str, params: dict | None = None, headers: dict | None = None) -> HttpResponse:
        # 1. Strips sensitive tokens before logging
        clean_headers = self._redactor.sanitize_headers(headers)
        logger.debug(f"HTTP GET {url} | headers={clean_headers}")

        # 2. Strict timeout and error translation
        try:
            response = await self._client.get(url, params=params, headers=headers, timeout=self._timeout)
            return self._handle_response(response)
        except httpx.TimeoutException:
            return HttpResponse(status=ProviderStatus.TIMEOUT, error="Connection timed out")
        except httpx.RequestError as exc:
            return HttpResponse(status=ProviderStatus.NETWORK_ERROR, error=str(exc))
```

---

## 2. Unified Provider Status (`ProviderStatus`)

External provider outcomes map to a standard enumeration (`core/provider_status.py`):

| ProviderStatus | Description | Application Behavior |
|---|---|---|
| **`SUCCESS`** | Request completed successfully (HTTP 200). | Normal processing. |
| **`TIMEOUT`** | Connection or read timeout exceeded (8–10s). | Degrades to cache; notifies user without freezing UI. |
| **`NETWORK_ERROR`**| DNS failure, connection refused, or socket error. | Falls back to local offline mode. |
| **`RATE_LIMITED`** | HTTP 429 Too Many Requests. | Enforces exponential backoff; warns user cleanly. |
| **`SERVICE_ERROR`**| HTTP 500 / 502 / 503 upstream server error. | Reports server outage without exposing tracebacks. |
| **`PAYLOAD_ERROR`**| Malformed or unexpected JSON response structure. | Rejects payload; logs diagnostic failure. |
| **`UNAVAILABLE`** | Provider credentials missing or service offline. | Disables dependent tool; prompts user. |

---

## 3. Active External Providers Catalog

### 3.1 Open-Meteo REST API (`weather/providers/open_meteo.py`)
- **Endpoints**:
  - Forecast: `https://api.open-meteo.com/v1/forecast`
  - Geocoding: `https://geocoding-api.open-meteo.com/v1/search`
- **Authentication**: None (free open-access API).
- **Privacy Guarantee**: Zero user tracking, zero device identifiers sent.
- **Cache TTL**: 30 minutes for current weather, 3 hours for multi-day forecasts.

### 3.2 Google Gemini API (`core/llm/gemini_provider.py`)
- **Endpoint**: Google Generative AI API (`gemini-2.5-flash`).
- **Authentication**: API key stored in Windows Credential Manager or `GEMINI_API_KEY` environment variable.
- **Payload Privacy**: Passes through `SecretRedactor` before transmission. Personal database is never dumped in bulk.

### 3.3 Microsoft Edge TTS (`voice/tts_provider.py`)
- **Endpoint**: WSS / HTTPS speech synthesis endpoint.
- **Authentication**: Ephemeral websocket handshake (free).
- **Usage**: Converts response strings to high-quality Indian English, Hindi, and Kannada audio.
