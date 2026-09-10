# Development Roadmap (Phases 6.6 – 6.10) — ProAssist AI

This document outlines the **architectural blueprint for future implementation phases**. Development must proceed sequentially without skipping phases or prematurely building future infrastructure.

---

## 1. Phase 6.5 — Web Search & Information Retrieval (COMPLETE)

### Objective
Provide ProAssist AI with the ability to perform targeted, real-time web searches and extract clean factual information without turning the LLM into an unrestricted web browser.

---

## 2. Phase 6.6 — Calendar Integration (COMPLETE)

### Objective
Integrate local SQLite calendar event scheduling with optional cloud calendar synchronization (Google Calendar / Microsoft Outlook via OAuth).

### Key Architectural Deliverables
1. **Local SQLite Storage**: Complete local offline calendar event management in `calendars` and `calendar_events` tables.
2. **Provider Abstraction**: Pluggable `CalendarProvider` supporting `LocalCalendarProvider`, `MockCalendarProvider`, and `GoogleCalendarProvider`.
3. **Zero Fake OAuth**: Truthful `AUTHENTICATION_ERROR` reporting when credentials are missing; zero token fabrication.
4. **Conflict Detection**: Non-destructive `CalendarConflictWarning` prompting rather than silent overriding or auto-rejection.
5. **Agent & Tools**: `CalendarAgent` exposing 7 tools with strict risk classification (`delete_event` requires `CONFIRMATION`).

---

## 3. Phase 6.7 — Email Integration (NEXT AUTHORIZED PHASE)

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
