# ❓ Requests

A request is a JSON structure that is passed via a POST HTTP call to WS4SQLite, using the port specified when [running ](running.md#port)the server application.

First and foremost, the database we connect to is specified in the URL of the POST call. It is something like this:

```bash
http://localhost:12321/db2
```

That `db2` is the database ID, and must match the [`id`](configuration-file.md#id) of a database defined in the configuration file.

This is a JSON that exemplifies all possible elements of a request (but for [encryption](encryption.md), to keep things simple).

```json
{
    "credentials": {
        "user": "myUser1",
        "pass": "myHotPassword"
    },
    "transaction": [
        {
            "query": "SELECT * FROM TEMP"
        },
        {
            "query": "SELECT * FROM TEMP WHERE ID = :id",
            "values": { "id": 1 }
        },
        {
            "statement": "INSERT INTO TEMP (ID, VAL) VALUES (0, 'ZERO')"
        },
        {
            "noFail": true,
            "statement": "INSERT INTO TEMP (ID, VAL) VALUES (:id, :val)",
            "values": { "id": 1, "val": "a" }
        },
        {
            "statement": "#Q2",
            "valuesBatch": [
                { "id": 2, "val": "b" },
                { "id": 3, "val": "c" }
            ]
        }
    ]
}
```

Let's go through it.

#### Authentication Credentials

_Lines 2-5; object_\
\
If [authentication ](authentication.md)is enabled in `INLINE` mode, this object describes the credentials. See the [detailed docs](authentication.md#credentials-in-the-request-inline-mode).

#### List of Queries/Statements

_Line 6; list of objects; mandatory_\
__\
__Must be not empty. The list of the queries or statements that will actually be performed on the database, with their own parameters.\
\
They will be run in a transaction, and the transaction will be committed only if all the queries that are not marked as `noFail` (see the [relevant section](errors.md)) do successfully complete.

#### SQL Statements to Execute

_Lines 8, 11, 15, 19; string; mandatory one of `query` or `statement`_\
\
The actual SQL to execute.&#x20;

Specifying it as `query` means that a resultset is expected (typically, `SELECT` queries or queries with a `RETURNING` clause).&#x20;

Specifying a `statement` will not return a resultset, but a count of affected records.

#### Stored Query Reference

_Line 23; string; mandatory as the above_\
\
A `query` or a `statement` (see above) can consist of a reference to a Stored Query. They are prepended by a `#`. An error will occour if the S.Q. with an ID equal to the part after the `#` was not defined for this database.

See the [relevant section](stored-statements.md).

#### Parameter Values for the Query/Statement

_Lines 12, 20; object_\
\
If the query needs to be parametrized, named parameters can be defined in the statement using SQLite [syntax ](https://www.sqlite.org/c3ref/bind\_blob.html)(e.g. `:id` or `@id`), and the proper values for them must be specified here. You can specify values that do not match a parameter; they'll be ignored.

{% hint style="warning" %}
What happens if some parameter values aren't defined in the `values` object? If there are _less_ parameter values than expected, it will give an error. If they are correct in number, but some parameter names don't match, the missing parameters will be assigned a value of `null`.
{% endhint %}

#### Batch Parameter Values for a Statement

_Lines 24..27; list of objects_\
\
Only for `statement`s, more than one set of parameter values can be specified; the statement will be applied to them in a batch (by _preparing_ the statement).

#### NoFail: Don't Fail when Errors Occour

_Line 18; Boolean_\
\
When a query/statement fails, by default the whole transaction is rolled back and a response with a single error is returned (the first one for the whole transaction). Specifying this flag, the error will be reported for that statement, but the execution will continue and eventually be committed. See the [relevant page](errors.md) for more details.
