---
title: docker入门-搭建wordpress
---

> 运行mysql
```bash
docker pull mysql
docker run --name mysql_wp -e MYSQL_ROOT_PASSWORD=king -d mysql 数据库名:mysql_wp 密码:king 端口:3306
docker exec -it [container_id] /bin/sh 进入容器
mysql -uroot -p king  测试mysql 是否正常运行

```
> 运行wordpress

```bash
docker pull wordpress
docker run --name some-wordpress --link mysql_wp:mysql -p 8080:80 -d wordpress 连接mysql ，暴露外部端口为8080
http://localhost:8080 访问wordpress

```

> docker-compse.yml

```bash
version: '3'
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
       - "8080:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data:
运行: docker-compose up -d
访问:http://localhost:8080

```