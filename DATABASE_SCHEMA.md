# Database Schema Reference — ProAssist AI

This document is the authoritative schema reference for all **21 tables** in `proassist.db` as defined in `memory/database.py`.

---

## 1. System & Audit Tables

### `audit_logs`
Append-only immutable record of all security decisions and tool executions.
```sql
CREATE TABLE audit_logs (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp       TEXT    NOT NULL, -- UTC ISO-8601
    authenticated   INTEGER NOT NULL DEFAULT 0, -- 1 if voice verified, 0 if public
    user_command    TEXT,             -- Redacted user utterance
    agent           TEXT,             -- Subsystem agent name
    tool            TEXT,             -- Tool invoked
    action          TEXT,             -- Action description
    parameters      TEXT,             -- Sanitized JSON parameters
    result_status   TEXT,             -- SUCCESS | FAILURE | DENIED | CANCELLED
    result_detail   TEXT,             -- Human-readable outcome summary
    error_message   TEXT              -- Exception trace if failure
);
CREATE INDEX idx_audit_timestamp ON audit_logs(timestamp);
```

### `user_memory`
Persistent key-value memory store for user preferences and facts.
```sql
CREATE TABLE user_memory (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    key         TEXT    NOT NULL UNIQUE, -- e.g. "project_guide"
    value       TEXT    NOT NULL,        -- e.g. "Chaitra"
    category    TEXT    NOT NULL DEFAULT 'general', -- contact | preference | fact
    created_at  TEXT    NOT NULL,
    updated_at  TEXT    NOT NULL
);
CREATE INDEX idx_user_memory_key ON user_memory(key);
```

### `task_history`
High-level execution records for recent user commands.
```sql
CREATE TABLE task_history (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp       TEXT    NOT NULL,
    command         TEXT    NOT NULL,
    agent           TEXT,
    action          TEXT,
    status          TEXT    NOT NULL, -- SUCCESS | FAILURE | CANCELLED
    duration_ms     INTEGER,
    result_summary  TEXT
);
CREATE INDEX idx_task_history_ts ON task_history(timestamp);
```

### `conversations` & `conversation_turns`
Stores multi-turn conversational session history.
```sql
CREATE TABLE conversations (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    session_id  TEXT    NOT NULL UNIQUE, -- UUID
    started_at  TEXT    NOT NULL,
    ended_at    TEXT,
    language    TEXT    NOT NULL DEFAULT 'en'
);

CREATE TABLE conversation_turns (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    session_id      TEXT    NOT NULL,
    sequence        INTEGER NOT NULL,
    role            TEXT    NOT NULL, -- user | assistant | system
    content         TEXT    NOT NULL,
    language        TEXT,
    timestamp       TEXT    NOT NULL,
    FOREIGN KEY (session_id) REFERENCES conversations(session_id)
);
CREATE INDEX idx_conv_turns_session ON conversation_turns(session_id);
```

### `voice_profiles`
Encrypted voice biometric templates for speaker verification.
```sql
CREATE TABLE voice_profiles (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id         TEXT    NOT NULL UNIQUE DEFAULT 'owner',
    provider        TEXT    NOT NULL, -- local_spectral_centroid
    model_version   TEXT    NOT NULL,
    embedding_data  BLOB    NOT NULL, -- Fernet-encrypted 128-d acoustic vector
    num_samples     INTEGER NOT NULL DEFAULT 1,
    created_at      TEXT    NOT NULL,
    updated_at      TEXT    NOT NULL
);
CREATE INDEX idx_voice_profile_user ON voice_profiles(user_id);
```

---

## 2. Integration & Security Tables

### `oauth_accounts`
OAuth metadata for connected external accounts (Phase 6.0). Zero plaintext secrets.
```sql
CREATE TABLE oauth_accounts (
    service              TEXT    PRIMARY KEY, -- e.g. "google_calendar"
    account_email        TEXT    NOT NULL,
    provider             TEXT    NOT NULL,
    scopes               TEXT    NOT NULL,    -- Comma-separated list
    expires_at           TEXT,                -- UTC ISO timestamp
    credential_reference TEXT    NOT NULL,    -- Windows Credential Vault key
    created_at           TEXT    NOT NULL,
    updated_at           TEXT    NOT NULL
);
CREATE INDEX idx_oauth_service ON oauth_accounts(service);
```

