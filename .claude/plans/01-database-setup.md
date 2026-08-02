# Plan: Database Setup (Step 1)

## Context

This step establishes the data layer foundation for the Spendly expense tracker. We need to implement three functions in `database/db.py` that handle SQLite connection setup, table initialization, and demo data seeding. The database file should be named `expense_tracker.db` to match the existing `.gitignore` entry and avoid any untracked file issues.

## Implementation Approach

### 1. `database/db.py` — Complete Implementation

**Structure:**
- Import: `sqlite3`, `os`, `generate_password_hash` from `werkzeug.security`
- Define `DB_PATH` using `os.path` to resolve to project root regardless of CWD
- Implement three functions: `get_db()`, `init_db()`, `seed_db()`

**`get_db()` function:**
- Opens SQLite connection to `expense_tracker.db` in project root
- Sets `row_factory = sqlite3.Row` to enable dict-like row access
- Executes `PRAGMA foreign_keys = ON` before returning (per-connection setting, must happen every time)
- Returns the connection object

**`init_db()` function:**
- Calls `get_db()` to open connection
- Creates `users` table with: id (INTEGER PRIMARY KEY AUTOINCREMENT), name (TEXT NOT NULL), email (TEXT UNIQUE NOT NULL), password_hash (TEXT NOT NULL), created_at (TEXT DEFAULT (datetime('now')))
  - Note: `datetime('now')` must be parenthesized in DEFAULT clause — bare `DEFAULT datetime('now')` is a SQLite syntax error
- Creates `expenses` table with: id (INTEGER PRIMARY KEY AUTOINCREMENT), user_id (INTEGER NOT NULL with FOREIGN KEY constraint to users.id), amount (REAL NOT NULL), category (TEXT NOT NULL), date (TEXT NOT NULL YYYY-MM-DD format), description (TEXT nullable), created_at (TEXT DEFAULT (datetime('now')))
- Both use `CREATE TABLE IF NOT EXISTS` for idempotency
- Commits changes and closes connection

**`seed_db()` function:**
- Calls `get_db()` to open connection
- Checks if users table has any rows; if yes, close connection and return early (idempotent)
- If empty, inserts demo user: name="Demo User", email="demo@spendly.com", password=hashed "demo123" via `generate_password_hash`
- Retrieves inserted user_id via `lastrowid`
- Inserts 8 sample expenses covering all 7 categories (Food, Transport, Bills, Health, Entertainment, Shopping, Other):
  - 2 Food entries, 1 each of Transport, Bills, Health, Entertainment, Shopping, Other
  - Dates spread across current month (2026-08-01 through 2026-08-19)
  - All in YYYY-MM-DD format
  - All linked to the demo user via user_id
- Commits changes and closes connection

### 2. `app.py` — Initialization Calls

**Changes:**
- Add import: `from database.db import get_db, init_db, seed_db`
- After `app = Flask(__name__)` and before route definitions, add:
  ```python
  with app.app_context():
      init_db()
      seed_db()
  ```
- This ensures database and tables exist before any request is handled

**No other changes:** Existing placeholder routes remain unchanged per spec.

### 3. Database File Location

- File: `expense_tracker.db` (in project root, sibling to `app.py`)
- Already in `.gitignore`, so no `.gitignore` edits needed
- Path resolution must use `os.path` to work regardless of CWD

## Critical Files to Modify

1. `/home/labuser/Desktop/Persistent_Folder/ExpenseTracker_ClaudeCode/database/db.py` — full implementation
2. `/home/labuser/Desktop/Persistent_Folder/ExpenseTracker_ClaudeCode/app.py` — add imports and init calls

## Verification

After implementation, verify via `python3 -c` scripts and running the app:
- `expense_tracker.db` file is created in project root when app starts
- Both `users` and `expenses` tables exist with correct schema
- Demo user exists (1 row in users) with hashed password
- 8 sample expenses exist covering all 7 categories
- Re-running `seed_db()` doesn't create duplicates (idempotency check works)
- Foreign key constraint enforcement prevents invalid user_id inserts (raises `sqlite3.IntegrityError`)
- Unique email constraint prevents duplicate emails (raises `sqlite3.IntegrityError`)
- App starts without errors via `python3 app.py`
