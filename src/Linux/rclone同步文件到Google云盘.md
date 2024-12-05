# rclone同步文件到Google云盘

## 安装unzip
rclone安装用的到unzip

```
apt install -y unzip
```

## 安装fuse3
```
apt install -y fuse3
```

## 安装socat (可选)
socat 用于后续配置的时候, 代理127.0.0.1上的端口.也可以用其他代理工具代理

```
apt install -y socat 
```

## 安装rclone
```
curl https://rclone.org/install.sh | sudo bash
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
#如 控制台这样提示的时候, 使用下面命令 2024/05/10 01:38:21 NOTICE: If your browser doesn't open automatically go to the following link: http://127.0.0.1:53682/auth?state=Czs9IR66wt5eLQYQtlogbQ
socat TCP-LISTEN:1234,fork,reuseaddr TCP:127.0.0.1:53682
```

代理成功后, 在桌面端浏览器访问, 访问完成,授权登录后,Linux终端会提示Success,即完成了配置

## 创建Service, 启动挂载

需要特别注意的是: **rclone mount 每次挂载后,总是用云盘的数据覆盖本地的挂载文件夹, 所以云盘上不存在,但是存在于本地挂载文件夹/google_drive_local下的数据一定要先做备份,避免覆盖丢失**

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
## 创建备份脚本
vim /usr/local/bin/vps_backup 内容如下
```
cd / && tar -zcvf app.tar.gz /app/ && mv app.tar.gz /google_drive_local/
```
赋予执行权限```chmod +x /usr/local/bin/vps_backup```

## 编写定时任务
执行```crontab -e```打开定时任务配置文件,
增加以下内容
```shell
#12小时同步一次
0  */12 * * *  /usr/local/bin/vps_backup
```

```shell
#30分钟执行一次
*/30 * * * *  /usr/local/bin/vps_backup
```

