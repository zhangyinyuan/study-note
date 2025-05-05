1. Python是解释型语言

2. 除法运算

   1. // 是整除运算符，返回一个整数, `3//6 == 0`
   2. / 是普通除法运算符，返回一个浮点数, `'3/6=0.5'`

3. `**`用来操作幂运算```>>> 5 ** 2 ``` , 得到的是5 的平方的值

4. Python中没有`char` 类型, `char = 'A'  # 单字符本质是字符串`

5. Python区间问题

   1. 大多数情况都是**左闭右开**

      1. `'012'[1:2]`,结果是1

      2. `list(range(1, 4))`,结果是 [1, 2, 3]

      3. ```python
         import random
         random.randrange(1, 4) #返回1到4的一个数字,只返回1个.不包括4
         ```

   2. 个别情况

      1. ```python
         arr = [0,1,2,3]
         arr.pop(1) # 直接删除下标1,得到arr的值为[0,2,3]
         ```

6. 字符串截取

   1. [:3], 表示从左到右向字符串末尾截取, 从下标0开始,截取3位
      1. `"0123"[:3]` 得到的结果是`'012'`
      2. "0123"[:10] 得到的结果是`'0123'`
   2. [2:], 表示从下标2开始,包括下标2, `"0123"[2:]`, 结果是`'23'`
   3. `"0123"[:]`,完整输出`'0123'`
   4. -1表示最后1个字符的下标
      1. `"0123"[-3:-1]`,结果是`'12'`
      2. `"0123"[-1:-3]`,结果是空字符串`''`

7. range场景+语法

   1. 循环遍历固定次数
      ```python
      for i in range(5):  #函数定义后的冒号省略会导致报错
          print(i)  # 输出: 0, 1, 2, 3, 4
      ```

   2. 遍历列表/字符串的索引
      ```python
      s = "hello"
      for i in range(len(s)):
          print(s[i])  # 输出: h, e, l, l, o
      ```

   3. 生成等差数列
      ```python
      # 生成 1~10 的偶数
      even_numbers = list(range(2, 11, 2))  # [2, 4, 6, 8, 10]
      ```

   4. 反向遍历
      ```python
      for i in range(5, 0, -1):
          print(i)  # 输出: 5, 4, 3, 2, 1
      ```

   5. 快速初始化列表
      ```python
      zeros = [0] * 10          # [0, 0, ..., 0]
      indices = list(range(10)) # [0, 1, 2, ..., 9]
      ```

   6.  数学计算中的序列生成
      ```python
      [x**2 for x in range(1, 6)] # [1, 4, 9, 16, 25]
      ```

8. **冒号 `:` 是 Python 语法的一部分**，它指示接下来的一段代码属于某个控制结构的代码块（如 `for`、`if`、`while`,以及函数定义末尾 等）,省略之后程序会抛出 SyntaxError 错误

   ```python
   # 片段1
   for i in range(5):
       print(i)
       
   # 片段2
   def add(a, b):
       return a + b
   ```

9. 没有 `switch` 语句

10. `else` 可以与 `for`、`while` 和 `try` 语句结合使用。对于 `try` 语句，`finally` 总会执行（无论是否发生异常）。

   ```python
   try:
       x = 10 / 0
   except ZeroDivisionError:
       print("Can't divide by zero")
   else:
       print("No exception occurred")
   finally:
       print("This will always execute")
   ```

11. Python 支持多重继承

12. 不强制类型转换: Python 动态类型语言，无需显式类型转换 / Java 是静态类型语言，需要显式的类型转换

13. 元组不可修改: 用圆括号 `()` 来定义，创建后不能更改其中的元素/ Java可以修改成功
    ```python
    my_tuple = (1, 2, 3)
    # 尝试修改元素会引发错误
    # my_tuple[0] = 10  # 会引发 TypeError
    ```

14. 

