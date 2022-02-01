# 🥇 Features

* A [**single executable file**](documentation/installation/) (it's written in Go);
* HTTP/JSON access, with [**client libraries**](client-libraries.md) for convenience;
* Can load [**several databases**](documentation/configuration-file.md) at once, for convenience;
* [**Batching** ](documentation/requests.md#batch-parameter-values-for-a-statement)of multiple values set for a single statement;
* All queries of a call are executed in a [**transaction**](documentation/requests.md);
* For each query/statement, specify if a failure should rollback the whole transaction, or the failure is [**limited** ](documentation/requests.md#nofail-dont-fail-when-errors-occour)to that query;
* "[**Stored statements**](documentation/requests.md#stored-query-reference)": define SQL in the server, and call it with a key from the client;
* [**CORS** ](documentation/configuration-file.md#corsorigin)mode, configurable per-db;
* [**Maintenance** ](documentation/maintenance.md)scheduling (VACUUM and backups), also configurable per-db;
* Builtin [**encryption** ](documentation/encryption.md)of fields, given a symmetric key;
* Supports [**in-memory DBs**](documentation/configuration-file.md#path-mandatory);
* Allows to provide [**initialization statements**](documentation/configuration-file.md#initstatements) to execute when a DB is created;
* [**WAL**](https://sqlite.org/wal.html) mode enabled by default, [can be disabled](documentation/configuration-file.md#disablewalmode);
* Very fast (benchmarks coming up!);
* Compact codebase (< 800 lines of code);
* Comprehensive [**test** ](building-and-testing.md#testing)suite (`cd src; go test -v`);
* ****[**Docker images**](documentation/installation/docker.md) are available, both for amd64 and for arm (32).

### Security Features

* ****[**Authentication** ](documentation/security.md#authentication)can be configured
  * on the client, either using HTTP Basic Authentication or specifying the credentials in the request;
  * on the server, either by specifying credentials (also with hashed passwords) or providing a query to look them up in the db itself;
* A database can be opened in [**read-only mode**](documentation/security.md#read-only-databases) (only queries will be allowed);
* It's possible to enforce using [**only stored statements**](documentation/security.md#stored-statements-to-prevent-sql-injection), to avoid some forms of SQL injection and receiving SQL from the client altogether;
* ****[**CORS Allowed Origin**](documentation/security.md#cors-allowed-origin) can be configured and enforced;
* It's possible to [**bind** ](documentation/security.md#binding-to-a-network-interface)to a network interface, to limit access.

Some deliberate choices have been made:

* Very thin layer over SQLite. Errors and type translation, for example, are those provided by the SQLite driver;
* Doesn't include HTTPS, as this can be done easily (and much more securely) with a [reverse proxy](documentation/security.md#use-a-reverse-proxy-if-going-on-the-internet);
* Doesn't support SQLite extensions, to improve portability.
