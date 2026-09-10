# Network & External Provider Architecture — ProAssist AI

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

    async def get(self, url: str, params: dict | None = None, headers: dict | None = None) -> tuple[ProviderStatus, Any, str | None]:
        ...
```

---

## 2. Unified Provider Status (`ProviderStatus`)

External provider outcomes map to a standard enumeration (`core/provider_status.py`):

| ProviderStatus | Description | Application Behavior |
|---|---|---|
| **`SUCCESS`** | Request completed successfully (HTTP 200). | Normal processing. |
| **`TIMEOUT`** | Connection or read timeout exceeded (8–10s). | Degrades to cache/fallback; notifies user without freezing UI. |
| **`FAILURE`** | Network error, DNS failure, or 5xx server error. | Falls back gracefully or alerts user. |
| **`AUTHENTICATION_ERROR`** | Missing or invalid API key / token (HTTP 401). | Alerts user to configure credentials. |
| **`AUTHORIZATION_ERROR`**  | Insufficient scopes or forbidden resource (HTTP 403). | Denies request with clear explanation. |
| **`RATE_LIMITED`** | HTTP 429 Too Many Requests. | Warns user cleanly; avoids hammering provider. |
| **`NOT_FOUND`** | Resource missing (HTTP 404). | Graceful not-found report. |
| **`CONFLICT`** | Version or state collision (HTTP 409). | Prompts user or manages idempotency. |
| **`UNKNOWN`** | Outcome indeterminate. | Treats as potential failure. |

---

## 3. Active External Providers Catalog

### 3.1 Open-Meteo REST API (`weather/providers/open_meteo.py`)
- **Endpoints**:
  - Forecast: `https://api.open-meteo.com/v1/forecast`
  - Geocoding: `https://geocoding-api.open-meteo.com/v1/search`
- **Authentication**: None (free open-access API).
- **Privacy Guarantee**: Zero user tracking, zero device identifiers sent.
- **Cache TTL**: 30 minutes for current weather, 3 hours for multi-day forecasts.

### 3.2 Web Search Providers (`search/providers/`)
- **`DuckDuckGoSearchProvider`** (`search/providers/duckduckgo.py`):
  - Endpoints: `https://lite.duckduckgo.com/lite/` and `https://api.duckduckgo.com/`
  - Authentication: None (zero API key, privacy-friendly).
  - Privacy Guarantee: Zero user tracking, zero personal information transmitted.
- **`BraveSearchProvider`** (`search/providers/brave.py`):
  - Endpoint: `https://api.search.brave.com/res/v1/web/search`
  - Authentication: API key (`X-Subscription-Token`) loaded from `CredentialStore`.
  - Rate Limiting: Handles 429 natively as `ProviderStatus.RATE_LIMITED`.
- **`MockSearchProvider`** (`search/providers/mock.py`):
  - Deterministic canned results and simulated error states for testing and offline runs.

### 3.3 Calendar Providers (`calendar_integration/providers/`)
- **`LocalCalendarProvider`** (`calendar_integration/providers/local.py`):
  - 100% offline local SQLite implementation.
  - Zero outbound networking, instant local execution.
- **`MockCalendarProvider`** (`calendar_integration/providers/mock.py`):
  - Deterministic in-memory provider for unit tests, offline development, and error simulation (`TIMEOUT`, `FAILURE`, `RATE_LIMITED`, `AUTHENTICATION_ERROR`).
- **`GoogleCalendarProvider`** (`calendar_integration/providers/google.py`):
  - REST endpoint: `https://www.googleapis.com/calendar/v3`
  - Authentication: Bearer token loaded from `CredentialStore`.
  - Zero Fake OAuth: If unconfigured, truthfully returns `ProviderStatus.AUTHENTICATION_ERROR` without simulating credentials.

## Email Providers (`email_integration/providers/`)
- `LocalEmailProvider`: Completely offline local SQLite provider.
- `MockEmailProvider`: In-memory deterministic provider with configurable fault injection (`simulate_timeout`, `simulate_rate_limit`, `simulate_auth_error`, `simulate_unknown_status`).
- `GoogleEmailProvider`: Gmail REST API provider using `SafeHttpClient`. Enforces Zero Fake OAuth — returns `ProviderStatus.AUTHENTICATION_ERROR` if unconfigured.
- Invariants: Zero ambient networking, no background polling or sync.
