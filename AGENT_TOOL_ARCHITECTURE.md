# Agent & Tool Architecture — Complete Inventory

ProAssist AI features a multi-agent execution hierarchy comprising **8 specialized domain agents** and **59 registered tools**. Every tool is strictly governed by `PermissionManager` risk ratings and `ToolRegistry` schema contracts.

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
| **`web_search_agent`**| `WebSearchAgent`| Targeted web search and factual citation retrieval. | 2 |
| **`calendar_agent`**| `CalendarAgent`| Calendar scheduling, event retrieval, conflict warnings. | 7 |
| **Total** | | | **59** |

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
| `create_contact` | Creates contact record with phone/email/relationship | `LOW` | `AUTHENTICATED` | No |
| `get_contact` | Retrieves detailed contact information | `LOW` | `AUTHENTICATED` | No |
| `search_contacts`| Searches contacts by name, alias, email, phone | `LOW` | `AUTHENTICATED` | No |
| `update_contact` | Updates existing contact fields | `LOW` | `AUTHENTICATED` | No |
| `delete_contact` | Deletes contact record | `HIGH` | `CONFIRMATION` | **YES** |
| `list_contacts` | Lists all stored contacts | `LOW` | `AUTHENTICATED` | No |

---

### 2.4 Task Agent Tools (`agents/task_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `create_task` | Creates new todo task with priority/due date | `LOW` | `AUTHENTICATED` | No |
| `list_tasks` | Lists active tasks with filtering | `LOW` | `AUTHENTICATED` | No |
| `get_task` | Retrieves specific task details | `LOW` | `AUTHENTICATED` | No |
| `update_task` | Updates title, priority, or due date | `LOW` | `AUTHENTICATED` | No |
| `complete_task` | Marks task as completed | `LOW` | `AUTHENTICATED` | No |
| `delete_task` | Permanently deletes task | `HIGH` | `CONFIRMATION` | **YES** |
| `create_reminder`| Schedules reminder notification | `LOW` | `AUTHENTICATED` | No |
| `list_reminders` | Lists upcoming scheduled reminders | `LOW` | `AUTHENTICATED` | No |
| `cancel_reminder`| Cancels an active reminder | `LOW` | `AUTHENTICATED` | No |
| `snooze_reminder`| Postpones reminder by specified duration | `LOW` | `AUTHENTICATED` | No |
| `dismiss_reminder`| Acknowledges and dismisses reminder | `LOW` | `AUTHENTICATED` | No |

---

### 2.5 Notes Agent Tools (`agents/notes_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `create_note` | Creates note and builds FTS5 search index | `LOW` | `AUTHENTICATED` | No |
| `get_note` | Retrieves note content and metadata | `LOW` | `AUTHENTICATED` | No |
| `search_notes` | Searches notes using FTS5 BM25 ranked ranking | `LOW` | `AUTHENTICATED` | No |
| `update_note` | Updates note content or title | `LOW` | `AUTHENTICATED` | No |
| `archive_note` | Moves note to archived state | `LOW` | `AUTHENTICATED` | No |
| `restore_note` | Restores archived note | `LOW` | `AUTHENTICATED` | No |
| `delete_note` | Permanently removes note and search index | `HIGH` | `CONFIRMATION` | **YES** |
| `list_notes` | Lists notes with state filters | `LOW` | `AUTHENTICATED` | No |

---

### 2.6 Weather Agent Tools (`agents/weather_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `get_current_weather` | Retrieves temperature and condition | `LOW` | `AUTHENTICATED` | No |
| `get_weather_forecast`| Retrieves multi-day weather forecast | `LOW` | `AUTHENTICATED` | No |
| `get_weather_location`| Reads configured default weather location | `LOW` | `AUTHENTICATED` | No |
| `set_weather_location`| Sets default weather location in DB | `LOW` | `AUTHENTICATED` | No |
| `clear_weather_location`| Clears default weather location | `LOW` | `AUTHENTICATED` | No |

---

### 2.7 Web Search Agent Tools (`agents/web_search_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `web_search` | Executes privacy-preserving web search with factual citations | `LOW` | `AUTHENTICATED` | No |
| `get_search_providers`| Lists registered and active web search providers | `LOW` | `AUTHENTICATED` | No |

---

### 2.8 Calendar Agent Tools (`agents/calendar_agent.py`)

| Tool Name | Description | Risk Level | Required Auth | Confirmation? |
|---|---|:---:|:---:|:---:|
| `list_calendars` | Lists available local and connected calendars | `LOW` | `AUTHENTICATED` | No |
| `list_events` | Lists events within a specified date/time window | `LOW` | `AUTHENTICATED` | No |
| `get_event` | Retrieves detailed information for a specific event | `LOW` | `AUTHENTICATED` | No |
| `search_events` | Searches calendar events by keyword query | `LOW` | `AUTHENTICATED` | No |
| `create_event` | Schedules new event with conflict warning support | `MEDIUM` | `AUTHENTICATED` | No |
| `update_event` | Modifies existing event time, title, or location | `MEDIUM` | `AUTHENTICATED` | No |
| `delete_event` | Permanently removes calendar event | `HIGH` | `CONFIRMATION` | **YES** |
