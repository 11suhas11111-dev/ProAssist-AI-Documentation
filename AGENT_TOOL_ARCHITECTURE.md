# Agent & Tool Architecture — Complete Inventory

ProAssist AI features a multi-agent execution hierarchy comprising **6 specialized domain agents** and **50 registered tools**. Every tool is strictly governed by `PermissionManager` risk ratings and `ToolRegistry` schema contracts.

---

## 1. Agent Overview

| Agent Name | Class | Domain Responsibility | Registered Tools |
|---|---|---|:---:|
| **`system_agent`** | `SystemAgent` | Desktop OS operations, process management, volume, screenshots. | 10 |
| **`file_agent`** | `FileAgent` | Safe file manipulation, searching, archives, duplicate file detection. | 10 |
| **`contact_agent`**| `ContactAgent`| Contact management, relationship mapping, alias resolution. | 6 |
| **`task_agent`** | `TaskAgent` | Todo tasks, natural language scheduling, active reminders. | 11 |
| **`notes_agent`** | `NotesAgent` | Personal knowledge base, SQLite FTS5 BM25 full-text search. | 8 |
| **`weather_agent`**| `WeatherAgent`| Weather forecasts and location management via Open-Meteo REST API. | 5 |
| **Total** | | | **50** |

---

## 2. Complete Tool Catalog

### 2.1 System Agent Tools (`agents/system_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `open_application` | Launches Windows executable by name or alias | `MEDIUM` | `AUTHENTICATED` | No |
| `close_application` | Terminates process by executable name | `MEDIUM` | `AUTHENTICATED` | No |
| `take_screenshot` | Captures display and saves PNG to Pictures | `LOW` | `AUTHENTICATED` | No |
| `volume_up` | Increases system master volume by 5% | `LOW` | `AUTHENTICATED` | No |
| `volume_down` | Decreases system master volume by 5% | `LOW` | `AUTHENTICATED` | No |
| `mute_volume` | Toggles system audio mute | `LOW` | `AUTHENTICATED` | No |
| `get_clipboard` | Reads plain text from Windows clipboard | `LOW` | `AUTHENTICATED` | No |
| `set_clipboard` | Writes text content to Windows clipboard | `LOW` | `AUTHENTICATED` | No |
| `get_system_info` | Reads CPU, RAM, and Disk percentage | `LOW` | `PUBLIC` | No |
| `list_processes` | Returns list of active running processes | `LOW` | `AUTHENTICATED` | No |

---

### 2.2 File Agent Tools (`agents/file_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `list_files` | Lists directory contents with filters | `LOW` | `AUTHENTICATED` | No |
| `search_files` | Searches filesystem by filename or glob pattern | `LOW` | `AUTHENTICATED` | No |
| `copy_files` | Copies file or directory to target destination | `MEDIUM` | `AUTHENTICATED` | No |
| `move_files` | Moves file to destination (modifies source) | `MEDIUM` | `CONFIRMATION` | **YES** |
| `rename_file` | Renames a file or folder in-place | `MEDIUM` | `AUTHENTICATED` | No |
| `delete_files` | Permanently removes files or directories | `HIGH` | `CONFIRMATION` | **YES** |
| `create_directory`| Creates directory structure | `LOW` | `AUTHENTICATED` | No |
| `compress_files` | Packs files into a standard ZIP archive | `MEDIUM` | `AUTHENTICATED` | No |
| `extract_archive` | Extracts ZIP archive to target directory | `MEDIUM` | `AUTHENTICATED` | No |
| `find_duplicates` | Scans for duplicate files by SHA-256 hash | `LOW` | `AUTHENTICATED` | No |

---

### 2.3 Contact Agent Tools (`agents/contact_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `create_contact` | Creates contact with aliases, phones, emails | `MEDIUM` | `AUTHENTICATED` | No |
| `get_contact` | Retrieves contact by ID or name resolution | `LOW` | `AUTHENTICATED` | No |
| `search_contacts` | Queries contacts by text or relationship | `LOW` | `AUTHENTICATED` | No |
| `list_contacts` | Returns paginated list of all contacts | `LOW` | `AUTHENTICATED` | No |
| `update_contact` | Updates contact details or relationships | `MEDIUM` | `AUTHENTICATED` | No |
| `delete_contact` | Removes contact and all child records | `HIGH` | `CONFIRMATION` | **YES** |

---

### 2.4 Task Agent Tools (`agents/task_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `create_task` | Creates todo task with priority and due date | `MEDIUM` | `AUTHENTICATED` | No |
| `get_task` | Retrieves task details by ID or title match | `LOW` | `AUTHENTICATED` | No |
| `list_tasks` | Lists tasks filtered by status | `LOW` | `AUTHENTICATED` | No |
| `complete_task` | Marks task status as COMPLETED | `LOW` | `AUTHENTICATED` | No |
| `cancel_task` | Sets task status to CANCELLED | `LOW` | `AUTHENTICATED` | No |
| `delete_task` | Permanently removes task record | `HIGH` | `CONFIRMATION` | **YES** |
| `create_reminder`| Schedules reminder with natural time parsing | `MEDIUM` | `AUTHENTICATED` | No |
| `get_reminder` | Retrieves reminder details | `LOW` | `AUTHENTICATED` | No |
| `list_reminders` | Lists upcoming scheduled reminders | `LOW` | `AUTHENTICATED` | No |
| `cancel_reminder`| Cancels scheduled reminder in scheduler | `LOW` | `AUTHENTICATED` | No |
| `delete_reminder`| Permanently deletes reminder | `HIGH` | `CONFIRMATION` | **YES** |

---

### 2.5 Notes Agent Tools (`agents/notes_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `create_note` | Stores personal note (explicit intent only) | `MEDIUM` | `AUTHENTICATED` | No |
| `get_note` | Retrieves note content by title or UUID | `LOW` | `AUTHENTICATED` | No |
| `list_notes` | Lists active notes with pagination | `LOW` | `AUTHENTICATED` | No |
| `search_notes` | Performs SQLite FTS5 BM25 ranked search | `LOW` | `AUTHENTICATED` | No |
| `update_note` | Appends or replaces note content | `MEDIUM` | `AUTHENTICATED` | No |
| `archive_note` | Sets status to ARCHIVED | `MEDIUM` | `AUTHENTICATED` | No |
| `restore_note` | Restores ARCHIVED note back to ACTIVE | `MEDIUM` | `AUTHENTICATED` | No |
| `delete_note` | Permanently deletes note and FTS5 index | `HIGH` | `CONFIRMATION` | **YES** |

---

### 2.6 Weather Agent Tools (`agents/weather_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `get_current_weather` | Fetches current weather (live or cached) | `LOW` | `AUTHENTICATED` | No |
| `get_weather_forecast`| Fetches multi-day forecast | `LOW` | `AUTHENTICATED` | No |
| `get_weather_location`| Returns configured default location | `LOW` | `AUTHENTICATED` | No |
| `set_weather_location`| Sets user default location in SQLite | `MEDIUM` | `AUTHENTICATED` | No |
| `clear_weather_location`| Removes configured default location | `LOW` | `AUTHENTICATED` | No |
