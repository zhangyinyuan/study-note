# docker-compose搭建Nextcloud 
## docker-compose.yml
```docker
version: '3.9'

services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    ports:
      - "8080:80"  # 将主机的 8080 端口映射到容器的 80 端口
    volumes:
      - nextcloud_data:/var/www/html
      - ./config/config.php:/var/www/html/config/config.php
    environment:
      - MYSQL_HOST=mysql5.7.28 # 使用你的 MySQL 容器名称作为主机名
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud_user
      - MYSQL_PASSWORD=Aphu2jieChie6mo2
    networks:
      - mysql_default

networks:
  mysql_default:
    external: true

volumes:
  nextcloud_data:
```

### ./config/config.php配置

支持HTTPS域名访问,输入自己的域名

```xml
  'trusted_domains' => 
  array (
    0 => 'nextcloud.mydomain.xyz',
  ),
```

然后使用Nginx转发即可, 最后重启Nginx,即可完成https访问.可以参考**[Linux常用技巧](../Linux常用.md)**篇的**Nginx https配置**