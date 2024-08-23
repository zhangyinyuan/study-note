# docker常用操作记录

## 容器中执行apt update报错
解决办法
```
sed -i 's/stretch/buster/g' /etc/apt/sources.list
```