# Linux常用技巧

### 每次vim的时候总是粘贴不了.需要输入:set mouse-=a回车,才行
解决办法, vim ~/.vimrc 内容如下
```
set mouse-=a
```
保存后, ```souce ~/.vimrc```, 再次编辑文件, 就可以粘贴了

### apt安装ping
```
apt install iputils-ping
```

### 给泛解析加一个匹配不到的预留映射
vim \*.nguone.eu.org.conf
```
server {
    listen 80;
    listen 443 ssl;
    server_name *.nguone.eu.org;
    ssl_certificate /etc/nginx/ssl/vaultwarden.crt;
    ssl_certificate_key /etc/nginx/ssl/vaultwarden.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    # 可以选择进行适当的处理或返回404
    # 开启HSTS后能够提升到A+. SSL评估 https://myssl.com/
    add_header Strict-Transport-Security "max-age=31536000";
    return 404;
}

```
## Nginx Docker 部署
创建网络
```
docker network create --driver=bridge --subnet=172.16.238.0/24 --gateway=172.16.238.1 my_docker_network
```

```
version: '3.8'

services:
  nginx:
    image: nginx:1.21.1
    container_name: nginx
    ports:
      - 80:80
      - 443:443
    volumes:
      - /app/docker/nginx/nginx:/etc/nginx
      - /root/.acme.sh/*.nguone.eu.org_ecc/fullchain.cer:/etc/nginx/ssl/vaultwarden.crt:ro
      - /root/.acme.sh/*.nguone.eu.org_ecc/*.nguone.eu.org.key:/etc/nginx/ssl/vaultwarden.key:ro     
    networks:
      - my_docker_network

networks:
  my_docker_network:
    external: true
```

### Nginx https 配置
当前nginx版本
```
nginx -v
nginx version: nginx/1.21.1
```

vaultwarden.conf示例配置如下
```
server {
    listen 80; #侦听80端口，如果强制所有的访问都必须是HTTPs的，这行需要注销掉
    listen 443 ssl; 
    server_name vaultwarden.nguone.eu.org;
    ssl_certificate /etc/nginx/ssl/vaultwarden.crt;
    ssl_certificate_key /etc/nginx/ssl/vaultwarden.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    client_max_body_size 1024m;
    # 开启HSTS后能够提升到A+. SSL评估 https://myssl.com/
    add_header Strict-Transport-Security "max-age=31536000";
    
    location / {
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://vaultwarden;
    }
}
```
