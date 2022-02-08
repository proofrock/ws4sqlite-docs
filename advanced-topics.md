# 🎓 Advanced Topics

### Serve two instances of the same database file

It can be done, and it makes perfect sense (e.g expose a read only instance without authentication, and a read-write one with auth). Just specify the same file in two db configurations, and use them normally.

See FAQ #5 at the [SQLite's site](https://www.sqlite.org/faq.html), and the test case `TestTwoServesOneDb()` in the code.

