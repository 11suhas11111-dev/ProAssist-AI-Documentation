# Data Architecture — ProAssist AI / Friday

ProAssist AI utilizes a **single-file, local-first embedded SQLite database** (`proassist.db`) managed asynchronously via `aiosqlite`. The data layer is engineered for zero-maintenance desktop operation, absolute crash resilience, and high-performance concurrent read/write transactions.

---

## 1. Core Database Philosophy

1. **Zero External Database Infrastructure**:
   No PostgreSQL, MySQL, Redis, or external daemon is required. All state resides in `~/.proassist/proassist.db` (auto-created on first run).
2. **Write-Ahead Logging (WAL)**:
   Every connection enables `PRAGMA journal_mode=WAL` immediately. This allows concurrent readers to proceed without blocking writers, providing sub-millisecond query latencies.
3. **Strict Relational Integrity**:
   Every connection enables `PRAGMA foreign_keys=ON`. Foreign key cascades automatically maintain referential integrity (e.g. deleting a contact cascades to their aliases, phones, and emails).
4. **Idempotent Non-Destructive Migrations**:
   Schema updates are applied using `CREATE TABLE IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`, and idempotent triggers. Upgrades NEVER drop or overwrite existing tables.
5. **No Database Leaks to Cloud**:
   The database is never synchronized to cloud servers or dumped into LLM context prompts.

---

## 2. Connection Lifecycle & Transaction Management

The `DatabaseManager` (`memory/database.py`) acts as the singleton async connection gateway:

```python
class DatabaseManager:
    @asynccontextmanager
    async def connection(self) -> AsyncGenerator[aiosqlite.Connection, None]:
        async with aiosqlite.connect(str(self._db_path)) as conn:
            conn.row_factory = aiosqlite.Row
            await conn.execute("PRAGMA journal_mode=WAL")
            await conn.execute("PRAGMA foreign_keys=ON")
            try:
                yield conn
                await conn.commit()   # Automatic atomic commit
            except Exception:
                await conn.rollback() # Automatic atomic rollback on failure
                raise
```

---

## 3. Data Subsystem Patterns

### 3.1 Audit & Diagnostics
- `audit_logs`: Append-only, immutable ledger of all tool executions, risk levels, user identities, and parameters. Indexed by `timestamp`.
- `task_history`: Records high-level user commands, executing agents, execution duration in milliseconds, and status.

### 3.2 Full-Text Search (FTS5) & Personal Knowledge
- `notes` & `notes_fts`: Powered by SQLite FTS5 with BM25 ranking algorithm.
- Triggers (`notes_ai`, `notes_ad`, `notes_au`) keep the FTS5 virtual index in sync with `notes` table changes with zero application overhead.
- Supports prefix matching (`term*`) and relevance ranking.

### 3.3 Tasks, Reminders & Scheduling
- `tasks`: Supports statuses `PENDING`, `COMPLETED`, `CANCELLED`, `OVERDUE`.
- `reminders`: Supports natural time schedules, recurrence patterns (`daily`, `weekly:<day>`, `weekdays`), and active scheduling states (`SCHEDULED`, `TRIGGERED`, `COMPLETED`, `CANCELLED`, `MISSED`, `FAILED`).
- Linked via foreign key `associated_task_id` with `ON DELETE SET NULL`.

### 3.4 Contacts & Relationship Graph
- `contacts`: Central entity record.
- Satellite child tables: `contact_aliases`, `contact_relationships`, `contact_phones`, `contact_emails`.
- Normalized search fields (`normalized_name`, `normalized_alias`, `normalized_relationship`) facilitate rapid case-insensitive lookup.

### 3.5 Weather Cache & Expiration
- `weather_locations`: Stored geographic points with default location flag (`is_default`).
- `weather_cache`: TTL-based caching layer for meteorological payloads. Indexed by `(location_key, cache_type)` and `expires_at`.
- Query resolution checks `expires_at >= current_utc_time`. If expired, it triggers a background refresh or degrades to `STALE` if the network fails.

### 3.6 Idempotency Records
- `idempotency_records`: Coordinates distributed execution steps using composite keys (`operation_id`, `step_id`).
- Transitions through explicit states: `PENDING` -> `IN_PROGRESS` -> `SUCCEEDED` / `FAILED`.
