# Linux代理搭建和使用
```
sudo yum install squid
sudo systemctl start squid
sudo systemctl enable squid
```

- 查看配置文件
```
cat /etc/squid/squid.conf
```
默认配置如下
```
http_port 8087
cache_mem 128 MB
cache_dir ufs /var/spool/squid 4096 16 256
cache_effective_user squid  #设置用户
cache_effective_group squid  #设置用户组
access_log /var/log/squid/access.log   #设置访问日志文件
cache_log /var/log/squid/cache.log  #设置缓存日志文件
cache_store_log /var/log/squid/store.log  #设置缓存记录文件
visible_hostname cdn.abc.com  #设置squid服务器主机名
cache_mgr root@root.com
acl all src 0.0.0.0/0.0.0.0  #设置访问控制列表，默认开启
http_access allow all
acl client dstdomain -i www.abc.com    #找到TAG: acl标签，在其最后添加下面内容
http_access deny client  #禁止所有客户机访问www.abc.com域名
acl client131 src 192.168.237.131  #禁止IP地址为192.168.237.131的客户机访问外网
http_access deny client131
acl client129 dst 192.168.237.129  #禁止所有用户访问IP地址为192.168.237.129的网站
http_access deny client129
acl client163 url_regex -i 163.com  #禁止所有用户访问域名中包含有163.com的网站
http_access deny client163
acl clientdate src 192.168.237.0/255.255.255.0  #禁止这个网段所有的客户机在周一到周五的18:00-21:00上网
acl worktime time MTWHF 18:00-21:00
http_access deny clientdate worktime
#acl clientxiazai urlpath_regex -i \.mp3$  \.exe$  \.zip$  \.rar$
#http_access deny clientxiazai  #禁止客户机下载*.mp3、*.exe、*.zip和*.rar类型的文件
```

- 在需要代理的机器上执行
```
export http_proxy=http://instance1_ip:8087
export https_proxy=http://instance1_ip:8087
```