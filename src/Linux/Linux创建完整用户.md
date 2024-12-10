# Linux创建完整用户
## 创建用户
- 创建用户
```
sudo useradd -m -s /bin/bash myuser
```

- 设置用户密码
```
sudo passwd myuser
```
---

## 给用户赋予sudo权限
- 添加用户到 sudo 组
```
sudo usermod -aG wheel myuser
```

## wheel组的用 配置无密码 sudo 权限
> 注意这是放行所有wheel组的用户, 无密码执行sudo
```
sudo visudo
```

找到这一行 `%wheel  ALL=(ALL)       NOPASSWD: ALL`
将前面的#去掉. 

## 指定用户,配置无密码执行sudo权限
新增一行
```
myuser ALL=(ALL) NOPASSWD: /bin/chmod
```

## 其他辅助性命令
- 查看所有用户
```
cat /etc/passwd
```

- 查看用户组
```
cat /etc/group
```

- 查看用户的权限和组
```
id myuser
```
会输出类似于以下的信息
```
uid=1001(myuser) gid=1001(myuser) groups=1001(myuser),10(wheel)
```

- 查看所有组
```
getent passwd
```

- 使用 groups 命令查看当前用户的组
```
groups
```

- 使用 getent 命令检查特定组
```
getent group docker
``` 

- 将用户添加到 docker 组
```
sudo usermod -aG docker myuser
```
添加后, 执行sudo docker 命令可能提示无权限. 解决办法有两种:
- 完全退出并重新登录
- newgrp docker






