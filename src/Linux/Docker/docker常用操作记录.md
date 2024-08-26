# docker常用操作记录

## 容器中执行apt update报错
解决办法
```
sed -i 's/stretch/buster/g' /etc/apt/sources.list
```

## 查看指定容器的近10条日志
```
docker logs --tail 10 -f mariadb1
```

## docker指定常用参数
- 指定网络
- 指定ipv4地址
- 绑定hosts
```
version: '3.8'

services:
  nginx:
    image: nginx:1.21.1
    container_name: nginx
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - /app/docker/nginx/nginx:/etc/nginx
      - /root/.acme.sh/*.nguone.eu.org_ecc/fullchain.cer:/etc/nginx/ssl/vaultwarden.crt:ro
      - /root/.acme.sh/*.nguone.eu.org_ecc/*.nguone.eu.org.key:/etc/nginx/ssl/vaultwarden.key:ro
    extra_hosts:
      - "remotework.nguone.eu.org:172.31.5.162"     
    networks:
      my_docker_network:
        ipv4_address: 172.16.238.6

networks:
  my_docker_network:
    external: true
```