### `idempotency_records`
Atomic leasing and execution deduplication for multi-step tasks.
```sql
CREATE TABLE idempotency_records (
    idempotency_key TEXT    PRIMARY KEY, -- operation_id:step_id
    operation_id    TEXT    NOT NULL,
    step_id         TEXT    NOT NULL,
    tool_name       TEXT    NOT NULL,
    status          TEXT    NOT NULL, -- PENDING | IN_PROGRESS | SUCCEEDED | FAILED
    result_summary  TEXT,
    created_at      TEXT    NOT NULL,
    updated_at      TEXT    NOT NULL
);
CREATE INDEX idx_idempotency_op ON idempotency_records(operation_id);
CREATE INDEX idx_idempotency_status ON idempotency_records(status);
```

---

## 3. Contacts Subsystem Tables (Phase 6.1)

```sql
CREATE TABLE contacts (
    id               TEXT PRIMARY KEY, -- UUID
    display_name     TEXT NOT NULL,
    normalized_name  TEXT NOT NULL,
    notes            TEXT,
    created_at       TEXT NOT NULL,
    updated_at       TEXT NOT NULL
);
CREATE INDEX idx_contacts_norm_name ON contacts(normalized_name);

CREATE TABLE contact_aliases (
    id               INTEGER PRIMARY KEY AUTOINCREMENT,
    contact_id       TEXT NOT NULL,
    alias            TEXT NOT NULL,
    normalized_alias TEXT NOT NULL,
    created_at       TEXT NOT NULL,
    FOREIGN KEY (contact_id) REFERENCES contacts(id) ON DELETE CASCADE
);
CREATE INDEX idx_contact_aliases_norm ON contact_aliases(normalized_alias);

CREATE TABLE contact_relationships (
    id                      INTEGER PRIMARY KEY AUTOINCREMENT,
    contact_id              TEXT NOT NULL,
    relationship            TEXT NOT NULL, -- e.g. "mother", "manager"
    normalized_relationship TEXT NOT NULL,
    created_at              TEXT NOT NULL,
    FOREIGN KEY (contact_id) REFERENCES contacts(id) ON DELETE CASCADE
);
CREATE INDEX idx_contact_rel_norm ON contact_relationships(normalized_relationship);

CREATE TABLE contact_phones (
    id               INTEGER PRIMARY KEY AUTOINCREMENT,
    contact_id       TEXT NOT NULL,
    phone_number     TEXT NOT NULL,
    normalized_phone TEXT NOT NULL,
    label            TEXT NOT NULL DEFAULT 'mobile',
    is_primary       INTEGER NOT NULL DEFAULT 1,
    created_at       TEXT NOT NULL,
    FOREIGN KEY (contact_id) REFERENCES contacts(id) ON DELETE CASCADE
);
CREATE INDEX idx_contact_phones_norm ON contact_phones(normalized_phone);

CREATE TABLE contact_emails (
    id               INTEGER PRIMARY KEY AUTOINCREMENT,
    contact_id       TEXT NOT NULL,
    email_address    TEXT NOT NULL,
    normalized_email TEXT NOT NULL,
    label            TEXT NOT NULL DEFAULT 'personal',
    is_primary       INTEGER NOT NULL DEFAULT 1,
    created_at       TEXT NOT NULL,
    FOREIGN KEY (contact_id) REFERENCES contacts(id) ON DELETE CASCADE
);
CREATE INDEX idx_contact_emails_norm ON contact_emails(normalized_email);
```

---

## 4. Tasks & Reminders Tables (Phase 6.2)

```sql
CREATE TABLE tasks (
    id               TEXT PRIMARY KEY, -- UUID
    title            TEXT NOT NULL,
    normalized_title TEXT NOT NULL,
    description      TEXT,
    status           TEXT NOT NULL DEFAULT 'PENDING', -- PENDING | COMPLETED | CANCELLED | OVERDUE
    priority         TEXT NOT NULL DEFAULT 'medium',  -- low | medium | high | urgent
    due_at           TEXT,                            -- UTC ISO timestamp
    created_at       TEXT NOT NULL,
    updated_at       TEXT NOT NULL,
    completed_at     TEXT,
    cancelled_at     TEXT
);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_due_at ON tasks(due_at);
CREATE INDEX idx_tasks_norm_title ON tasks(normalized_title);

CREATE TABLE reminders (
    id                 TEXT PRIMARY KEY, -- UUID
    title              TEXT NOT NULL,
    normalized_title   TEXT NOT NULL,
    message            TEXT,
    scheduled_at       TEXT NOT NULL, -- UTC ISO timestamp
    timezone           TEXT NOT NULL DEFAULT 'UTC',
    recurrence_rule    TEXT,          -- daily | weekly:<day> | weekdays | null
    status             TEXT NOT NULL DEFAULT 'SCHEDULED', -- SCHEDULED | TRIGGERED | COMPLETED | CANCELLED | MISSED | FAILED
    created_at         TEXT NOT NULL,
    updated_at         TEXT NOT NULL,
    last_triggered_at  TEXT,
    next_trigger_at    TEXT,
    completed_at       TEXT,
    cancelled_at       TEXT,
    associated_task_id TEXT,
    FOREIGN KEY (associated_task_id) REFERENCES tasks(id) ON DELETE SET NULL
);
CREATE INDEX idx_reminders_status ON reminders(status);
CREATE INDEX idx_reminders_sched ON reminders(scheduled_at);
CREATE INDEX idx_reminders_next ON reminders(next_trigger_at);
CREATE INDEX idx_reminders_norm_title ON reminders(normalized_title);
```

