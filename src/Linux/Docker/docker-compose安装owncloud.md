# docker-compose安装owncloud

## docker-compose.yml

```yml
version: "3"

services:
  owncloud:
    image: owncloud/server:${OWNCLOUD_VERSION}
    container_name: owncloud_server
    restart: always
    # ports:
    #   - ${HTTP_PORT}:8080 #因为使用nginx做了容器内部通信.所以不需要暴漏外网端口
    depends_on:
      - mariadb
      - redis
    environment:
      - OWNCLOUD_DOMAIN=${OWNCLOUD_DOMAIN}
      - OWNCLOUD_TRUSTED_DOMAINS=${OWNCLOUD_TRUSTED_DOMAINS}
      - OWNCLOUD_DB_TYPE=mysql
      - OWNCLOUD_DB_NAME=owncloud
      - OWNCLOUD_DB_USERNAME=owncloud
      - OWNCLOUD_DB_PASSWORD=Aexoh5ohs6tae5Ae
      - OWNCLOUD_DB_HOST=mariadb
      - OWNCLOUD_ADMIN_USERNAME=${ADMIN_USERNAME}
      - OWNCLOUD_ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - OWNCLOUD_MYSQL_UTF8MB4=true
      - OWNCLOUD_REDIS_ENABLED=true
      - OWNCLOUD_REDIS_HOST=redis
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - ./owncloud/data:/mnt/data
    networks:
      - nginx_ngu_network  # 使用和 nginx 相同的网络      

  mariadb:
    image: mariadb:10.11 # minimum required ownCloud version is 10.9
    container_name: owncloud_mariadb
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MARIADB_AUTO_UPGRADE=1
    command: ["--max-allowed-packet=128M", "--innodb-log-file-size=64M"]
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-u", "root", "--password=${MYSQL_ROOT_PASSWORD}"]
      interval: ${HEALTHCHECK_INTERVAL}
      timeout: ${HEALTHCHECK_TIMEOUT}
      retries: ${HEALTHCHECK_RETRIES}
    volumes:
      - ./mysql/mysql:/var/lib/mysql
    networks:
      - nginx_ngu_network  # 使用和 nginx 相同的网络   

  redis:
    image: redis:6
    container_name: owncloud_redis
    restart: always
    command: ["--databases", "1"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - ./redis:/data
    networks:
      - nginx_ngu_network  # 使用和 nginx 相同的网络   

networks:
  nginx_ngu_network:
    external: true  # 假设您的 nginx 容器在这个外部网络中运行
```

## .env

```shell
# OWNCLOUD
OWNCLOUD_VERSION=10.15
OWNCLOUD_DOMAIN=localhost:8080
OWNCLOUD_TRUSTED_DOMAINS=自己的域名
ADMIN_USERNAME=admin
ADMIN_PASSWORD=owncloud控制台密码
HTTP_PORT=38080

# DATABASE
MYSQL_ROOT_PASSWORD=自定义
MYSQL_USER=owncloud
MYSQL_PASSWORD=自定义
MYSQL_DATABASE=owncloud
MARIADB_AUTO_UPGRADE=1
HEALTHCHECK_INTERVAL=15s
HEALTHCHECK_TIMEOUT=5s
```

## 启动

```shell
docker-compose up -d
```

## 忘记了MySQL的Root密码

> 通过给MySQL的cmd中添加参数`--skip-grant-tables`,跳过权限表的验证.这种模式无需密码登录可以直接修改root密码.需要注意的是,做次操作的时候,不要映射外网端口,防止跳过权限表验证的期间.被人连接截取数据库信息

## 启动并启动MySQL

```shell
docker-compose up -d 
docker exec -it owncloud_mariadb bash

USE mysql;
UPDATE user SET authentication_string = PASSWORD('新密码') WHERE user = 'root';
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
FLUSH PRIVILEGES;
```

## 调整docker-compose.yml成正常模式,重新启动