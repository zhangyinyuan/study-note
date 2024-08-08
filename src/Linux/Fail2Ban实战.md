## 环境
```
root@x:~# cat /etc/issue
Debian GNU/Linux 12 \n \l
```

## Fail2Ban 安装相关
```
# 安装
apt install fail2ban -y

# 启动进程
systemctl start fail2ban 

# 开机启动
systemctl enable fail2ban
```

## 查看状态
```
systemctl status fail2ban.service
```
发现没有启动成功,控制台如下:
```
× fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Mon 2024-08-05 03:41:24 UTC; 3 days ago
   Duration: 195ms
       Docs: man:fail2ban(1)
   Main PID: 1981 (code=exited, status=255/EXCEPTION)
        CPU: 160ms

Aug 05 03:41:24 lightsail systemd[1]: Started fail2ban.service - Fail2Ban Service.
Aug 05 03:41:24 lightsail fail2ban-server[1981]: 2024-08-05 03:41:24,878 fail2ban.configreader   [1981]: WARNING 'allowipv6' not defined in 'Definition'. Using default one: 'auto'
Aug 05 03:41:24 lightsail fail2ban-server[1981]: 2024-08-05 03:41:24,912 fail2ban                [1981]: ERROR   Failed during configuration: Have not found any log file for sshd jail
Aug 05 03:41:24 lightsail fail2ban-server[1981]: 2024-08-05 03:41:24,919 fail2ban                [1981]: ERROR   Async configuration of server failed
Aug 05 03:41:24 lightsail systemd[1]: fail2ban.service: Main process exited, code=exited, status=255/EXCEPTION
```

解决办法:
vim /etc/fail2ban/jail.local

```
vim /etc/fail2ban/jail.local
#内容如下
[sshd]
enabled = true
port    = ssh
logpath = /var/log/auth.log
backend = auto
```
创建日志文件/var/log/auth.log
```
touch /var/log/auth.log
```

## 重启并且查看状态
```
systemctl restart fail2ban
systemctl status fail2ban
```
启动成功,控制台输出如下:
```
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-08-08 09:23:46 UTC; 4s ago
       Docs: man:fail2ban(1)
   Main PID: 17051 (fail2ban-server)
      Tasks: 5 (limit: 2303)
     Memory: 14.0M
        CPU: 189ms
     CGroup: /system.slice/fail2ban.service
             └─17051 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

Aug 08 09:23:46 lightsail systemd[1]: Started fail2ban.service - Fail2Ban Service.
Aug 08 09:23:46 lightsail fail2ban-server[17051]: 2024-08-08 09:23:46,319 fail2ban.configreader   [17051]: WARNING 'allowipv6' not defined in 'Definition'. Using default one: 'auto'
Aug 08 09:23:46 lightsail fail2ban-server[17051]: Server ready

```

