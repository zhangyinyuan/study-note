# journalctl查看ssh连接日志

## 使用 journalctl 查看认证日志
```
sudo journalctl -u ssh
```

## 持续监控这些日志，可以使用 -f 选项，类似于 tail -f
```
sudo journalctl -u ssh -f
```

## 查看所有与认证相关的日志记录
```
sudo journalctl | grep ssh
```

## 查看登录成功的记录
```
sudo journalctl | grep ssh | grep 'Accepted'
```

## 查看被拒绝的记录
```
sudo journalctl | grep ssh | grep preauth
```


## 日志分析
```
Oct 16 06:10:01 amd2 sshd[7597]: Connection reset by invalid user admin 154.47.27.80 port 42672 [preauth]
```
对主机名为amd2的VPS进行ssh连接,invalid user 表示无效用户, 即使用不存在的用户admin进行ssh连接. preauth被拒绝
即 用户输入的用户名还未通过认证程序校验，系统已经发现这是一个无效用户并且连接已被重置


