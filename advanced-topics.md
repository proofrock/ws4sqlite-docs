# 🎓 Advanced Topics

### Pass parameters when opening a database

If you want to specify parameters when opening a db, you can appent to the [Path ](documentation/configuration-file.md#path)configuration of a database definition one or more parameters. They are in "raw" go-sqlite3 format, for more details you can see [here](https://github.com/mattn/go-sqlite3#connection-string).

For example, to set the cache size you can use:

```yaml
   path: ~/first.db?_cache_size=-16000
```

The setting will be reflected in the text printed at the server's start.

### Serve two instances of the same database

It can be done, and it makes perfect sense (e.g expose a read only instance without authentication, and a read-write one with auth). Just specify the same file in two db configurations, and use them normally.

See FAQ #5 at the [SQLite's site](https://www.sqlite.org/faq.html), and the test case `TestTwoServesOneDb()` in the code.

