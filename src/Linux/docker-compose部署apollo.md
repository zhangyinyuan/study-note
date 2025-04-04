# docker-compose部署apollo 2.2.0

- 创建文件夹
```bash
mkdir -p /app/docker/apollo
```

- 最终目录结构如下:
```bash
tree
```

`
.
├── config
│   └── apollo-env.properties
├── docker-compose.yml
└── sql
    ├── 01-apolloconfigdb.sql
    ├── 02-apolloconfigdb.sql
    ├── 03-init-user.sql
    └── 04-init-eureka-url.sql

3 directories, 6 files
`