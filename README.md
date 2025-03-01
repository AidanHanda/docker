# Monica's docker image

[![Docker Pulls](https://img.shields.io/docker/pulls/library/monica.svg)](https://hub.docker.com/_/monica/)
![Monica's docker](https://github.com/monicahq/docker/workflows/Monica's%20docker/badge.svg)

[![amd64 build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/amd64/job/monica.svg?label=amd64)](https://doi-janky.infosiftr.net/job/multiarch/job/amd64/job/monica)
[![arm32v5 build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/arm32v5/job/monica.svg?label=arm32v5)](https://doi-janky.infosiftr.net/job/multiarch/job/arm32v5/job/monica)
[![arm32v6 build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/arm32v6/job/monica.svg?label=arm32v6)](https://doi-janky.infosiftr.net/job/multiarch/job/arm32v6/job/monica)
[![arm32v7 build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/arm32v7/job/monica.svg?label=arm32v7)](https://doi-janky.infosiftr.net/job/multiarch/job/arm32v7/job/monica)
[![arm64v8 build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/arm64v8/job/monica.svg?label=arm64v8)](https://doi-janky.infosiftr.net/job/multiarch/job/arm64v8/job/monica)
[![i386 build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/i386/job/monica.svg?label=i386)](https://doi-janky.infosiftr.net/job/multiarch/job/i386/job/monica)
[![mips64le build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/mips64le/job/monica.svg?label=mips64le)](https://doi-janky.infosiftr.net/job/multiarch/job/mips64le/job/monica)
[![ppc64le build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/ppc64le/job/monica.svg?label=ppc64le)](https://doi-janky.infosiftr.net/job/multiarch/job/ppc64le/job/monica)
[![s390x build status badge](https://img.shields.io/jenkins/s/https/doi-janky.infosiftr.net/job/multiarch/job/s390x/job/monica.svg?label=s390x)](https://doi-janky.infosiftr.net/job/multiarch/job/s390x/job/monica)


[![MonicaHQ](https://avatars0.githubusercontent.com/u/25832602?s=200&v=4)](https://hub.docker.com/_/monica)

Monica can run with Docker images.

## Prerequisites

You can use [Docker](https://www.docker.com) and [docker-compose](https://docs.docker.com/compose/) to pull or build
and run a Monica image, complete with a self-contained MySQL database.
This has the nice properties that you don't have to install lots of software directly onto your system, and you can be up and running
quickly with a known working environment.

For any help about how to install Docker, see their [documentation](https://docs.docker.com/install/)

## Use Monica docker image

There are two versions of the image you may choose from.

The `apache` tag contains a full Monica installation with an apache webserver. This points to the default `latest` tag too.

The `fpm` tag contains a fastCGI-Process that serves the web pages. This image should be combined with a webserver used as a proxy, like apache or nginx.

### Using the apache image

This image contains a webserver that exposes port 80. Run the container with:
```console
docker run -d -p 8080:80 monica
```

### Using the fpm image

This image serves a fastCGI server that exposes port 9000. You may need an additional web server that can proxy requests to the fpm port 9000 of the container.
Run this container with:
```console
docker run -d -p 9000:9000 monica:fpm
```

### Persistent data storage

To have a persistent storage for your datas, you may want to create volumes for your db, and for monica you will have to save the `/var/www/html/storage` directory.

Run a container with this named volume:
```console
docker run -d 
-v monica_data:/var/www/html/storage
monica
```

### Connect to a mysql database

Monica needs a database connection, and currently supports mysql only. Run these to have a running environment:
```console
mysqlCid="$(docker run -d \
 -e MYSQL_RANDOM_ROOT_PASSWORD=true \
 -e MYSQL_DATABASE=monica \
 -e MYSQL_USER=homestead \
 -e MYSQL_PASSWORD=secret \
 "mysql:5.7")"
docker run -d \
 --link "$mysqlCid":mysql \
 -e DB_HOST=mysql \
 -p 8080:80 \
 monica
```

If you want to use a mysql installation that already exists, you can pass the database environment variables directly into the `docker run` command, with `DB_HOST` indicating the IP address of the MySQL database. Ensure all permissions are granted to the user specified in the enviornment variable `DB_USERNAME`.

```console
docker run -d \
 -e DB_PORT=3306 \
 -e DB_DATABASE=monica \
 -e DB_USERNAME=homestead \
 -e DB_PASSWORD=secret \
 -e DB_HOST=0.0.0.0 \
 -p 8080:80 \
 monica
```

Wait until all migrations are done and then access Monica at http://localhost:8080/ from your host system.
If this looks ok, add your first user account.

### Run commands inside the container

Like every Laravel application, the `php artisan` command is very usefull for Monica.
To run a command inside the container, run

```sh
docker exec CONTAINER_ID php artisan COMMAND
```

or for docker-compose
```sh
docker-compose exec monica php artisan COMMAND
```
where `monica` is the name of the service in your `docker-compose.yml` file.


## Running the image with docker-compose

See some examples of docker-compose possibilities in the [example section](/.examples).

---

### Apache version

This version will use the apache image and add a mysql container. The volumes are set to keep your data persistent. This setup provides **no ssl encryption** and is intended to run behind a proxy.

Make sure to pass in values for `APP_KEY` variable before you run this setup.

Set `APP_KEY` to a random 32-character string. You can for instance copy and paste the output of `echo -n 'base64:'; openssl rand -base64 32`.

1. Create a `docker-compose.yml` file

```yaml
version: "3.4"

services:
  app:
    image: monica
    depends_on:
      - db
    ports:
      - 8080:80
    environment:
      - APP_KEY=
      - DB_HOST=db
      - DB_USERNAME=usermonica
      - DB_PASSWORD=secret
    volumes:
      - data:/var/www/html/storage
    restart: always

  db:
    image: mysql:5.7
    environment:
      - MYSQL_RANDOM_ROOT_PASSWORD=true
      - MYSQL_DATABASE=monica
      - MYSQL_USER=usermonica
      - MYSQL_PASSWORD=secret
    volumes:
      - mysql:/var/lib/mysql
    restart: always

volumes:
  data:
    name: data
  mysql:
    name: mysql
```

2. Set a value for `APP_KEY` variable before you run this setup. You can for instance copy and paste the output of
   ```sh
   echo -n 'base64:'; openssl rand -base64 32
   ```

3. Run
   ```sh
   docker-compose up -d
   ```

   Wait until all migrations are done and then access Monica at http://localhost:8080/ from your host system.
   If this looks ok, add your first user account.

4. Run this command once:
   ```sh
   docker-compose exec app php artisan setup:production
   ```

### FPM version

When using FPM image, you will need another container with a webserver to proxy http requests. In this example we use nginx with a basic container to do this.

1. Download `nginx.conf` and `Dockerfile` file for nginx image. An example can be found on the [`example section`](/.examples/nginx-proxy/web/)
   ```sh
   mkdir web
   curl -sSL https://raw.githubusercontent.com/monicahq/docker/master/.examples/nginx-proxy/web/nginx.conf -o web/nginx.conf
   curl -sSL https://raw.githubusercontent.com/monicahq/docker/master/.examples/nginx-proxy/web/Dockerfile -o web/Dockerfile
   ```
The `web` container image should be pre-build before each deploy with: `docker-compose build`

2. Create a `docker-compose.yml` file

```yaml
version: "3.4"

services:
  app:
    image: monica:fpm
    depends_on:
      - db
    environment:
      - APP_KEY=
      - DB_HOST=db
      - DB_USERNAME=usermonica
      - DB_PASSWORD=secret
    volumes:
      - data:/var/www/html/storage
    restart: always
  
  web:
    build: ./web
    ports:
      - 8080:80
    depends_on:
      - app
    volumes:
      - data:/var/www/html/storage:ro
    restart: always

  db:
    image: mysql:5.7
    environment:
      - MYSQL_RANDOM_ROOT_PASSWORD=true
      - MYSQL_DATABASE=monica
      - MYSQL_USER=usermonica
      - MYSQL_PASSWORD=secret
    volumes:
      - mysql:/var/lib/mysql
    restart: always

volumes:
  data:
    name: data
  mysql:
    name: mysql
```

3. Set a value for `APP_KEY` variable before you run this setup. You can for instance copy and paste the output of
   ```sh
   echo -n 'base64:'; openssl rand -base64 32
   ```

4. Run
   ```sh
   docker-compose up -d
   ```

   Wait until all migrations are done and then access Monica at http://localhost:8080/ from your host system.
   If this looks ok, add your first user account.

5. Run this command once:
   ```sh
   docker-compose exec app php artisan setup:production
   ```

## Make Monica available from the internet 

To expose your Monica instance for the internet, it's important to set `APP_ENV=production` in your `.env` file or environment variables. In this case `https` scheme will be **mandatory**.

### Using a proxy webserver on the host

One way to expose your Monica instance is to use a proxy webserver from your host with SSL capabilities. This is possible with a reverse proxy.

### Using a proxy webserver container

See some examples of docker-compose possibilities in the [example section](/.examples) to show how to a proxy webserver with ssl capabilities.