---

## 5. Notes & FTS5 Tables (Phase 6.3)

```sql
CREATE TABLE notes (
    id                 TEXT PRIMARY KEY, -- UUID
    title              TEXT NOT NULL,
    content            TEXT NOT NULL,
    normalized_title   TEXT NOT NULL,
    normalized_content TEXT NOT NULL,
    tags               TEXT NOT NULL DEFAULT '[]', -- JSON array of strings
    status             TEXT NOT NULL DEFAULT 'ACTIVE', -- ACTIVE | ARCHIVED | DELETED
    created_at         TEXT NOT NULL,
    updated_at         TEXT NOT NULL
);
CREATE INDEX idx_notes_status ON notes(status);
CREATE INDEX idx_notes_norm_title ON notes(normalized_title);
CREATE INDEX idx_notes_created_at ON notes(created_at);

-- Virtual full-text search table
CREATE VIRTUAL TABLE notes_fts USING fts5(
    note_id UNINDEXED,
    title,
    content,
    tags
);

-- Automated sync triggers
CREATE TRIGGER notes_ai AFTER INSERT ON notes BEGIN
    INSERT INTO notes_fts(note_id, title, content, tags)
    VALUES (new.id, new.title, new.content, new.tags);
END;

CREATE TRIGGER notes_ad AFTER DELETE ON notes BEGIN
    DELETE FROM notes_fts WHERE note_id = old.id;
END;

CREATE TRIGGER notes_au AFTER UPDATE ON notes BEGIN
    DELETE FROM notes_fts WHERE note_id = old.id;
    INSERT INTO notes_fts(note_id, title, content, tags)
    VALUES (new.id, new.title, new.content, new.tags);
END;
```

---

## 6. Weather & Location Tables (Phase 6.4)

```sql
CREATE TABLE weather_locations (
    id              TEXT PRIMARY KEY, -- UUID
    display_name    TEXT NOT NULL,
    normalized_name TEXT NOT NULL,
    latitude        REAL NOT NULL,
    longitude       REAL NOT NULL,
    timezone        TEXT NOT NULL DEFAULT 'UTC',
    is_default      INTEGER NOT NULL DEFAULT 0,
    created_at      TEXT NOT NULL,
    updated_at      TEXT NOT NULL
);
CREATE INDEX idx_weather_loc_norm ON weather_locations(normalized_name);
CREATE INDEX idx_weather_loc_default ON weather_locations(is_default);

CREATE TABLE weather_cache (
    id           TEXT PRIMARY KEY, -- UUID
    location_key TEXT NOT NULL,
    cache_type   TEXT NOT NULL DEFAULT 'current', -- 'current' | 'forecast'
    provider     TEXT NOT NULL DEFAULT 'open-meteo',
    fetched_at   TEXT NOT NULL,
    expires_at   TEXT NOT NULL,
    payload_json TEXT NOT NULL,
    created_at   TEXT NOT NULL,
    updated_at   TEXT NOT NULL
);
CREATE INDEX idx_weather_cache_key ON weather_cache(location_key, cache_type);
CREATE INDEX idx_weather_cache_exp ON weather_cache(expires_at);
```

---

## 7. Calendar Tables (Phase 6.6)

