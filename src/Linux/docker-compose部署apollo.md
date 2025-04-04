# docker-compose部署apollo 2.2.0

- 创建文件夹
```bash
mkdir -p /app/docker/apollo
```

- 最终目录结构如下:
```bash
tree
```

```bash
.
├── docker-compose.yml
└── sql
    ├── 01-apolloconfigdb.sql
    ├── 02-apolloconfigdb.sql
    ├── 03-init-user.sql
    └── 04-init-eureka-url.sql

2 directories, 5 files
```

- 01-apolloconfigdb.sql 文件内容
```
services:
  apollo-db:
    image: mariadb:10.11
    container_name: apollo-db
    environment:
      MYSQL_ROOT_PASSWORD: xoopooth4gie6doo5Aechieteev9genu
      MYSQL_USER: apollo
      MYSQL_PASSWORD: apollo123
    volumes:
      - ./sql:/docker-entrypoint-initdb.d
      - db_data:/var/lib/mysql
    command:
      - --default-authentication-plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-uapollo", "-papollo123", "--silent"]
      interval: 5s
      timeout: 10s
      retries: 10

  apollo-configservice:
    image: apolloconfig/apollo-configservice:2.2.0
    container_name: apollo-configservice
    depends_on:
      apollo-db:
        condition: service_healthy
    environment:
      SPRING_DATASOURCE_DRIVER_CLASS_NAME: com.mysql.cj.jdbc.Driver
      SPRING_DATASOURCE_URL: jdbc:mysql://apollo-db:3306/ApolloConfigDB?useSSL=false&characterEncoding=utf8
      SPRING_DATASOURCE_USERNAME: apollo
      SPRING_DATASOURCE_PASSWORD: apollo123
      EUREKA_INSTANCE_HOSTNAME: apollo-configservice
      EUREKA_INSTANCE_IP_ADDRESS: apollo-configservice
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://apollo-configservice:8080/eureka/
      EUREKA_SERVER_ENABLE_SELF_PRESERVATION: "false"
      EUREKA_SERVER_EVICTION_INTERVAL_TIMER_IN_MS: "5000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  apollo-adminservice:
    image: apolloconfig/apollo-adminservice:2.2.0
    container_name: apollo-adminservice
    depends_on:
      apollo-configservice:
        condition: service_healthy
    environment:
      SPRING_DATASOURCE_DRIVER_CLASS_NAME: com.mysql.cj.jdbc.Driver
      SPRING_DATASOURCE_URL: jdbc:mysql://apollo-db:3306/ApolloConfigDB?useSSL=false&characterEncoding=utf8
      SPRING_DATASOURCE_USERNAME: apollo
      SPRING_DATASOURCE_PASSWORD: apollo123
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://apollo-configservice:8080/eureka/
      EUREKA_INSTANCE_HOSTNAME: apollo-adminservice
      EUREKA_INSTANCE_IP_ADDRESS: apollo-adminservice
      EUREKA_CLIENT_REGISTER_WITH_EUREKA: "true"
      EUREKA_CLIENT_FETCH_REGISTRY: "true"
      # 显式覆盖所有可能的配置键（不同大小写和格式）
      EUREKA_CLIENT_SERVICEURL_DEFAULTZONE: http://apollo-configservice:8080/eureka/
      SPRING_CLOUD_CONFIG_ENABLED: "false"  # 禁用配置服务器覆盖
      SPRING_APPLICATION_JSON: '{"eureka":{"client":{"serviceUrl":{"defaultZone":"http://apollo-configservice:8080/eureka/"}}}}'  # 最高优先级
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8090/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  apollo-portal:
    image: apolloconfig/apollo-portal:2.2.0
    container_name: apollo-portal
    depends_on:
      apollo-configservice:
        condition: service_healthy
    environment:
      SPRING_DATASOURCE_DRIVER_CLASS_NAME: com.mysql.cj.jdbc.Driver
      SPRING_DATASOURCE_URL: jdbc:mysql://apollo-db:3306/ApolloPortalDB?useSSL=false&characterEncoding=utf8
      SPRING_DATASOURCE_USERNAME: apollo
      SPRING_DATASOURCE_PASSWORD: apollo123
      APOLLO_PORTAL_ENVS: dev
      DEV_META: http://apollo-configservice:8080
      PRO_META: http://apollo-configservice:8080
      APOLLO_CONFIG_SERVICE: http://apollo-configservice:8080
    ports:
      - "8070:8070"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8070/health"]
      interval: 30s
      timeout: 10s
      retries: 5
    networks:
      - nginx_ngu_network  #此处是为了加入已经创建好的nginx网络中.用于nginx代理.可选
      - default

networks:
  nginx_ngu_network:
    external: true

volumes:
  db_data:

```

- sql/01-apolloconfigdb.sql
> 下载文件 https://github.com/apolloconfig/apollo/blob/master/scripts/sql/profiles/mysql-default/apolloconfigdb.sql

- sql/02-apolloconfigdb.sql
> 下载文件 https://github.com/apolloconfig/apollo/blob/master/scripts/sql/profiles/mysql-default/apolloportaldb.sql

- sql/03-init-user.sql
```
CREATE USER IF NOT EXISTS 'apollo'@'%' IDENTIFIED BY 'apollo123';
GRANT ALL PRIVILEGES ON ApolloConfigDB.* TO 'apollo'@'%';
GRANT ALL PRIVILEGES ON ApolloPortalDB.* TO 'apollo'@'%';
FLUSH PRIVILEGES
```

- sql/04-init-eureka-url.sql
> 非常重要. apollo 2.2.0版本. apollo-adminservice默认的注册中心是http://localhost:8080/eureka/,启动后总是无法找到正常的apollo-configservice.用执行初始化sql的办法修改
```sql
UPDATE `ApolloConfigDB`.`ServerConfig`
SET `Value` = 'http://apollo-configservice:8080/eureka/'
WHERE `Key` = 'eureka.service.url';
```

- 启动
```
docekr-compose up -d
```

- 访问 http://localhost:8070