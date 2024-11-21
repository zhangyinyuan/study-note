# docker gitea搭建

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
> 3071 位长的 RSA 密钥

```
ssh-keygen -t rsa -b 3071 
```
生成之后, 登录gitea web控制台将公钥添加到SSH / GPG Kyes 中
就可以正常clone项目了


### 补充说明
## 通过ssh拉取项目
> 推荐搭建gitea的时候使用22端口
- 使用22端口拉取项目 `git clone git@git服务器IP/项目组或者用户名/work-record.git`
- 使用2222端口拉取项目 `ssh://git@git服务器IP:2222/项目组或者用户名/work-record.git`
