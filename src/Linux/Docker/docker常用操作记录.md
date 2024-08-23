# docker常用操作记录

## 容器中执行apt update报错
解决办法
```
sed -i 's/stretch/buster/g' /etc/apt/sources.list
```

## 查看指定容器的近10条日志
```
docker logs --tail 10 -f mariadb1
```