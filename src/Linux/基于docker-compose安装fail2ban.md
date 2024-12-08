# 基于docker-compose安装fail2ban

## 安装rsyslog
fail2ban依赖rsyslog,安装并启动后，会产生日志文件 `/var/log/auth.log`
```shell
apt update
apt install rsyslog -y
rsyslogd
```

## docker-compose.yml
```shell
mkdir -p /app/docker/fail2ban/
cd /app/docker/fail2ban/

cat <<EOF > docker-compose.yml
version: '3.8'

services:
  fail2ban:
    image: crazymax/fail2ban:latest
    container_name: fail2ban
    restart: always
    network_mode: host
    cap_add:
      - NET_ADMIN
      - NET_RAW
    volumes:
      - ./data:/data
      - /var/log:/var/log:ro
EOF
```
## 启动
```shell
docker-compose up -d
```

## 配置SSH规则
```shell
vim data/jail.d/sshd.conf
#内容如下
 mkdir -p data/jail.d
cat <<EOF > data/jail.d/sshd.conf
[sshd]
enabled = true
chain = INPUT
port = ssh
filter = sshd[mode=aggressive]
logpath = /var/log/auth.log
maxretry = 3
findtime = 60     # 1 分钟内
bantime = 31536000 # 封禁 365 天（31,536,000 秒）
EOF
```

## 重启启动
```shell
docker-compose up -d
```

## 查看被ban的记录
```shell
docker exec fail2ban fail2ban-client status sshd
```
如下：
```shell
root@:/app/docker/fail2ban#  docker exec fail2ban fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	0
|  `- File list:	/var/log/auth.log
`- Actions
   |- Currently banned:	3
   |- Total banned:	3
   `- Banned IP list:	123.58.207.155 92.255.85.107 92.255.85.253
```




