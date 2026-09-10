# Development Roadmap (Phases 6.5 – 6.10) — ProAssist AI / Friday

This document outlines the **architectural blueprint for future implementation phases**. Development must proceed sequentially without skipping phases or prematurely building future infrastructure.

---

## 1. Phase 6.5 — Web Search & Information Retrieval (COMPLETE)

### Objective
Provide ProAssist AI with the ability to perform targeted, real-time web searches and extract clean factual information without turning the LLM into an unrestricted web browser.

### Key Architectural Constraints
1. **Provider Abstraction**: Create `SearchProvider` protocol with concrete implementations:
   - `DuckDuckGoSearchProvider` (privacy-friendly, no API key).
   - `BraveSearchProvider` / `GoogleSearchProvider` (API key driven).
   - `MockSearchProvider` (offline testing).
2. **Safe HTTP Client Only**: All search queries must route through `SafeHttpClient`.
3. **Strict URL & Content Restrictions**:
   - Prohibit arbitrary HTML execution, JavaScript execution, or downloading binaries.
   - Extract plain text snippets only.
4. **Truthful Citations**: Returned results must include clean source titles and URLs.
5. **Security**: Search tools registered under `web_search_agent` with `LOW` risk and `AUTHENTICATED` requirement.

---

## 2. Phase 6.6 — Calendar Integration (NEXT AUTHORIZED PHASE)

### Objective
Integrate local SQLite calendar event scheduling with optional cloud calendar synchronization (Google Calendar / Microsoft Outlook via OAuth).

### Key Architectural Constraints
1. Local-first SQLite calendar store (`calendar_events` table).
2. Connected account metadata stored in `oauth_accounts`.
3. Refresh tokens stored exclusively in `WindowsCredentialStore`.
4. Conflict detection: Warn user before creating overlapping meetings.
5. Multi-participant invites require explicit user confirmation.

---

## 3. Phase 6.7 — Email Integration

### Objective
Read, search, draft, and send emails via IMAP/SMTP and Gmail/Outlook APIs.

### Key Architectural Constraints
1. Reading and searching emails is `LOW` risk (`AUTHENTICATED`).
2. Drafting emails is `MEDIUM` risk (`AUTHENTICATED`).
3. **MANDATORY CONFIRMATION**: Sending an email (`send_email`) is `HIGH` risk and strictly requires `AuthLevel.CONFIRMATION`. The confirmation card must display the recipient, subject, and body snippet.

---

## 4. Phase 6.8 — WhatsApp & Instant Messaging

### Objective
Send messages and notifications to resolved contacts via WhatsApp Web / Cloud API.

### Key Architectural Constraints
1. Integrates with `ContactResolver` from Phase 6.1 to resolve recipient phone numbers.
2. Draft-and-confirm workflow: Displays message preview before dispatch.
3. Sending messages is `HIGH` risk (`CONFIRMATION`).

---

## 5. Phase 6.9 — Cross-Service Workflows

### Objective
Coordinate cross-domain workflows linking Contacts, Tasks, Notes, Calendar, and Communication without expanding individual tool permissions.

---

## 6. Phase 6.10 — Reliability & Security Hardening

### Objective
Systemic resilience, memory leak elimination, offline fallback stress tests, and distribution preparation.
