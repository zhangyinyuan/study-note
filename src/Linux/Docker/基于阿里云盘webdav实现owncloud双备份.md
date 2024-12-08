# 基于阿里云盘webdav实现owncloud双备份
```shell
version: '3.8'
services:
  aliyundrive-webdav:
    image: messense/aliyundrive-webdav
    container_name: aliyundrive-webdav
    restart: always
    # ports:
    #   - "28080:8080"
    environment:
      - REFRESH_TOKEN=${REFRESH_TOKEN} # REFRESH_TOKEN写在.env文件中，防止太长的字符串在docker-compose中被截断，注意这里实际上是对应access_token
      - WEBDAV_AUTH_USER=这里是webdav的网页登录用户名，自定义
      - WEBDAV_AUTH_PASSWORD=这里是webdav的网页登录密码，自定义
    env_file:
      - .env 
```
nano .env
```shell
REFRESH_TOKEN=xxxx改成自己的token
```

- 参考 ：https://github.com/messense/aliyundrive-webdav?tab=readme-ov-file

- [通过在线工具获取 refresh token](https://messense-aliyundrive-webdav-backendrefresh-token-ucs0wn.streamlit.app/)

## 启动aliyundrive-webdav
```shell
docker-compose up -d 
```

## 通过rclone webdav配置
```shell
rclone config
# 按照提示配置。选webdav。密码和密码就是上面配置的
```

## 挂载
> 假如上面配置的阿里云盘名叫aliyun_drive
```shell
nohup rclone mount aliyun_drive:/ /aliyun_drive_local --vfs-cache-mode writes >> /tmp/aliyun_drive_local.log 2>&1 &
```

## 安装inotify，用于监听owncloud文件变化
```shell
```
## 配置脚本sync_with_inotify.sh
nano /app/sync_with_inotify.sh
```shell
#!/bin/bash

LOCAL_DIR="/app/docker/owncloud/owncloud/data/files/admin/files"
LOG_FILE="/tmp/rclone_owncloud.log"

## 使用 inotifywait 监听本地文件夹变化
inotifywait -m -r -e create,modify,delete,move "$LOCAL_DIR" |
while read -r directory events filename; do
  echo "Detected $events on $filename"

  # 延迟执行，避免频繁触发
  sleep 10

  # 防止重复同步
  if pgrep -f "rclone sync" > /dev/null; then
    echo "Sync is already running, skipping this event."
    continue
  fi

  rclone sync "$LOCAL_DIR" aliyun_drive:/owncloud_backup \
    --exclude "*.d*" \
    --transfers 2 \
    --checkers 2 \
    --delete-during \
    --log-level DEBUG \
    --log-file "$LOG_FILE"
done
```

## 添加执行权限
```shell
chmod +x /app/sync_with_inotify.sh
```

## 作为服务启动
```shell
sudo nano /etc/systemd/system/sync_with_inotify.service
[Unit]
Description=Sync files with Aliyun Drive using rclone and inotify
After=network.target

[Service]
ExecStart=/bin/bash /app/sync_with_inotify.sh
Restart=always
RestartSec=10s
User=root
WorkingDirectory=/app
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

## 重新加载 systemd 配置
```shell
sudo systemctl daemon-reload
```

## 启动并使服务开机启动
```shell
sudo systemctl start sync_with_inotify.service
sudo systemctl enable sync_with_inotify.service
sudo systemctl status sync_with_inotify.service
sudo systemctl restart sync_with_inotify.service
sudo systemctl stop sync_with_inotify.service
```

## 查看日志
```shell
journalctl -u sync_with_inotify.service -f
```
## 手动同步
> 多次同步不影响
```shell
rclone sync "/app/docker/owncloud-ok/files/files/" aliyun_drive:/owncloud_backup   --exclude "*.d*"   --transfers 2   --checkers 2   --delete-during   --log-level DEBUG   --log-file /app/rclone_owncloud.log
```

## 查看日志文件
```shell
tail -f /tmp/rclone_owncloud.log
```

## 需要将webdav的IP固定
> 避免重启服务后,IP变化....最终导致rclone中的webdav地址发生变化,无法连接到webdav

## 不想挂载时，卸载已挂载的文件夹
```shell
fusermount -uz /aliyun_drive_local
```