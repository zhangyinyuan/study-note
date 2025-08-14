# docker搭建达梦数据库

## Docker 安装

```dockerfile
docker run -d -p 30236:5236 \
--restart=always \
--name dm8 \
--privileged=true \
-e PAGE_SIZE=16 \
-e LD_LIBRARY_PATH=/opt/dmdbms/bin \
-e EXTENT_SIZE=32 \
-e BLANK_PAD_MODE=1 \
-e LOG_SIZE=1024 \
-e UNICODE_FLAG=1 \
-e LENGTH_IN_CHAR=1 \
-e INSTANCE_NAME=dm8 \
-e CASE_SENSITIVE=0 \
-e CHARSET=1 \
tommyyusky/dm8_20230808_rev197096_x86_rh6_64
```

## 默认账户

```yaml
SYSDBA/SYSDBA001
```

## [达梦官网资料下载](https://eco.dameng.com/download/)

### 达梦 JDBC 驱动下载

## 常用命令

## 修改数据库密码

```sql
ALTER USER CURRENT_USER IDENTIFIED BY eizeejee3oh REPLACE SYSDBA001;

# ALTER USER SYSDBA IDENTIFIED BY eizeejee3oh;
```





