# docker-compose网络
## docker-compose网络
### 网段隔离
在不同的文件下存放docker-compose.yml, 如mysql/docker-compose.yml, redis/docker-compose.yml, 当执行docker-compose up -d会自动创建一个新的网络. 通过```docker network ls```可以看到. 每一个网络相互隔离.
```
root@ec2-0804:# docker network ls
NETWORK ID     NAME                   DRIVER    SCOPE
8fbc5468dcab   bridge                 bridge    local
46e3affd9d7c   host                   host      local
e778a30f25d4   mysql_default          bridge    local
3b2337441be6   vaultwarden_default    bridge    local
```
### 网段打通
有时候, 并不想隔离, 比如zk和kafka共享一个网络. 比如应用微服务和mysql共享同一个网络
### 方法一
nginx和kafka存放在同一个nginx\docker-compose.yml中,启动后自然就是同一个网络. 
如
```
version: '3.8'

services:
  nginx:
    image: zookeeper
    container_name: nginx

  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden 
```
启动后, 会创建一个新的网络nginx_default,前缀来自于当前docker-compose.yml所在的文件夹的名字

### 方法二
手动创建一个网络
```
docker network create --driver=bridge --subnet=172.16.238.0/24 --gateway=172.16.238.1 my_docker_network
```
在不同的docker-compose.yml中指定想要加入的网络
```
version: '3.8'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      # 
      - WEBSOCKET_ENABLED=true
      # 禁止邀请
      - INVITATIONS_ALLOWED=false
      # 禁止注册. 注意, 初始化搭建的时候.需要改成true,启动后注册用户.注册成功后,改成false,重新执行docker-compose up -d 
      - SIGNUPS_ALLOWED=false
      - ADMIN_TOKEN=AqaiTTm4LbMaTCyVFLTFzwmHPxxYa4Kg
    volumes:
      - /app/docker/vaultwarden/data:/data/
    #ports:
     # 由于vaultwarden和nginx部署在了同一个网络下,所以用服务名通信了.因此不需要暴漏80端口 
     #- "80:80"
    networks:
      - my_docker_network

networks:
  my_docker_network:
    external: true
```