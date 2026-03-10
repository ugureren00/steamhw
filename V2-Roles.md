```sql
DO $$
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'steam_admin') THEN
        CREATE ROLE steam_admin LOGIN PASSWORD 'steam_admin_pass';
    END IF;

    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'steam_app') THEN
        CREATE ROLE steam_app LOGIN PASSWORD 'steam_app_pass';
    END IF;

    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'steam_readonly') THEN
        CREATE ROLE steam_readonly LOGIN PASSWORD 'steam_readonly_pass';
    END IF;
END
$$;

GRANT CONNECT ON DATABASE steam_hw TO steam_admin, steam_app, steam_readonly;
GRANT USAGE, CREATE ON SCHEMA public TO steam_admin;
GRANT USAGE ON SCHEMA public TO steam_app, steam_readonly;

ALTER ROLE steam_admin SET search_path TO public;
ALTER ROLE steam_app SET search_path TO public;
ALTER ROLE steam_readonly SET search_path TO public;
```
