# 🔦 Cheat Sheet

## Commandline

```bash
ws4sqlite 
    --bind-host 0.0.0.0 \        # Optional
    --port 12321 \               # Optional
    --db ~/file1.db \            # File-based db, cfg at ~/file1.yaml
    --mem-db mem1:~/mem1.yaml \  # Memory-based db, with cfg
    --mem-db mem2                # Memory-based db, with default cfg
```

## Configuration file

```yaml
# Every first-level element is optional
auth:
  mode: HTTP                   # INLINE or HTTP
  # Or byCredentials; query must have :user and :password
  byQuery: SELECT 1 FROM AUTH WHERE USER = :user AND PASSWORD = :password
  byCredentials:                  # Or byQuery:
    - user: myUser1
      password: myCoolPassword
    - user: myUser2
      hashedPassword: b133...     # SHA-256 of the password in HEX form
disableWALMode: true
readOnly: true
maintenance:
  schedule: 0 0 * * *             # Cron format without seconds
  doVacuum: true
  doBackup: true
  backupTemplate: ~/temp_%s.db    # %s must be present, yyyyMMdd_HHmm
  numFiles: 3                     # Backup files to keep 
corsOrigin: https://myownsite.com # Access-Control-Allow-Origin
useOnlyStoredStatements: true     
storedStatements:
  - id: Q1
    sql: SELECT * FROM TEMP 
  - id: Q2
    sql: INSERT INTO TEMP VALUES (:id, :val)
initStatements:                   # Executed when a db is new
  - CREATE TABLE AUTH (USER TEXT PRIMARY KEY, PASSWORD TEXT)
  - INSERT INTO AUTH VALUES ('myUser1', 'myCoolPassword')
  - CREATE TABLE TEMP (ID INT PRIMARY KEY, VAL TEXT)
  - INSERT INTO TEMP (ID, VAL) VALUES (1, 'ONE'), (4, 'FOUR')
```
