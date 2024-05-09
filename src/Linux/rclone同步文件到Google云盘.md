# rclone同步文件到Google云盘

## 安装rclone
```
curl https://rclone.org/install.sh | sudo bash
```

## 安装fuse3
```
apt install -y fuse3
```

## 安装socat (可选)
> socat 用于后续配置的时候, 代理127.0.0.1上的端口.也可以用其他代理工具代理
```
apt install -y socat 
```

## 创建本地文件夹,用于同步远程
```
mkdir -p /google_drive_local
```

## 配置
```
rclone config
```
注意是 google drive，不是 google cloud 或 google photos


## socat代理端口
配置的过程中, 会等待回调http:127.0.0.1:xxxx这样的端口
由于Linux上没有安装桌面端,需要代理出来,在桌面端浏览器访问

如下,将服务器上的127.0.0.1的xxxx端口映射到Linux公网IP上的1234端口
```
socat TCP-LISTEN:xxxx,fork,reuseaddr TCP:Linux本机外网IP:1234
```
本机外网IP可以通过 ```curl ipinfo.io```查询到

代理成功后, 在桌面端浏览器访问, 访问完成,授权登录后,Linux终端会提示Success,即完成了配置

## 创建Service, 启动挂载
vim /etc/systemd/system/rclone-mount.service 文件内容如下
```
[Unit]
Description=RClone Mount Service
After=network-online.target

[Service]
User=root
Group=root
ExecStart=/usr/bin/rclone mount google_drive:/vps_backup/xjp /google_drive_local --allow-other --allow-non-empty --vfs-cache-mode writes
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

## 重新加载 systemd 并启动服务
```
sudo systemctl daemon-reload
sudo systemctl start rclone-mount.service
sudo systemctl enable rclone-mount.service
```

