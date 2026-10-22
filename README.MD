# Database Restore (PowerShell)

Replace the placeholders (without the angle brackets):
`<PG_VERSION>`, `<PORT>`, `<DB_USER>`, `<DB_NAME>`, `<BACKUP_FILE>`, `<CLEAN_FILE>`, `<LIST_FILE>`

## Custom format (`.dump`)

### Step 1: Delete the old database

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\dropdb.exe" -h localhost -p <PORT> -U <DB_USER> --if-exists --force <DB_NAME>
```

### Step 2: Create the database

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\createdb.exe" -h localhost -p <PORT> -U <DB_USER> -O <DB_USER> -E UTF8 <DB_NAME>
```

### Step 3: Create a restore list without pgvector

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\pg_restore.exe" -l "<BACKUP_FILE>" | Where-Object { $_ -notmatch 'EXTENSION - vector' -and $_ -notmatch 'COMMENT - EXTENSION vector' } | Set-Content "<LIST_FILE>" -Encoding ASCII
```

### Step 4: Restore the backup

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\pg_restore.exe" -h localhost -p <PORT> -U <DB_USER> -d <DB_NAME> --no-owner --role=<DB_USER> -L "<LIST_FILE>" -v "<BACKUP_FILE>"
```

### Step 5: Verify the restore

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\psql.exe" -h localhost -p <PORT> -U <DB_USER> -d <DB_NAME> -c "\dt"
```

## Plain SQL (`.sql`)

### Step 1: Delete the old database

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\dropdb.exe" -h localhost -p <PORT> -U <DB_USER> --if-exists --force <DB_NAME>
```

### Step 2: Create the database

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\createdb.exe" -h localhost -p <PORT> -U <DB_USER> -O <DB_USER> -E UTF8 <DB_NAME>
```

### Step 3: Create a cleaned copy without pgvector

```powershell
[System.IO.File]::WriteAllLines("<CLEAN_FILE>", ([System.IO.File]::ReadLines("<BACKUP_FILE>") | Where-Object { $_ -notmatch 'CREATE EXTENSION IF NOT EXISTS vector' -and $_ -notmatch 'COMMENT ON EXTENSION vector' }), (New-Object System.Text.UTF8Encoding($false)))
```

### Step 4: Restore the backup

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\psql.exe" -h localhost -p <PORT> -U <DB_USER> -d <DB_NAME> -v ON_ERROR_STOP=1 -f "<CLEAN_FILE>"
```

### Step 5: Verify the restore

```powershell
& "C:\Program Files\PostgreSQL\<PG_VERSION>\bin\psql.exe" -h localhost -p <PORT> -U <DB_USER> -d <DB_NAME> -c "\dt"
```