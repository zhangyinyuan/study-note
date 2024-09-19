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

- map简化代码

```java
//简化前
        Map<Integer, PtBloodIdx> bloodIdxMap = listByIds(bloodIds).stream().collect(Collectors.toMap(PtBloodIdx::getId, Function.identity(), (k1, k2) -> k1));
        for (PtBloodTree ptBloodTree : trees) {
            Integer bloodId = ptBloodTree.getBloodId();
            PtBloodIdx ptBloodIdx = bloodIdxMap.get(bloodId);
            if (ptBloodIdx == null) {
                continue;
            }
            ChildBloodTreeResVo res = new ChildBloodTreeResVo();
            res.setId(ptBloodIdx.getId());
            res.setBloodCode(ptBloodIdx.getBloodCode());
            res.setBloodName(ptBloodIdx.getBloodName());
            resList.add(res);
        }
        retrun resList;

//简化后
        return trees.stream()
                .map(tree -> bloodIdxMap.get(tree.getBloodId()))
                .filter(Objects::nonNull)
                .map(ptBloodIdx -> {
                    Integer bloodId = .getId();
                    List<BloodNodeResVo> parentBloods = getParentBloods(bloodId);
                    return ChildBloodTreeResVo.builder().id(bloodId).bloodCode(ptBloodIdx.getBloodCode()).bloodName(ptBloodIdx.getBloodName()).parentBloods(parentBloods).build();
                })
                .collect(Collectors.toList());
```

解释:

- 通过第一个.map直接从tree转换成了PtBloodIdx
- filter过滤后,只保留每一个不为null的对象
- 第二个map开始组装最终的结果里的每一个对象



