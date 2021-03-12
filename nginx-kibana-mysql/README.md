## Compose Three (3) Docker Containers
### Kibana with an Nginx proxy and a MySQL database

Project structure:
```
.

├── mysql
│   └── password.txt
├── nginx
│   ├── conf
│   └── Dockerfile
├── docker-compose.yaml
└── README.md
```

[_docker-compose.yaml_](docker-compose.yaml)
```
services:
 kibana:
    image: kibana:6.4.2
    container_name: kib
    ports:
      - "5601:5601"
    networks:
      - mysql
      - nginx
  mysql:
    image: mysql:8.0.19
    command: '--default-authentication-plugin=mysql_native_password'
    secrets:
      - db-password
    environment:
      - MYSQL_DATABASE=example
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db-password
    networks:
      - kibana
      - nginx
  nginx:
  image: nginx
    build: nginx
    ports:
      - 80:80
    networks:
      - mysql
      - kibana
networks:
      - mysql
      - kibana
      - nginx
secrets:
  db-password:
    file: db/password.txt
    ...
```
The compose file creates three Docker images, run containers from them, configures Nginx.
The three docker containers are able to ping each other.
.
Note: Make sure port 80 and 5601:5601 on the host are not already being in use.

## Deploy with docker-compose

```
$ docker-compose up -d
Creating network "nginx-kibana-mysql_default" with the default driver
Building 
Step 1/8 : FROM :1.13-alpine AS build
1.13-alpine: Pulling from library
...
Successfully built 5f7c899f9b49
Successfully tagged nginx-kibana-mysql_proxy:latest
WARNING: Image for service proxy was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating nginx-kibana-mysql_mysql_1 ... done
Creating nginx-kibana-mysql_nginx_1   ... done
```

## Expected result

Listing containers must show three containers running and the port mapping as below:
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
8906b14c5ad1        nginx-kibana-mysql_nginx     "nginx -g 'daemon of…"   2 minutes ago       Up 2 minutes        0.0.0.0:80->80/tcp    nginx-kibana-mysq
l_nginx_1
13e0e0a7715a        nginx-kibana-mysql_kibana  "/server"                2 minutes ago       Up 2 minutes        8000/tcp              nginx-kibana-mysq
l_kibana_1
ca8c5975d205        mysql:5.7                    "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        3306/tcp, 33060/tcp   nginx-kibana-mysq
l_mysql_1
```


Stop and remove the containers
```
$ docker-compose down
```
