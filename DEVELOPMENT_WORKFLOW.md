# Development Workflow & Engineering Standards — ProAssist AI

This document outlines the **mandatory engineering practices, testing protocols, and development rules** for contributors and autonomous coding agents working on ProAssist AI.

---

## 1. Development Environment Setup

- **Host OS**: Windows 11 (64-bit)
- **Python**: Python 3.11.9 (`C:\Users\DELL\AppData\Local\Programs\Python\Python311\python.exe`)
- **Project Directory**: `c:\Users\DELL\OneDrive\Documents\ProAssist AI`

```powershell
# Verify Python version
& "C:\Users\DELL\AppData\Local\Programs\Python\Python311\python.exe" --version
# Expected: Python 3.11.9

# Install dependencies
& "C:\Users\DELL\AppData\Local\Programs\Python\Python311\python.exe" -m pip install -r requirements.txt
```

---

## 2. Regression Testing Barrier

Before making any modification, and before completing any phase:

```powershell
# Run the complete test suite
& "C:\Users\DELL\AppData\Local\Programs\Python\Python311\python.exe" -m pytest -q
```

**MANDATORY RULE**:
The test suite currently produces **367 passed tests** (0 failed, 0 errors). Any code modification that causes a regression, failure, or test count drop MUST BE FIXED immediately before proceeding.

---

## 3. Phase Development Lifecycle

Future phases (Phase 6.5 Web Search, Phase 6.6 Calendar, etc.) must follow this strict development sequence:

1. **Architecture Planning**:
   - Inspect existing interfaces in `core/`, `security/`, and `memory/`.
   - Never assume missing functionality; verify existing tools first.
   - Define exact contracts and SQLite schema additions.
2. **Schema & Models**:
   - Add new tables to `SCHEMA_SQL` in `memory/database.py` using `IF NOT EXISTS`.
   - Create typed dataclasses and enums in the target subsystem package.
3. **Repository & Service Layer**:
   - Implement async database access via `DatabaseManager`.
   - Implement domain logic with `SafeHttpClient` and `SecretRedactor`.
4. **Tools & Agent Registration**:
   - Subclass `BaseTool` and `BaseAgent`.
   - Register tool permissions in `security/permission_manager.py` with explicit `RiskLevel` and `AuthLevel`.
   - Register tool signatures in `core/llm/tool_registry.py`.
5. **TaskRouter & Main Integration**:
   - Add regex voice patterns to `core/task_router.py` (supporting EN, HI, KN).
   - Wire the agent into `main._async_init()`.
6. **Testing & Verification**:
   - Write comprehensive unit tests in `tests/test_phase<X>_<feature>.py`.
   - Verify full test suite passes with zero regressions.
   - Run manual/scripted Windows validation scenarios.

---

## 4. Secret & Git Hygiene

- **NEVER** commit `.env` or files containing raw passwords or API keys.
- **NEVER** commit `proassist.db` if it contains personal user data.
- **ALWAYS** run `SecretRedactor` checks on newly introduced logging statements.
