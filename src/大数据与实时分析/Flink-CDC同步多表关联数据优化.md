# Flink-CDC同步多表关联数据优化

> 场景9张表关联, 主表数据3万不到.其他表多次关联, 其他关联表数据很少. 但是9次关联之后,一次完整同步需要耗时30分钟

## 环境

### StarRocks版本

```sql
SHOW VARIABLES LIKE '%version%';
```

| Variable_name   | Value                    | 说明                                                         |
| --------------- | ------------------------ | ------------------------------------------------------------ |
| version         | 5.1.0                    | 表示当前 StarRocks 的 **SQL 语法兼容性** 基于 **MySQL 5.1.0** 协议 |
| version_comment | StarRocks version 2.5.10 | StarRocks **实际的数据库版本号**                             |

### flink版本

```bash
flink --version
```

Version: **1.18.1**

### dinky版本

**1.1.0**

### MySQL版本

```sql
|Variable_name|Value|
|-------------|-----|
|innodb_version|5.7.24|
|protocol_version|10|
|slave_type_conversions||
|version|10.2.21-MariaDB-log|
|version_comment|MariaDB Server|
|version_compile_machine|x86_64|
|version_compile_os|Linux|
|version_malloc_library|system|
|version_ssl_library|YaSSL 2.4.4|
|wsrep_patch_version|wsrep_25.23|
```



### docker-compose部署调试环境

```yaml
```











