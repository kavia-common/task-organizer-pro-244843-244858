# Task Management Database Migrations (PostgreSQL)

This project uses a lightweight “migration runner” pattern (Python script) instead of Alembic.
It is designed for the Kavia environment where PostgreSQL is already started via `startup.sh`.

## Connection source of truth

**Always** use the connection string from:

- `task_management_database/db_connection.txt`

Example content:

- `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

The backend should use environment variables in deployment, but for local/dev in this repo you can also use the same credentials.

## How to apply migrations

From the backend container (recommended), run:

```bash
python -m src.db.migrate
```

This script is **idempotent**: it uses `CREATE TABLE IF NOT EXISTS` and `CREATE INDEX IF NOT EXISTS`.

## Schema overview

### users
- id (uuid, pk)
- email (unique)
- password_hash
- created_at

### tasks
- id (uuid, pk)
- user_id (fk -> users.id)
- title
- description
- status: todo | in_progress | done
- priority: low | medium | high
- due_date (date, nullable)
- created_at / updated_at

Indexes:
- tasks(user_id, status)
- tasks(user_id, due_date)
- tasks(user_id, priority)
- tasks(user_id, created_at DESC)
- tasks.title/description full text search via GIN on to_tsvector

Notes:
- Search uses PostgreSQL full-text search over title+description.
- All task queries are scoped to the authenticated user.
