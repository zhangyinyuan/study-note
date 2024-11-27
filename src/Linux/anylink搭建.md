# [docker anylink搭建](https://github.com/stilleshan/anylink:0.9.3)


## 生成web控制台登录密码
> nooqu6aifahsh5Ae为web控制台登录密码
```shell
docker run -it --rm stilleshan/anylink:0.9.3 tool -p nooqu6aifahsh5Ae
```

## 生成 jwt secret
```shell
docker run -it --rm stilleshan/anylink:0.9.3 tool -s
```