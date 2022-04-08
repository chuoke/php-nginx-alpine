# An PHP Docker that works for Laravel also works for PHP.

---

[DOCKER HUB](https://hub.docker.com/r/chuoke/php-nginx-alpine)

## Example usage

Here's an example for my own use. And I prefer to use docker-compose.

`.env`:

```env
COMPOSE_PROJECT_NAME=some-site

APP_HOSTNAME=site.test

APP_CODE_PATH=/var/www/site

APP_RUN_ENV=local

```

`docker-compose.yml` :

```yml
version: "3.8"

networks:
  proxy:
    external: true
    name: traefik_default
  redis:
    external: true
    name: redis_default
  mysql:
    external: true
    name: mysql_default

services:
  php:
    image: chuoke/php-nginx-alpine:8.0
    container_name: ${COMPOSE_PROJECT_NAME}
    working_dir: ${APP_CODE_PATH}
    environment:
      - PGID=1000
      - PUID=1000
      - APP_CODE_PATH=${APP_CODE_PATH}
      - APP_RUN_ENV=${APP_RUN_ENV}
    volumes:
      - ../:${APP_CODE_PATH}
      # - ${APP_CODE_PATH}/vendor # For performance on Windows
      - ./nginx/site.conf:/etc/nginx/sites-available/site.conf
      # - ./nginx/nginx-octane.conf:/etc/nginx/sites-available/site.conf
      - ./nginx/logs:/var/log/nginx
      - ./supervisord/site.conf:/etc/supervisor/conf.d/supervisord.d/site.conf
    networks:
      - proxy
      - redis
      - mysql
    expose:
      - "80"
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik_default"
      - "traefik.http.routers.${COMPOSE_PROJECT_NAME}-router.rule=Host(`${APP_HOSTNAME}`)"
      - "traefik.http.routers.${COMPOSE_PROJECT_NAME}-router.entrypoints=web"
```

These example files like `./nginx/site.conf` and `./supervisord/site.conf` can be found in the sample `config` of the project.

---

## Build it from Dockerfile by self

Download this project to local and run command following:

```shell
docker build path -f Dockerfile -t [name]:[tag]

# example：
docker build ./8.0 -f ./8.0/Dockerfile -t php-nginx-alpine:8.0

# Push to Docker Hub
docker tag php-nginx-alpine:8.0 your-name/php-nginx-alpine:8.0
docker push your-name/php-nginx-alpine:8.0
```

## Export/Import image for uploading to server

#### Export

```
docker save [image] -o file.tar

```

#### Import

```
docker load -i file.tar
```

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
