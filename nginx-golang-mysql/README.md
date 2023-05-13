## Compose sample application

### Use with Docker Development Environments

You can open this sample in the Dev Environments feature of Docker Desktop version 4.12 or later.

[Open in Docker Dev Environments <img src="../open_in_new.svg" alt="Open in Docker Dev Environments" align="top"/>](https://open.docker.com/dashboard/dev-envs?url=https://github.com/docker/awesome-compose/tree/master/nginx-golang-mysql)

### Go server with an Nginx proxy and a MariaDB/MySQL database

Project structure:
```
.
├── backend
│   ├── Dockerfile
│   ├── go.mod
│   ├── go.sum
│   └── main.go
├── db
│   └── password.txt
├── proxy
│   └── nginx.conf
├── compose.yaml
└── README.md
```

[_compose.yaml_](compose.yaml)
```yaml
services:
  backend:
    build:
      context: backend
      target: builder
    ...
  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10-focal
    # If you really want to use MySQL, uncomment the following line
    #image: mysql:8
    ...
  proxy:
    image: nginx
    volumes:
      - type: bind
        source: ./proxy/nginx.conf
        target: /etc/nginx/conf.d/default.conf
        read_only: true
    ports:
    - 80:80
    ...
```
The compose file defines an application with three services `proxy`, `backend` and `db`.
When deploying the application, docker compose maps port 80 of the proxy service container to port 80 of the host as specified in the file.
Make sure port 80 on the host is not already being in use.

> ℹ️ **_INFO_**  
> For compatibility purpose between `AMD64` and `ARM64` architecture, we use a MariaDB as database instead of MySQL.  
> You still can use the MySQL image by uncommenting the following line in the Compose file   
> `#image: mysql:8`

## Deploy with docker compose (docker compose up -d)

```shell
$ docker compose up -d
Creating network "nginx-golang-mysql_default" with the default driver
Building backend
Step 1/8 : FROM golang:1.13-alpine AS build
1.13-alpine: Pulling from library/golang
...
Successfully built 5f7c899f9b49
Successfully tagged nginx-golang-mysql_proxy:latest
WARNING: Image for service proxy was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating nginx-golang-mysql_db_1 ... done
Creating nginx-golang-mysql_backend_1 ... done
Creating nginx-golang-mysql_proxy_1   ... done
```

![image](https://github.com/GOCity313/awesome-compose/assets/126258837/8a4de4f7-5ddc-48d5-adea-fc899003ac6a)

![image](https://github.com/GOCity313/awesome-compose/assets/126258837/7a382cc7-d722-48c5-8a6d-ed6cdc7badca)

## Expected result

Listing containers must show three containers running and the port mapping as below:

![image](https://github.com/GOCity313/awesome-compose/assets/126258837/00ca7019-4650-47ea-9d1f-4fafb11d516a)

```shell
$ docker compose ps
NAME                           COMMAND                  SERVICE             STATUS              PORTS
nginx-golang-mysql-backend-1   "/code/bin/backend"      backend             running
nginx-golang-mysql-db-1        "docker-entrypoint.s…"   db                  running (healthy)   3306/tcp
nginx-golang-mysql-proxy-1     "/docker-entrypoint.…"   proxy               running             0.0.0.0:80->80/tcp
l_db_1
```
![image](https://github.com/GOCity313/awesome-compose/assets/126258837/a51c6a5e-b643-4278-b970-ae15adffe663)

![image](https://github.com/GOCity313/awesome-compose/assets/126258837/4b17d941-1ea0-4582-94a1-e74351afa06b)

After the application starts, navigate to `http://localhost:80` in your web browser or run:
```shell
$ curl localhost:80
["Blog post #0","Blog post #1","Blog post #2","Blog post #3","Blog post #4"]
```
from Docker Deskstop :

![image](https://github.com/GOCity313/awesome-compose/assets/126258837/2a1dd6e3-71d0-49fa-90b9-8ee47785293c)


Stop and remove the containers
```shell
$ docker compose down
```
