# MySQL服务器常用操作

## 创建一个数据库

```sql
CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

## 创建用户并授权

```sql
CREATE USER 'nextcloud_user'@'%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud_user'@'%';
FLUSH PRIVILEGES;
```

## 注意事项

1. **性能影响**
   - `utf8mb4_bin` 因为逐字符比较和区分大小写，性能可能略低于 `utf8mb4_general_ci`。
   - 选择排序规则时需要在性能和功能之间权衡。
2. **文件系统的影响**
   即使数据库支持大小写敏感排序，文件系统（如 `ext4`、`NTFS`）可能不区分大小写，这可能导致文件名冲突。



## 总结

- **默认选择**: `utf8mb4_general_ci`，除非明确需要区分大小写。
- 如果需要大小写敏感，使用 `utf8mb4_bin`。
- 区分大小写的需求较为少见，但在文件名敏感的场景中可能是必要的。
