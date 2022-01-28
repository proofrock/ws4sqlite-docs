# 🐳 Docker

WS4SQLite provides a standard Docker image, based on Alpine, at [Docker Hub](https://hub.docker.com/r/germanorizzo/ws4sqlite).

It's available both for amd64 and for arm (32); for the latter, append `-arm` to the tag (e.g. `:latest-arm` or `:0.9.0-arm`).

Here are the relevant configurations:

|              |         |                           |
| ------------ | ------- | ------------------------- |
| Exposed port | 12321   | Fixed; remap it with `-p` |
| Config dir   | `/data` | Fixed; remap it with `-v` |

#### Example

```
docker run -d \
 --restart=unless-stopped \
 --name=ws4sqlite \
 -p 8080:12321 \
 -v /mnt/DockerHome/ws4sqlite_config:/data \
 germanorizzo/ws4sqlite:latest
```

This command will install and run ws4sqlite, configuring it to:

* Use port 8080 (Line 4);
* Config dir at `/mnt/DockerHome/ws4sqlite_config` (Line 5).

The rest of the lines in this example are standard Docker.

#### Caveats

* You have to insert your own `config.yaml` (see the [docs](../configuration-file.md)) in the directory mapped to `/data`;
* Of course, the same is true for the database files;
* The path for the database file should be absolute, and remember that your directory is mapped to `/data` so probably _that_ directory must be used.
