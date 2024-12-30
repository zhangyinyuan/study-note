# docker常用操作记录

## 禁止docker创建容器时自动开放端口
```
sudo tee /etc/docker/daemon.json <<EOF
{
  "iptables": false
}
EOF
systemctl restart docker
```

## ufw允许nginx容器访问所有端口
> 用于ngix做负载均衡、转发
```shell
sudo ufw allow from $(docker inspect nginx | grep '"IPv4Address"' | awk -F '": "' '{print $2}' | tr -d '",') to any comment "允许nginx容器访问所有端口"
```

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