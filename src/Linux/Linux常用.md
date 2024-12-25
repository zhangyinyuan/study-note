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

添加一个http登录认证
配置如下:
```
server {
    listen 80; #侦听80端口，如果强制所有的访问都必须是HTTPs的，这行需要注销掉
    server_name work;
    client_max_body_size 1024m;
    # 开启HSTS后能够提升到A+. SSL评估 https://myssl.com/
    add_header Strict-Transport-Security "max-age=31536000";
    
    location / {
        # 启用 HTTP Basic Authentication
        auth_basic "Restricted Access";  # 提示信息
        auth_basic_user_file /etc/nginx/.htpasswd;  # 指定存放用户名和密码的文件        
	proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://host.docker.internal:3000;
    }
}
```
生成验证密码,保存在`/etc/nginx/.htpasswd`
> 密码错误, 会导致nginx提示500错误
```
htpasswd -c -b /etc/nginx/.htpasswd nguone AiQuaichei2Eer2C
```

### location

```shell
location = /abc {
    add_header Content-Type text/plain;
    add_header X-Server-IP $server_addr always;

    return 200 'Success';
}
```

解释:

- `location = /abc`：表示精确匹配 `/abc`，只有请求的 URI 正好是 `/abc` 时才会匹配这个块.其他以 `/abc/` 开头的路径（比如 `/abc/jlljlk`）将不会匹配这个配置,将会返回404
- 如果设置为`/abc`,那么访问`/abc`以及`/abc`开头的任何路径都会当做`/abc`处理
- `add_header Content-Type text/plain` 表示返回的信息是纯文本
- `add_header X-Server-IP $server_addr always` 强制在响应头信息中返回**nginx**的**IP**

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

## 重置终端root@后的别名.
```
# 重置PS1
echo "export PS1='[\u@\h \W]\$ '" >> ~/.bashrc && exec bash
```

## 端口扫描
### 安装
```shell
sudo apt-get install nmap   # Debian/Ubuntu 系统
sudo yum install nmap       # CentOS/Red Hat 系统
```

### 扫描单个主机
```shell
nmap -sT 192.168.1.1
```

### 扫描整个网段
```shell
nmap -p 1-65535 192.168.1.0/24
```

### 查看硬盘类型
```shell
cat /sys/block/sda/queue/rotational
#如果输出是 1，则表示该磁盘是 旋转硬盘（HDD）。
#如果输出是 0，则表示该磁盘是 固态硬盘（SSD）。
```

### 查看所有自启服务列表
```shell
systemctl list-units --type=service
```

### docker 容器安装telnet
```shell
# 查看操作系统信息
cat /etc/os-release

# alpine 安装telnet
apk add busybox-extras
```

### grafana 重置admin密码
```shell
grafana-cli admin reset-admin-password <new password>
```