# 📃 Configuration File

The configuration files are a set of companion files for each databases. They can be absent, and the database will take default values; if they are present, they indicate the parameters with which ws4sqlite must "treat" the database.

For **file based db**s, the configuration file follows a naming convention: it must have the same path and filename of the db file, buit with a `.yaml` extension instead of `.db`. For **memory based db**s, the config files can be specified in the [`--mem-db`](running.md#mem-db) commandline argument.

The configuration is in YAML format. A couple of examples that describe the entire set of configurations are as follows:

```yaml
auth:
  mode: HTTP
  byCredentials:
    - user: myUser1
      password: myCoolPassword
    - user: myUser2
      hashedPassword: b133a0c0e9bee3be20163d2ad31d6248db292aa6dcb1ee087a2aa50e0fc75ae2
disableWALMode: true
readOnly: false
maintenance:
  schedule: 0 0 * * *
  doVacuum: true
  doBackup: true
  backupTemplate: ~/temp_%s.db
  numFiles: 3
```

And:

```yaml
auth:
  mode: INLINE
  byQuery: SELECT 1 FROM AUTH WHERE USER = :user AND PASSWORD = :password
corsOrigin: https://myownsite.com
useOnlyStoredStatements: true
storedStatements:
  - id: Q1
    sql: SELECT * FROM TEMP 
  - id: Q2
    sql: INSERT INTO TEMP VALUES (:id, :val)
initStatements:
  - CREATE TABLE AUTH (USER TEXT PRIMARY KEY, PASSWORD TEXT)
  - INSERT INTO AUTH VALUES ('myUser1', 'myCoolPassword')
  - CREATE TABLE TEMP (ID INT PRIMARY KEY, VAL TEXT)
  - INSERT INTO TEMP (ID, VAL) VALUES (1, 'ONE'), (4, 'FOUR')
```

A description of the fields (or areas) follows, with indication of the relevant lines in the example.

If an error occurs while parsing them, the application will exit with a status code of 1, after printing the error to stderr.

#### `auth`

_Lines 1-7 of #1, 1-3 of #2; object_

Configuration for the authentication. See the [relevant section](authentication.md).

#### `disableWALMode`

_Line 8 of #1; boolean_

By default a database is opened in [WAL mode](https://sqlite.org/wal.html). This line instructs ws4sqlite to open the database in rollback mode (i.e. non-WAL).

#### `readOnly`

_Line 9 of #1; boolean_

If this boolean flag is present and set to true, the database will be treated as read-only. Only queries are allowed, that don't alter the database structure.

Under the hood, this is performed by adding to the URL the following [parameter string](https://github.com/mattn/go-sqlite3#connection-string):

`?mode=ro&immutable=1&_query_only=1`

This is not compatibile with [`initStatements`](configuration-file.md#initstatements) and [`maintenance`](configuration-file.md#maintenance). Also, a read-only, in-memory db doesn't make much sense.

#### `maintenance`

_Lines 10-15 of #1; object_

If present, instructs ws4sqlite on how to perform scheduled maintenance on this database. See the [relevant section](maintenance.md).

This can be specified only for non-[`readOnly`](configuration-file.md#readonly) databases.

#### `corsOrigin`

_Line 4 of #2; string_

If specified, it enables serving CORS headers in the response, and specifies the value of the `Access-Control-Allow-Origin` header.

It can be set to `*`.

Used to both configure the server to serve CORS headers and, when non-`*`, restrict access to calls from a single, trusted web address.

#### `storedStatements` and `useOnlyStoredStatements`

_`storedStatements`: Lines 6-10 of #2; object_\
_`useOnlyStoredStatements`: Line 5 of #2; boolean_

Stored Statements are a way to specify (some of) the statement/queries you will use in the server instead of sending them over from the client.

See the [relevant section](stored-statements.md).

#### `initStatements`

_Lines 11-13 of #2; list of strings_

When creating a database, it's often useful or even necessary to initialize it. This node allows to list a series of statement that will be applied to a newly-created database.

Notice that an in-memory database is always considered "newly created".

{% hint style="warning" %}
The statements are not run in a transaction: if one fails, the server will exit, but the file may remain created.
{% endhint %}
