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

## 匹配进程的名字并且全部杀掉进程
如杀掉所有kafka进程
```
pgrep -f kafka | xargs kill -9
```
**解释**
- **pgrep -f kafka**：查找所有匹配 kafka 的进程 PID。
- **xargs kill -9**：将找到的 PID 传递给 kill -9，强制终止所有这些进程。

这个命令将会找到所有 Kafka 相关的进程，并使用 kill -9 发送信号强制终止它们。

**注意事项**
- 确保匹配准确：确认 pgrep -f kafka 只会匹配到 Kafka 进程，避免误杀其他重要进程。
- 检查进程：在执行命令前，可以运行 pgrep -f kafka 先查看将会被杀掉的进程。
- 这种方式是一条命令的解决方案，但请务必在执行之前确认对系统的影响。

## 在不关闭shell的情况下重新启动Bash Shell
```
exec bash
```
**主要作用**
- **应用配置更改**
  - 当你修改了 ~/.bashrc 或其他 shell 配置文件（如 ~/.bash_profile）后，使用 exec bash 可以使这些更改立即生效，而无需重新登录或重启终端
- **刷新环境**
  - 有时需要重新加载环境变量或 shell 配置时，exec bash 会刷新当前 shell 的所有设置，使得新的环境变量或配置被加载