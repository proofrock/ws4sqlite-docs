# 🐳 Docker

ws4sqlite provides a standard Docker image, based on Alpine, at [Docker Hub](https://hub.docker.com/r/germanorizzo/ws4sqlite).

It's available both for amd64 and for arm (32); for the latter, append `-arm` to the tag (e.g. `:latest-arm` or `:0.11.0-arm`). 

Here are the relevant configurations:

|              |         |                           |
| ------------ | ------- | ------------------------- |
| Exposed port | 12321   | Fixed; remap it with `-p` |
| Config dir   | `/data` | Fixed; remap it with `-v` |

#### Example

```bash
docker run -d \
 --restart=unless-stopped \
 --name=ws4sqlite \
 -p 8080:12321 \
 -v /mnt/DockerHome/myDir:/data \
 germanorizzo/ws4sqlite:latest \
 --db /data/testDb.db
```

This command will install and run ws4sqlite, configuring it to:

* Use port 8080 (Line 4);
* Map a local dir to path `/data` in the docker environment (Line 5).
* Use free cli arguments, as it was the ws4sqlite binary (Line 7).

The rest of the lines in this example are standard Docker.

#### Caveats

* You have to reference database and companion files that are in the directory mapped to `/data` as they were in `/data`;
* The path for the database file should be absolute.
