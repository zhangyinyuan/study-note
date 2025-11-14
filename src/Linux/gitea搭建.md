# docker gitea搭建

## 配置Nginx

> git.conf

```ini
server {
    listen 80; #侦听80端口，如果强制所有的访问都必须是HTTPs的，这行需要注销掉
    listen 443 ssl;
    server_name git.xxx.com;
    ssl_certificate /root/.acme.sh/*.xxx.com_ecc/fullchain.cer;
    ssl_certificate_key /root/.acme.sh/*.xxx.com_ecc/*.xxx.com.key;
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
        proxy_pass http://127.0.0.1:3000;
    }
}
```

## Nginx配置重载

```shell
nginx -s reload
```

## docker-compose.yml
```
version: '3.8'

services:
  gitea:
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/gitea/gitea:1.22.3 
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=sqlite3  
    restart: always
    volumes:
      - ./data:/data
    ports:
      - "3000:3000" # Gitea 的 Web UI 端口,也是通过http 克隆项目的端口
      - "2222:22" # 如果这里使用22端口. 拉取项目的时候可以省去端口号.因为通过ssh key clone项目用的是22端口
```

## 启动
> 需要注意的时候, git要使用22端口.默认情况下22端口被宿主机的ssh占用.所以需要修改宿主机下的Port 22 为其他端口, 修改后切记放行新的ssh端口,防止无法ssh .`systemctl resetart sshd` 重新启动sshd
```
docker-compose up -d
```

## 特别注意

> 如果启动后访问 https://git.xxx.com, 自动跳转到其他的域名，可能的解决方案是：
>
> 编辑 /etc/nginx/sites-available/default
>
> 直接注释掉`server_name _;`
>
> 然后重载Nginx配置

## 启动成功后,访问http://宿主机IP:3000访问web控制台.注册一个管理员

> 需要注意的是,如果端口改成非22端口了,比如2222.那么初始化安装的时候, [SSH Server Port]要修改ssh端口为2222,而不是默认的22
> 这里的修改的2222是指以后拉取项目用的端口地址,而不是容器的内部端口,内部端口仍然是22
### 注册好管理员后, 通过命令行把注册禁用掉
```
docker exec -it gitea bash

vi /data/gitea/conf/app.ini

# 找到 DISABLE_REGISTRATION, 将值从false,改为true.
```
修改完后,退出docker容器. 重启gitea `docker restart gitea`

### ssh key clone
gitea 默认使用3071长度的ssh key拉取项目.所以Windows或者Linux默认的2048无法使用.

#### 生成3071长度的密钥
> 3071 表示 位长的 RSA 密钥

```
ssh-keygen -t rsa -b 3071 
```
生成之后, 登录gitea web控制台将公钥添加到SSH / GPG Kyes 中
就可以正常clone项目了


### 补充说明
## 通过ssh拉取项目
> 推荐搭建gitea的时候使用22端口
- 使用22端口拉取项目 `git clone git@git服务器IP/项目组或者用户名/work-record.git`
- 使用2222端口拉取项目 `git clone ssh://git@git服务器IP:2222/项目组或者用户名/work-record.git`
