# Docker Image OpenJDK Headless

Support hot reload using PM2

-   OS image: `Alpine Linux`
-   Base OpenJDK image: `zulu-openjdk` [source](https://github.com/zulu-openjdk/zulu-openjdk)
-   Hot Reload: `PM2` [Documentation](https://pm2.io/)
-   Supervisor [Documentation](http://supervisord.org/)

Setup docker image multiple platform:

-   [Multi-platform images](https://docs.docker.com/build/building/multi-platform/)
-   [Docker container driver](https://docs.docker.com/build/drivers/docker-container/)

## Platform build images:

-   amd64
-   arm64

Docker image spec:

| Tag | OpenJDK                        | Node Js    | PM2   |
| --- | ------------------------------ | ---------- | ----- |
| 20  | OpenJDK JRE Headless 20-latest | 18.17.1-r0 | 5.3.0 |
| 19  | OpenJDK JRE Headless 19-latest | 18.17.1-r0 | 5.3.0 |

## To Do's

-   ✅ Prepare Docker buildx setup
-   ✅ Manual build Docker image
-   ✅ Manual publish Docker image
-   ⬜ Automatic build Docker image (GitHub workflows)
-   ⬜ Automatic publish Docker image (GitHub workflows)

## Makefile Commands

Helping utility commands for simple build and push Docker image.

### Help command:

```bash
make help
```

### Build Multi-platform peparation:

```bash
make prepare
```

### Building multi-platform images (build & push):

```bash
make build VER=20 TAG=20
```

or

```bash
make build VER=20 TAG=latest
```

### Publish docker image to docker hub:

Before publish image, first login to docker hub via cli:

```bash
docker login
```

Publish docker image:

```bash
make push VER=20
```

Or

```bash
make push VER=20 TAG=latest
```

### Docker inspect the image:

```bash
make inspect VER=19 TAG=19
```

## Image Environment

| Environment variable | Description | Default |
| -------------------- | ----------- | ------- |
| `TIMEZONE`           | `timezone`  | `UTC`   |

## Docker Compose setup (running sample jar file)

Sample build `jar` file is here, spring-boot` [download](https://github.com/mkyong/spring-boot).

build jar file command: `mvn clean package`, sample project `spring-rest-hello-world`

```yaml
# docker-compose.yml
version: "3.7"

services:
    application:
        image: docker.io/asapdotid/openjdk:19
        expose:
            - 9000
        networks:
            - proxy
        environment:
            - TIMEZONE=Asia/Jakarta
        volumes:
            - ./project/jar:/app
            - ./cron.conf:/etc/supervisor/conf.d/cron.conf:ro
            - /scripts:/scripts:ro
        command: sh -c "sh /scripts/entrypoint.sh"
        labels:
            - traefik.enable=true
            - traefil.docker.network=proxy
            - traefik.http.routers.app-stats.entrypoints=https
            - traefik.http.routers.app-stats.rule=Host(`app.jogjascript.com`)
            - traefik.http.services.app-stats.loadbalancer.server.port=8080

networks:
    proxy:
        name: proxy
        driver: bridge
```

Script file for running jar file:

```bash
#!/bin/sh

set -o errexit
set -o nounset
set -o pipefail

print_line() {
  echo "-----------------------------------------------------------------------------"
}

setup_applications() {
  echo "Project ready for starting, please wait a moment to build and start!"
  if [[ -f "./spring-rest-hello-world-1.0.jar" ]]; then
    pm2-runtime start java --watch "./**/*" --name "app-stats" -- -jar ./spring-rest-hello-world-1.0.jar
  else
    echo "Project cannot be starting, please check .jar file!"
  fi
}

main() {
  print_line
  setup_applications
}

main $@
```

### Supervisor Config

Place config to `/etc/supervisor/conf.d/`
