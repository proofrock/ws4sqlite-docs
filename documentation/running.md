# 🏃 Running

Running WS4SQLite can be done via the commandline, and it's possible to specify its behaviour via commandline parameters.

### Commandline Parameters

This is a complete commandline for WS4SQLite:

```
ws4sqlite --cfg-dir CFGDIR --bind-host 0.0.0.0 --port 12321
```

Of course, the usual `--help` and `--version` are supported. Let's discuss the other commandline parameters one by one.

#### `--cfg-dir`

Specifies where the [`config.yaml`](configuration-file.md) file is to be found. You can specify a relative or absolute path, and it's possible to use `~` to expand the user home.

If not specified, uses the current working directory.

#### `--bind-host`

Host to bind to. Defaults to `0.0.0.0`, meaning to accept connections from any local network interface. Different values are possible to restrict connections to a particular subnet.

#### `--port`

Port to use for incoming network communication. Defaults to `12321`.

### Output

ws4sqlite will parse the YAML [config file](configuration-file.md), attempt to open and connect to the databases, creating their respective files as needed. Then it will output a summary of all the configurations, like this:

```
ws4sqlite x.y.z
- Serving database 'db1' from /home/mano/first.db?_cache_size=-16000&mode=ro&immutable=1&_query_only=1
  + Read only
  + Authorization enabled, with 2 credentials
  + Maintenance scheduled at 12:00 am
- Serving database 'db2' from :memory:?_journal=WAL
  + Using WAL
  + Strictly using stored statements
  + With 2 stored statements
  + 4 init statements performed
  + Authorization enabled, with query
  + CORS Origin set to https://myownsite.com
- Web Service listening on 0.0.0.0:12321
```
