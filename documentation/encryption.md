# 🗝 Encryption

ws4sqlite includes a functionality not directly related to SQLite, but useful in many use cases. It allows you to specify a set of column that, when executing a statement, can be encrypted before being saved to a db; and conversely decrypted when reading them in a query, before returning them to a client. It can also compress the string before encryption, if it's very long.

In the request it's necessary to specify a password to encrypt/decrypt the data, and of course it must be the same when calling encryption and decryption.

The data saved to the database are normal `TEXT` data, so a text/varchar column is enough.

### How To

Let's suppose to have a table called `TEMP`, with an integer primary key named `ID` and a text column called `VAL`.

A simple request that saves some data in encrypted form and reads them both undecrypted and decrypted is this:

```json
{
    "transaction": [
        {
            "statement": "INSERT INTO TEMP VALUES (:id, :val)",
            "values": { "id": 1, "val": "Secret!!" },
            "encoder": {
                "pwd": "myPassword",
                "compressionLevel": 6,
                "columns": [ "val" ]
            }
        }, {
            "query": "SELECT ID, VAL FROM TEMP"
        }, {
            "query": "SELECT ID, VAL FROM TEMP",
            "decoder": {
                "pwd": "myPassword",
                "columns": [ "VAL" ]
            }
        }
    ]
}
```

Relevant structures are at Lines 6-10 for encryption, and at Lines 15-18 for decryption.

Notice that:

* The password is the same (Lines 7 and 16) and is specified as clear text (be sure to use a secure connection);
* What to encode is specified as a list of fields from the `values` or `valuesBatch` nodes (Line 9). Fields that are not in this list will be saved as they are.
* What to decode is specified as a list of fields from the resultset (Line 17).
* Compression is requested, at Line 8. This is level `6`; valid levels are `1`-`19`. A level of 0 (or omitting the node) disable the compression.
  * Compression is applied only if it actually reduces the size of the text; if it doesn't, it's discarded.
* You don't need to specify that data are compressed, when decoding.

The response is as this:

```json
{
    "results": [
        {
            "success": true,
            "rowsUpdated": 1
        }, {
            "success": true,
            "resultSet": [
                {
                    "ID": 1,
                    "VAL": "AQC4Vv9iFNBAxGXD7pCWfF/oZo3R6wxDx585hokQUcjO5PhD9lul6KB7G50rim7f7mw="
                }
            ]
        }, {
            "success": true,
            "resultSet": [
                {
                    "ID": 1,
                    "VAL": "Secret!!"
                }
            ]
        }
    ]
}
```

Notice that:

* The 2nd request doesn't specify a decoder, so the response return the "raw" data in the field `VAL`. This is clearly not what you want;
* The 3rd request does specify decryption, so the original value is returned;
* If we had specified the wrong decryption password, an error would have been generated, of course it's possible to use `noFail` to manage it.

### Implementation

For this function, the [crypgo ](https://github.com/proofrock/crypgo)library was used. Please refer to its documentation for the algorythms used.
