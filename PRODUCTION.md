# HabitNexus Production Setup

## PostgreSQL

HabitNexus uses PostgreSQL when `DATABASE_URL` is set. Without `DATABASE_URL`, it falls back to local SQLite for development.

Required environment variables:

```bash
APP_ENV=production
SECRET_KEY=your-long-random-secret
DATABASE_URL=postgresql://user:password@host:5432/database
DATABASE_SSLMODE=prefer
SESSION_COOKIE_SECURE=0
DEVELOPER_EMAILS=your-admin-email@example.com
DB_POOL_MIN=1
DB_POOL_MAX=5
DB_CONNECT_TIMEOUT=5
DB_KEEPALIVES=1
DB_KEEPALIVES_IDLE=30
DB_KEEPALIVES_INTERVAL=10
DB_KEEPALIVES_COUNT=5
DB_POOL_HEALTH_CHECK_ATTEMPTS=2
```

Use `DATABASE_SSLMODE=prefer` for PostgreSQL running on the same EC2 instance.
Use `DATABASE_SSLMODE=require` for a managed database that requires SSL, such as many hosted PostgreSQL providers.

`DB_POOL_MAX` is per Gunicorn worker process. With `--workers 2` and `DB_POOL_MAX=5`, the app can open up to 10 PostgreSQL connections.

Use `SESSION_COOKIE_SECURE=0` while testing over plain HTTP. Change it to `SESSION_COOKIE_SECURE=1` only after Nginx is serving the site over HTTPS.

Install dependencies:

```bash
pip install -r requirements.txt
```

Initialize PostgreSQL tables:

```bash
DATABASE_URL="postgresql://user:password@host:5432/database" python3 -c "from database import init_db; init_db()"
```

Migrate existing SQLite data:

```bash
DATABASE_URL="postgresql://user:password@host:5432/database" python3 scripts/migrate_sqlite_to_postgres.py --sqlite database.db --replace
```

`--replace` clears PostgreSQL tables first. Omit it only when importing into a new empty database.

## Run

```bash
gunicorn app:app --workers 2 --threads 4 --timeout 120 --access-logfile -
```

For hosted platforms, keep `database.db` out of production and use the managed PostgreSQL connection string as `DATABASE_URL`.

`init_db()` is intentionally not called during app startup. Run it once during setup or migration, then start Gunicorn. This keeps hosted workers fast and prevents every web worker from running schema checks on boot.

If developer access is lost after migration, set `DEVELOPER_EMAILS` to the email address of your admin account. After that account logs in, HabitNexus will restore `is_developer=1` automatically.