### `calendars`
Local and synchronized external calendar accounts and collections.
```sql
CREATE TABLE calendars (
    id                   TEXT    PRIMARY KEY, -- UUID string
    account_id           TEXT    NOT NULL DEFAULT 'local',
    provider             TEXT    NOT NULL DEFAULT 'local', -- 'local' | 'mock' | 'google' | 'outlook'
    name                 TEXT    NOT NULL,
    normalized_name      TEXT    NOT NULL,
    description          TEXT,
    timezone             TEXT    NOT NULL DEFAULT 'UTC',
    is_primary           INTEGER NOT NULL DEFAULT 1,
    is_read_only         INTEGER NOT NULL DEFAULT 0,
    sync_token           TEXT,
    created_at           TEXT    NOT NULL,
    updated_at           TEXT    NOT NULL
);
CREATE INDEX idx_calendars_account    ON calendars(account_id);
CREATE INDEX idx_calendars_provider   ON calendars(provider);
CREATE INDEX idx_calendars_norm_name  ON calendars(normalized_name);
```

### `calendar_events`
Scheduled calendar events with start/end UTC ISO timestamps and metadata.
```sql
CREATE TABLE calendar_events (
    id                   TEXT    PRIMARY KEY, -- UUID string
    calendar_id          TEXT    NOT NULL,
    title                TEXT    NOT NULL,
    normalized_title     TEXT    NOT NULL,
    description          TEXT,
    location             TEXT,
    start_time           TEXT    NOT NULL, -- UTC ISO string
    end_time             TEXT    NOT NULL, -- UTC ISO string
    timezone             TEXT    NOT NULL DEFAULT 'UTC',
    is_all_day           INTEGER NOT NULL DEFAULT 0,
    recurrence_rule      TEXT,
    attendees            TEXT    NOT NULL DEFAULT '[]', -- JSON list of strings
    status               TEXT    NOT NULL DEFAULT 'CONFIRMED', -- 'CONFIRMED' | 'TENTATIVE' | 'CANCELLED'
    provider_event_id    TEXT,
    etag                 TEXT,
    created_at           TEXT    NOT NULL,
    updated_at           TEXT    NOT NULL,
    FOREIGN KEY (calendar_id) REFERENCES calendars(id) ON DELETE CASCADE
);
CREATE INDEX idx_cal_events_cal_id    ON calendar_events(calendar_id);
CREATE INDEX idx_cal_events_start     ON calendar_events(start_time);
CREATE INDEX idx_cal_events_end       ON calendar_events(end_time);
CREATE INDEX idx_cal_events_norm_titl ON calendar_events(normalized_title);
CREATE INDEX idx_cal_events_status    ON calendar_events(status);
```

### 11. `email_accounts` Table
Stores configured email accounts and provider metadata.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PRIMARY KEY | Unique account identifier (e.g. "primary") |
| `email_address` | TEXT | NOT NULL | User email address |
| `display_name` | TEXT | | Friendly display name |
| `provider_type` | TEXT | NOT NULL DEFAULT 'local' | 'local', 'google', or 'mock' |
| `is_default` | INTEGER | NOT NULL DEFAULT 0 | Default account flag (1/0) |
| `created_at` | TEXT | NOT NULL | ISO-8601 creation timestamp |
| `updated_at` | TEXT | NOT NULL | ISO-8601 update timestamp |

### 12. `email_drafts` Table
Stores composed local email drafts before send.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PRIMARY KEY | Unique draft UUID |
| `account_id` | TEXT | NOT NULL | FK to email_accounts(id) |
| `recipient_emails` | TEXT | NOT NULL | JSON array of recipient emails |
| `cc_emails` | TEXT | | JSON array of CC emails |
| `bcc_emails` | TEXT | | JSON array of BCC emails |
| `subject` | TEXT | | Draft email subject |
| `body` | TEXT | | Draft body text |
| `attachments` | TEXT | | JSON array of attachment metadata |
| `created_at` | TEXT | NOT NULL | ISO-8601 creation timestamp |
| `updated_at` | TEXT | NOT NULL | ISO-8601 update timestamp |

### 13. `email_messages_metadata` Table
Stores cached email message headers and metadata.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PRIMARY KEY | Unique message ID |
| `account_id` | TEXT | NOT NULL | FK to email_accounts(id) |
| `thread_id` | TEXT | | Thread identifier |
| `sender` | TEXT | NOT NULL | Sender email |
| `recipient_emails` | TEXT | NOT NULL | JSON array of recipient emails |
| `subject` | TEXT | | Email subject |
| `snippet` | TEXT | | Short message preview snippet |
| `timestamp` | TEXT | NOT NULL | ISO-8601 sent/received timestamp |
| `is_read` | INTEGER | NOT NULL DEFAULT 0 | Read flag (1/0) |
| `has_attachments` | INTEGER | NOT NULL DEFAULT 0 | Attachments present flag (1/0) |
