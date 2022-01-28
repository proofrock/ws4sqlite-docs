#🏫 Tutorial

In this tutorial we'll run ws4sqlite for the first time, in order to serve a single database, and we'll run a couple of queries and statements against it.&#x20;

We'll use Linux, but the information here is "portable" to MacOS and Windows as well.

{% hint style="success" %}
If ws4sqlite offers a certain capability that is important to know but doesn't fit in a tutorial, it will be explained in an info box like this, and a link to the relevant documentation will be provided.
{% endhint %}

Let's start!

### 🔧 Installation

The installation is simple, as ws4sqlite "is" just an executable file. [Download ](https://github.com/proofrock/ws4sqlite/releases)or [build ](building-and-testing.md)it (matching your OS and architecture), and put it somewhere on the filesystem.

Also, create a directory somewhere, with any name, that will contain the configuration file. Since you are there, create the file (it's called `config.yaml`).

In Linux/MacOS:

```bash
mkdir ~/ws4sqlite_config
cd ~/ws4sqlite_config
touch config.yaml
```

### 🚀 First Run & Configuration

Let's edit the `config.yaml` file we just created, with these contents:

```yaml
databases:
  - id: testDb
    path: ~/ws4sqlite_config/testDb.db
```

This file tells ws4sqlite to create a database `testDb.db` at the given path, and to assign an ID to it, `testDb`.

{% hint style="success" %}
Some more options are possible: provide [authentication](documentation/authentication.md), open the file as [read only](documentation/configuration-file.md), or [specify some queries/statements](documentation/stored-statements.md) on the server, that can be referenced in requests.
{% endhint %}

Let's now start the application:

```bash
ws4sqlite --cfgDir ~/ws4sqlite_config
```

Something like this will be printed; it gives information about what is now being served, and how.

```
ws4sqlite x.y.z
- Serving database 'testDb' from /home/mano/ws4sqlite_config/testDb.db?_journal=WAL
  + Using WAL
- Web Service listening on 0.0.0.0:12321
```

The service is now active and serving requests. Use `Ctrl-c` to exit, as usual.

{% hint style="success" %}
From the commandline, it's also possible to specify the [port](documentation/running.md#port) and the [host ](documentation/running.md#bind-host)to bind to.
{% endhint %}

{% hint style="info" %}
Since the db file is not present at the path given in the config file, it will be created.
{% endhint %}

### 🔍 First Request

Let's now do something useful. Use a tool like [postman ](https://www.postman.com)to submit a POST call with this body to `http://localhost:12321/testDb`:

```json
{
    "transaction": [
        {
            "statement": "CREATE TABLE TEST_TABLE (ID int primary key, VAL text)"
        }
    ]
}
```

{% hint style="warning" %}
Ensure that the header `Content-Type` is set to `application/json`
{% endhint %}

Let's see what is in the request:

* **URL:** in the connection URL, we specify the database ID to submit the request to, as defined in the config file;
* **Line 2**: specify the transaction operation list: an array of requests (queries or statements) to submit;
* **Line 4**: as the first (and only) request, specify a statement (i.e. a SQL command that doesn't return a resultset).

If all goes well, you should get a `200` response with the following body:

```json
{
    "results": [
        {
            "success": true,
            "rowsUpdated": 0
        }
    ]
}
```

You did it! 🚀 Going through the response:

* **Line 2**: the array of results has the same size of the corresponding `transaction`array in the request, and lists the results of each request, in turn;
* **Line 4**: specifies that the statement completed with success;
* **Line 5**: there were no updated rows (as reported by SQLite; this is a DDL command).

### 🤹 Multiple Requests

Let's say you want to run multiple SQLs in the same request. As you may suspect, this is just a matter of adding another item to the `transaction` array:

{% hint style="info" %}
As the name implies, the queries/statements are run in a single transaction. It will be committed at the end of the call, and rolled back if an error occours (see [the relevant chapter](tutorial.md#managing-errors)).
{% endhint %}

{% hint style="warning" %}
For this example, you'll need to start from a new db, so stop the application, remove the database file and start it again.
{% endhint %}

```json
{
    "transaction": [
        {
            "statement": "CREATE TABLE TEST_TABLE (ID int primary key, VAL text)"
        },
        {
            "statement": "INSERT INTO TEST_TABLE (ID, VAL) VALUES (1, 'hello')"
        }
    ]
}
```

The response now has 2 elements in the array:

```json
{
    "results": [
        {
            "success": true,
            "rowsUpdated": 0
        },
        {
            "success": true,
            "rowsUpdated": 1
        }
    ]
}
```

Please notice **Line 9**: `rowsUpdated` is 1, signaling that we updated one row with the `INSERT`.

### 🧯 Managing Errors

Let's send over the last request again (don't remove the file!). The table already exists, so the response will now fail with `500 Internal Server Error` , and with the body:

```json
{
    "qryIdx": 0,
    "error": "Table TEST_TABLE already exists",
}
```

* **Line 2**: the (0-based) index of the statement that failed in the `queries` array; an index of `-1` would tell us that it's a generic error, not tied to a particular statement;
* **Line 3**: the reason of the failure, as reported by SQLite.

This is a general failure. As queries in the same request are run in transaction, the whole transaction is rolled back.

It's also possible to "allow" a single query to fail. The transaction will be completed, and an error will be reported only on that query. Notice the `"noFail"` in the second statement and send the following request:

```json
{
    "transaction": [
        {
            "statement": "INSERT INTO TEST_TABLE (ID, VAL) VALUES (2, 'ciao')"
        },
        {
            "noFail": true,
            "statement": "CREATE TABLE TEST_TABLE (ID int primary key, VAL text)"
        }
    ]
}
```

The following result is produced, signaling that the second statement failed; the transaction is committed anyway, so the first statement is actually persisted:

```json
{
    "results": [
        {
            "success": true,
            "rowsUpdated": 1
        },
        {
            "success": false,
            "error": "Table TEST_TABLE already exists"
        }
    ]
}
```

### 🎂 Queries With a Result (Set)

Up to now, we tested only statements, that don't return results other than the number of affected rows. Let's see how to run a _query_.&#x20;

In the next example we will create a table, insert two rows in it, and read them.&#x20;

{% hint style="warning" %}
Please start from an empty database: kill ws4sqlite, remove the database file and start it again.
{% endhint %}

Request:

```json
{
    "transaction": [
        {
            "statement": "CREATE TABLE TEST_TABLE (ID int primary key, VAL text)"
        },
        {
            "statement": "INSERT INTO TEST_TABLE (ID, VAL) VALUES (1, 'hello'), (2, 'world')"
        },
        {
            "query": "SELECT * FROM TEST_TABLE ORDER BY ID ASC"
        }
    ]
}
```

* **Line 10**: notice that the key now is `query`, to signal that it will generate a result set.

```json
{
    "results": [
        ...omitted the first tworesults...
        {
            "success": true,
            "resultSet": [
                { "ID": 1, "VAL": "hello" },
                { "ID": 2, "VAL": "world" }
            ]
        }
    ]
}
```

* **Lines 7..8**: we now have an array with two results, containing objects. Each object has several fields, with the key being the name of the database field and the value being... the value. The key/field name is as reported by the database, so `*` works well.

### ♟️ Using Parameters

The last capability we'll cover is using parameters, either in a statement (e.g. an `INSERT`) or in a query. They are specified using named placeholders, like the following.

{% hint style="warning" %}
For this example, please use the database produced in the last step, as it has a table already defined.
{% endhint %}

```json
{
    "transaction": [
        {
            "statement": "INSERT INTO TEST_TABLE (ID, VAL) VALUES (:id, :val)",
            "values": { "id": 101, "val": "A hundred and 1" }
        },
        {
            "query": "SELECT * FROM TEST_TABLE WHERE ID = :id",
            "values": { "ID": 101, "VAL": "A hundred and 1" }
        }
    ]
}
```

* **Line 4**: in the statement, we use named placeholders like `:id` ;
* **Line 5**: we specify the actual values with a `values` object, containing a map with the keys being the placeholders;
* Same with queries, as in the **Lines 8 and 9**.

{% hint style="info" %}
It may seem more verbose than specifying the values in the SQL, but it is _always_ the preferrable solution, allowing for example to avoid nasty [SQL injection bugs](https://xkcd.com/327/).
{% endhint %}

{% hint style="success" %}
For statements, it is also possible to specify multiple sets of values ('batches'); the statement will be cached and replayed for each set of the list. See `valuesBatch` in the [docs](documentation/requests.md).
{% endhint %}

```json
{
    "results": [
        {
            "success": true,
            "rowsUpdated": 1
        },
        {
            "success": true,
            "resultSet": [
                {
                    "ID": 101,
                    "VAL": null
                }
            ]
        }
    ]
}j
```

As you can see, the response is the same, with only one result in the resultset (since the query selects by primary key).

### 🕯️ Conclusions

Thanks for reading so far, I hope you liked it! There are many more topics of interest, among which:

* Learn to protect your transactions with [authentication](documentation/authentication.md);
* Use a [reverse proxy](integrations/reverse-proxy.md) for HTTPS and additional security;
* Use [stored statements ](documentation/stored-statements.md)to avoid passing SQL from the client;
* Perform scheduled maintenance activities, `VACUUM`s or backups;
* Configure [CORS ](documentation/configuration-file.md#corsorigin)for more convenient access from a web page;
* ...and much more!

Have a nice day! ☀️
