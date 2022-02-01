# 📃 Configuration File

The configuration file is to be named `config.yaml`; the name is fixed, but the location can be specified with the [`--cfgDir`](running.md#cfgdir) commandline parameter. If not specified, ws4sqlite will look for it in the current working directory.

The configuration file specifies which database(s) are to be served, and all the settings for serving them. A complete example is as follows:

```yaml
databases:
  - id: db1
    path: ~/first.db?_cache_size=-16000
    auth:
      mode: HTTP
      byCredentials:
        - user: myUser1
          pass: myHotPassword
        - user: myUser2
          hashedPass: b133a0c0e9bee3be20163d2ad31d6248db292aa6dcb1ee087a2aa50e0fc75ae2
    disableWALMode: true
    readOnly: false
    maintenance:
      schedule: 0 0 * * *
      doVacuum: true
      doBackup: true
      backupTemplate: ~/first_%s.db
      numFiles: 3
  - id: db2
    path: ":memory:"
    auth:
      mode: INLINE
      byQuery: SELECT 1 FROM AUTH WHERE USER = :user AND PASS = :pass
    corsOrigin: https://myownsite.com
    useOnlyStoredStatements: false
    storedStatements:
      - id: Q1
        sql: SELECT * FROM TEMP 
      - id: Q2
        sql: INSERT INTO TEMP VALUES (:id, :val)
    initStatements:
      - CREATE TABLE AUTH (USER TEXT PRIMARY KEY, PASS TEXT)
      - INSERT INTO AUTH VALUES ('myUser1', 'myHotPassword')
      - CREATE TABLE TEMP (ID INT PRIMARY KEY, VAL TEXT)
      - INSERT INTO TEMP (ID, VAL) VALUES (1, 'ONE'), (4, 'FOUR')
```

The general structure is a list of objects, in the node `databases`. Each object represents a database to be served. A description of the fields (or areas) follows, with indication of the relevant lines in the example.

If an error occurs, the application will exit with a status code of 1, after printing the error to stderr.

#### `id`&#x20;

_Lines 2, 19; string; mandatory_

A String that indicates an handler for the database. Requests can address this database using this handler.

No two databases can specify the same ID.

#### `path`

_Lines 3, 20; string; mandatory_

Indicates the path of the SQLite file that contains the database.

You can specify a relative or absolute path, and it's possible to use `~` to expand the user home.

It can be `:memory:` or `file::memory:` for an in-memory database, as in line 18.

It can also include URL parameters, as it does in line 3: see the [docs](../advanced-topics.md#pass-parameters-when-opening-a-database).

If the file is not present, it will be attempted to create one. If the path is illegal, an error will be returned, and the application will exit.&#x20;

#### `auth`

_Lines 4-10, 21-23; object_

Configuration for the authentication. See the [relevant section](authentication.md).

#### `disableWALMode`

_Line 11; boolean_

By default a database is opened in [WAL mode](https://sqlite.org/wal.html). This line instructs ws4sqlite to open the first database in rollback mode (i.e. non-WAL).

#### `readOnly`

_Line 12; boolean_

If this boolean flag is present and set to true, the database will be treated as read-only. Only queries are allowed, that don't alter the database structure.

Under the hood, this is performed by adding to the URL the following string:

`?mode=ro&immutable=1&_query_only=1`

This is not compatibile with [`initStatements`](configuration-file.md#initstatements) and [`maintenance`](configuration-file.md#maintenance). Also, a read-only, in-memory db doesn't make much sense.

#### `maintenance`

_Lines 13-18; object_

Instructs ws4sqlite on how (and if) to perform scheduled maintenance on this database. See the [relevant section](maintenance.md).

This can be specified only for non-[`readOnly`](configuration-file.md#readonly) databases.

#### `corsOrigin`

_Line 24; string_

If specified, it enables serving CORS headers in the response, and specifies the value of the `Access-Control-Allow-Origin` header.

It can be set to `*`.

Used to both configure the server to serve CORS headers and, when non-`*`, restrict access to calls from a single, trusted web address.

#### `storedStatements` and `useOnlyStoredStatements`

_`storedStatements`: Lines 26-30; object_\
_`useOnlyStoredStatements`: Line 25; boolean_

Stored Statements are a way to specify (some of) the statement/queries you will use in the server instead of sending them over from the client.

See the [relevant section](stored-statements.md).

#### `initStatements`

_Lines 31-33; list of strings_

When creating a database, it's often useful or even necessary to initialize it. This node allows to list a series of statement that will be applied to a newly-created database.

Notice that an in-memory database is always considered "newly created".

{% hint style="warning" %}
The statements are not run in a transaction: if one fails, the server will exit, but the file may remain created.
{% endhint %}
