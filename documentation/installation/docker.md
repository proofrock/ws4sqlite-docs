# 🐳 Docker

ws4sqlite provides a standard Docker image, based on Alpine, at [Docker Hub](https://hub.docker.com/r/germanorizzo/ws4sqlite).

It's available for amd64, ARM and ARM64. Append `-arm` or `-arm64` to the usual tags (e.g. `:latest-arm`, `:0.11.2-arm64`).

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
