# Docker-Compose

干什么的？  批量容器编排，启动，管理

Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.

   定义一个`Dockerfile` 

2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.

   定义一个`docker-compose.yml`文件

3. Run `docker-compose up` and Compose starts and runs your entire app.

   使用docker-compose up 启动容器

## 栗子

启动一个web应用和一个redis应用，并进行容器互联

```yaml
version: '2.0'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```



启动一个wordpress博客应用

```yaml
version: '3.3'
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "80:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
```

### compose 官方例子

https://docs.docker.com/compose/gettingstarted/



## docker-compose.yml

docker-compose文件编写规则

https://docs.docker.com/compose/compose-file/#build

