# Java 8 Stream API 之 removeIf
---
## removeIf简化代码
-  移除所有负数
```
List<Integer> numbers = Arrays.asList(1, -2, 3, -4, 5);
numbers.removeIf(n -> n < 0);
// 结果: [1, 3, 5]
```

---
- 移除过期的订单
```
List<Order> orders = getOrders(); // 获取订单列表
orders.removeIf(order -> order.getExpiryDate().isBefore(LocalDate.now()));
// 移除所有过期的订单
```

---
- 移除名称重复的用户
```
List<UserDTO> users = getUsers();
if (CollectionUtils.isNotEmpty(users)) {
    Set<String> uniqueNames = new HashSet<>();
    users.removeIf(u -> !uniqueNames.add(u.getName()));
}
return users;
```