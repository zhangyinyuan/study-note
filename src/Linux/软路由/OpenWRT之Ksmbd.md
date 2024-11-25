# OpenWRT之Ksmbd

## 模板

```properties
[global]
	netbios name = |NAME|
	server string = |DESCRIPTION|
	workgroup = |WORKGROUP|
	interfaces = |INTERFACES|
	bind interfaces only = yes
	ipc timeout = 20
	deadtime = 15
	#map to guest = Bad User
	smb2 max read = 64K
	smb2 max write = 64K
	smb2 max trans = 64K
	cache read buffers = no
	cache trans buffers = no
    valid users = root
```

## 添加用户

```shell
ksmbd.adduser -a root
```

## 修改密码

```shell
ksmbd.adduser -p root
```

## 新增非root用户

### 新增用户**smbuser**

```shell
ksmbd.adduser -a smbuser
```

### 设置密码

```shell
ksmbd.adduser -p smbuser
```

### 修改 `/etc/config/ksmbd` 文件，将新用户添加为共享的访问用户

```shell
config share
    option browseable 'yes'
    option create_mask '0666'
    option dir_mask '0777'
    option name 'mi-share'
    option path '/mnt/sda4/mi-share'
    option read_only 'no'
    option guest_ok 'no'
    option users 'smbuser'
```

### 重启 ksmbd 服务

```shell
/etc/init.d/ksmbd restart
```

### 停止 ksmbd 服务

```shell
/etc/init.d/ksmbd stop
```



