# 上午题 # 8 UML

2025年3月29日12:43:34

## UML四种事务

| **类别**     | **核心特点**           | **典型例子**             | **类比记忆**             |
| :----------- | :--------------------- | :----------------------- | :----------------------- |
| **结构事物** | **静态的、基础构建块** | **类、接口、构件、节点** | 公司的「部门架构图」     |
| **行为事物** | **动态的、动作与流程** | **交互、状态机、活动**   | 员工的「工作流程表」     |
| **分组事物** | **组织管理的容器**     | 包（Package）            | 文件柜的「**分类抽屉**」 |
| **注释事物** | **附加说明的符号**     | 注释（Note）             | 贴在墙上的「**便利贴**」 |

![image-20250329124522229](../../../images/image-20250329124522229.png)

---

## UML五种关系

> **依赖借，关联友，聚合可拆组合死**

| **关系类型**                      | **UML表示**       | **代码表现**            | **生命周期** | **强度**                      |
| :-------------------------------- | :---------------- | :---------------------- | :----------- | :---------------------------- |
| **依赖**                          | `----->`（虚线）  | 局部变量/<br />方法参数 | 临时性       | ⭐                             |
| **关联<br />Association**         | ──>（实线）       | 成员属性                | 独立存在     | ⭐⭐                            |
| **聚合<br />Aggregation**         | `◇──`（空心菱）   | 成员属性                | 部分可独立   | ⭐⭐⭐<br />强调对象间的引用关系 |
| **组合**<br />Composition         | `◆──`（实心菱）   | 成员属性                | 同生共死     | ⭐⭐⭐⭐<br />人体与心脏          |
| **泛化/继承**<br />Generalization | `▲──`（空心三角） | `extends`               | 子类继承父类 | 特殊                          |
| 实现<br />Realization             | 虚线空心三角      | 类实现接口              | 子类实现父类 |                               |

![image-20250329131757929](../../../images/image-20250329131757929.png)

---

![image-20250329150511146](../../../images/image-20250329150511146.png)

---

![image-20250329153423510](../../../images/image-20250329153423510.png)

---

![image-20250329154643874](../../../images/image-20250329154643874.png)

---

![image-20250329154812796](../../../images/image-20250329154812796.png)

---

## 关联多重度

![image-20250329155321257](../../../images/image-20250329155321257.png)

---

## UML类图

![image-20250329160132623](../../../images/image-20250329160132623.png)

---

![image-20250329160512739](../../../images/image-20250329160512739.png)

---

![image-20250329160944296](../../../images/image-20250329160944296.png)

---

![image-20250329161614441](../../../images/image-20250329161614441.png)

---

## 对象图

![image-20250329173036085](../../../images/image-20250329173036085.png)

---

## 用例图

![image-20250329173644395](../../../images/image-20250329173644395.png)

---

