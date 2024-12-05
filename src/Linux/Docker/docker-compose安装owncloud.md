# docker-compose安装owncloud
```yml
version: '3'

services:
  owncloud:
    image: owncloud/server:latest
    container_name: owncloud
    restart: always
    environment:
      - MYSQL_HOST=mysql5.7.28
      - MYSQL_DATABASE=owncloud
      - MYSQL_USER=owncloud
      - MYSQL_PASSWORD=ohPo4
      - OWNCLOUD_DOMAIN=drive.one.xyz
      - HTTP_PORT=8080
    networks:
      - nginx_ngu_network  # 使用和 nginx 相同的网络

  # mysql:
  #   image: mysql:5.7
  #   container_name: owncloud-db
  #   restart: always
  #   environment:
  #     - MYSQL_ROOT_PASSWORD=rootpassword
  #     - MYSQL_DATABASE=owncloud
  #     - MYSQL_USER=owncloud
  #     - MYSQL_PASSWORD=yourpassword
  #   networks:
  #     - nginx_ngu_network  # 使用和 nginx 相同的网络

    networks:
      - nginx_ngu_network  # 使用和 nginx 相同的网络

networks:
  nginx_ngu_network:
    external: true  # 假设您的 nginx 容器在这个外部网络中运行
```