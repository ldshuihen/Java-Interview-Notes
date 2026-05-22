# Python核心编程

---

## 一、Python基础语法（入门核心）

这是所有编程的根基，目标是**能写简单代码、理解程序执行逻辑**。
### 核心知识点
1. 环境搭建：Python安装、pip包管理、IDE（PyCharm/VSCode）使用
2. 基础语法：注释、缩进规则、变量命名、输入输出（`input()`/`print()`）
3. 数据类型：数字（int/float）、字符串（str）、布尔值（bool）、空值（None）
4. 运算符：算术、比较、逻辑、赋值、成员运算符
5. 流程控制：`if-elif-else` 条件判断、`for/while` 循环、`break/continue`
6. 异常基础：`try-except` 捕获简单错误

### 应用场景

编写小工具、自动化脚本、算法入门、数据处理小逻辑

### 1. 环境搭建

#### 1.1 Python安装

- 官网下载对应系统版本，**勾选Add Python to PATH**自动配置环境变量
- 验证： cmd输入 `python --version` / `py -V` 显示版本即成功

#### 1.2 pip包管理

- 查看版本：`pip --version`
- 安装库：`pip install 库名`
- 卸载库：`pip uninstall 库名`
- 换源加速（国内镜像）：
```bash
pip install 库名 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 1.3 IDE使用

- **VSCode**：轻量简洁，适合入门刷题、写小脚本
- **PyCharm**：专业Python编辑器，自带项目管理、代码提示，适合正式开发

---

### 2. 基础语法规则

#### 2.1 注释（代码说明，不执行）

```python
# 单行注释
"""
多行注释
多行注释
"""
'''
单引号多行注释
'''
```

#### 2.2 缩进（Python核心规则，代替大括号）

- 必须统一：**4个空格**，禁止空格tab混用
- 缩进一致为同一代码块
```python
if 1>0:
    print("成立")  # 缩进内代码
print("外部代码")  # 无缩进
```

#### 2.3 变量命名规则

1. 由**字母、数字、下划线**组成，不能数字开头
2. 区分大小写 `name` 和 `Name` 不同
3. 不能使用Python关键字（if/for/while/def等）
4. 推荐规范：
   - 普通变量：小写下划线 `user_name`
   - 常量：全大写 `MAX_NUM`

#### 2.4 输入与输出

```python
# 输出
print("Hello Python")
print(123, 456)  # 多内容空格分隔

# 输入：默认接收字符串类型
name = input("请输入姓名：")
age = int(input("请输入年龄："))  # 强制转为数字
```

---

### 3. 五大基础数据类型

| 类型       | 写法              | 作用            |
| ---------- | ----------------- | --------------- |
| 整型 int   | `10` `-5`         | 整数            |
| 浮点 float | `3.14` `0.5`      | 小数            |
| 字符串 str | `"文本"` `'内容'` | 文字、字符      |
| 布尔 bool  | `True`/`False`    | 真/假，判断使用 |
| 空值 None  | `None`            | 空、无数据      |

#### 类型转换

```python
int("123")    # 字符串转整数
str(666)      # 数字转字符串
float(10)     # 转小数
bool(0)       # 0为空假，非0为真
```

---

### 4. 常用运算符

#### 4.1 算术运算符

`+ - * / % ** //`
```python
print(5+2)   # 加
print(5-2)   # 减
print(5*2)   # 乘
print(5/2)   # 除 得小数
print(5%2)   # 取余
print(5**2)  # 幂次方 5平方
print(5//2)  # 整除 取整数部分
```

#### 4.2 比较运算符

结果只有 **True / False**
`> < >= <= == !=`
```python
3>2    # True
3==3   # 等于
3!=4   # 不等于
```

#### 4.3 逻辑运算符

- `and` 并且：两边都真才为真
- `or` 或者：一边真就为真
- `not` 取反

#### 4.4 成员运算符

`in` / `not in` 判断元素是否在序列中
```python
"a" in "abc"   # True
```

#### 4.5 赋值运算符

`= += -= *= /=`
```python
a = 1
a += 1  # 等价 a = a+1
```

---

### 5. 流程控制（程序执行顺序）

#### 5.1 条件判断 if 语句

1. 单分支
```python
if 条件:
    满足执行代码
```
2. 双分支
```python
if 条件:
    成立执行
else:
    不成立执行
```
3. 多分支 `if-elif-else`
```python
score = 80
if score >=90:
    print("优秀")
elif score >=80:
    print("良好")
elif score >=60:
    print("及格")
else:
    print("不及格")
```

#### 5.2 循环结构

##### while 循环（条件循环）

```python
i = 1
while i <=5:
    print(i)
    i +=1
```

##### for 循环（遍历循环，最常用）

```python
# 遍历数字
for i in range(1,6):
    print(i)

# 遍历字符串
for s in "python":
    print(s)
```

#### 5.3 循环控制关键字

- `break`：**直接终止整个循环**
- `continue`：**跳过本次，进入下一次循环**

---

### 6. 异常基础 try-except

作用：防止代码报错崩溃，捕获错误并友好提示
```python
try:
    # 可能出错代码
    num = int(input("输入数字："))
except:
    # 出错后执行
    print("输入格式错误！")
```
精准捕获异常：
```python
try:
    a = 1/0
except ZeroDivisionError:
    print("不能除以0")
```

---

### 7. 实际应用场景

1. 日常小工具：计算器、猜数字游戏、密码判断
2. 简单自动化：批量重命名文件、批量整理文本
3. 算法入门：冒泡排序、二分查找、逻辑刷题
4. 数据简易处理：筛选文本数据、统计数量
5. 校园/办公小脚本：成绩统计、信息录入判断

需要我给你整理**基础语法必练50道练习题**吗？

---

## 二、Python数据结构（存储与操作数据）

Python的特色就是**内置强大数据结构**，是处理数据的核心工具。
### 核心知识点
1. 序列类型：列表（`list`）、元组（`tuple`）—— 有序可迭代
2. 映射类型：字典（`dict`）—— 键值对存储，查询最快
3. 集合类型：集合（`set`）—— 去重、交集/并集/差集运算
4. 高级操作：切片、推导式（列表/字典/集合推导式）、迭代器、生成器
5. 内置函数：`len()`/`max()`/`min()`/`sorted()`/`enumerate()`/`zip()`

### 应用场景
数据清洗、批量处理、日志解析、算法实现

### 整体分类

1. **序列类型**：有序、可下标取值 → `list`列表、`tuple`元组、字符串
2. **映射类型**：键值对、无序（3.7+有序）→ `dict`字典
3. **集合类型**：无序、元素唯一 → `set`集合

---

### 1. 列表 list []

#### 1.1 定义与特点

- 写法：`lst = [1,2,"python",True]`
- 特点：**有序、可变、可重复、支持任意类型**
- 用途：存放一组可变数据，最常用容器

#### 1.2 常用增删改查

```python
# 创建
lst = [10,20,30]

# 查
lst[0]      # 下标取值
lst[-1]     # 倒数第一个
lst.index(20) # 获取元素下标
10 in lst   # 判断是否存在

# 增
lst.append(40)      # 末尾追加
lst.insert(1,99)     # 指定位置插入
lst.extend([50,60]) # 合并列表

# 删
lst.pop()           # 默认删末尾，可按下标删
lst.remove(20)      # 按元素删除
lst.clear()         # 清空列表
del lst[0]          # 删除指定下标

# 改
lst[0] = 100
```

#### 1.3 列表排序

```python
lst.sort()           # 升序
lst.sort(reverse=True) # 降序
new_lst = sorted(lst) # 生成新列表排序
```

---

### 2. 元组 tuple ()

#### 2.1 定义与特点

- 写法：`t = (1,2,3)`
- 单个元素必须加逗号：`t = (5,)`
- 特点：**有序、不可变、只读、速度快**
- 用途：固定数据、函数多返回值、保护数据不被修改

#### 2.2 常用操作

```python
t = (11,22,33)
print(t[0])        # 只能查，不能增删改
len(t)
t.count(22)
t.index(33)
```
**元组不可变：内部列表可改**
```python
t = (1,[2,3])
t[1][0] = 99  # 合法
```

---

### 3. 字典 dict {}

#### 3.1 定义与特点

- 写法：`dic = {"name":"张三","age":18}`
- 结构：**键(key):值(value)**
- 特点：3.7版本后有序、**键唯一不可重复**、查询速度极快
- 键只能是不可变类型：字符串、数字、元组

#### 3.2 增删改查

```python
dic = {"name":"李四","sex":"男"}

# 查
dic["name"]
dic.get("age",18) # 无键不报错，设置默认值

# 增/改
dic["age"] = 20    # 有则改，无则增

# 删
dic.pop("sex")
dic.popitem()      # 删除最后一组
dic.clear()

# 遍历取值
dic.keys()    # 所有键
dic.values()  # 所有值
dic.items()   # 键值对
```

#### 3.3 字典快速初始化

```python
dict(zip(["a","b"],[1,2]))
```

---

### 4. 集合 set {}

#### 4.1 定义与特点

- 写法：`s = {1,2,3}` 空集合 `s = set()`
- 特点：**无序、元素自动去重、不可下标访问**
- 核心作用：**去重、关系运算**

#### 4.2 基础操作

```python
s = {1,2,3}
s.add(4)        # 添加元素
s.remove(2)     # 删除
s.discard(99)   # 不存在也不报错
```

#### 4.3 集合关系运算（高频）

```python
a = {1,2,3,4}
b = {3,4,5,6}

a & b   # 交集 {3,4}
a | b   # 并集 {1,2,3,4,5,6}
a - b   # 差集 a独有
a ^ b   # 对称差集 互相独有
```

#### 4.4 列表一键去重

```python
lst = [1,1,2,2,3]
new_lst = list(set(lst))
```

---

### 5. 切片操作（序列通用）

适用：**列表、元组、字符串**
语法：`[起始:结束:步长]`
- 左闭右开：取到**结束前一位**
- 默认从0开始，默认步长1
```python
s = [1,2,3,4,5]
s[1:4]    # [2,3,4]
s[:3]     # 前3个
s[2:]     # 从第3个到末尾
s[::-1]   # 反转
```

---

### 6. 三大推导式（极简写法）

#### 6.1 列表推导式

```python
# 生成1-10偶数
lst = [i for i in range(1,11) if i%2==0]
```
#### 6.2 字典推导式

```python
dic = {k:k*2 for k in range(1,6)}
```
#### 6.3 集合推导式

```python
s = {i**2 for i in range(5)}
```

---

### 7. 迭代器 & 生成器

#### 7.1 迭代器 Iterator

- 可被`for`遍历的对象，具备`__iter__`、`__next__`
- `iter()`转迭代器，`next()`逐个取值

#### 7.2 生成器 Generator

- 节省内存，边用边生成，不一次性存全部数据
- 写法1：`(i for i in range(10))`
- 写法2：函数里用 `yield`

```python
def gen():
    yield 1
    yield 2
```

---

### 8. 高频内置数据函数

```python
len()       # 获取长度
max()/min() # 最大最小值
sorted()    # 通用排序
enumerate() # 带下标遍历
zip()       # 多序列配对打包
```
#### 示例

```python
# 带下标遍历
for idx,val in enumerate([10,20,30]):
    print(idx,val)

# 两列表配对
list(zip([1,2],["a","b"]))
```

---

### 9. 四大数据结构选型对照表

| 结构  | 有序     | 可变 | 特点         | 适用场景             |
| ----- | -------- | ---- | ------------ | -------------------- |
| list  | 是       | 是   | 灵活通用     | 增删频繁、有序数据   |
| tuple | 是       | 否   | 安全快速     | 固定数据、函数返回值 |
| dict  | 是(3.7+) | 是   | 键值超快查询 | 字段信息、映射关系   |
| set   | 否       | 是   | 自动去重     | 数据去重、比对交集   |

---

### 10. 实际应用场景

1. **数据清洗**：列表筛选、集合去重、字典整理字段
2. **批量处理**：推导式快速生成批量数据
3. **日志解析**：字典存储日志字段，列表存放多条日志
4. **算法刷题**：双列表、哈希字典、集合判重
5. **接口数据处理**：字典解析JSON接口数据
6. **文本统计**：字典统计词频

需要我给你出**数据结构专项练习题+答案**吗？

---

## 三、函数式编程

解决**代码重复**问题，把逻辑封装成可调用的函数，是模块化编程的基础。
### 核心知识点
1. 函数定义与调用：`def` 关键字、参数、返回值（`return`）
2. 参数类型：位置参数、关键字参数、默认参数、可变参数（`*args`/`**kwargs`）
3. 作用域：全局变量、局部变量、`nonlocal`/`global` 关键字
4. 高阶函数：`map()`/`filter()`/`reduce()`、匿名函数（`lambda`）
5. 装饰器：给函数动态添加功能（Python核心语法糖）
6. 闭包：函数嵌套+变量引用

### 应用场景
封装业务逻辑、通用工具函数、代码解耦

### 核心作用

封装重复代码，一次定义多处调用，实现**代码复用、逻辑解耦、简化维护**，是模块化开发必备基础。

---

### 1. 函数定义与调用

#### 1.1 基础语法

```python
# 定义函数
def 函数名(形参列表):
    函数体逻辑
    return 返回值  # 无返回值可省略，默认返回None

# 调用函数
函数名(实参列表)
```

#### 1.2 简单示例

```python
# 定义求和函数
def add(a, b):
    res = a + b
    return res

# 调用
result = add(3, 5)
print(result)
```
- 无参函数：不传参数直接调用
- 无return：函数执行完毕自动返回`None`
- return 可终止函数，后续代码不再执行

---

### 2. 六大参数类型

#### 2.1 位置参数

按顺序一一对应传递，数量必须一致
```python
def func(x,y):
    print(x,y)
func(10,20)
```

#### 2.2 关键字参数

指定参数名传参，顺序可打乱
```python
func(y=20, x=10)
```
**规则**：位置参数必须写在关键字参数前面

#### 2.3 默认参数

形参设置默认值，不传参自动使用默认
```python
def info(name, age=18):
    print(name, age)
info("张三")       # age用默认值
info("李四",20)    # 手动传参覆盖默认
```
⚠️ 坑：**默认参数尽量不用可变对象（列表/字典）**

#### 2.4 可变位置参数 *args

接收**任意多个位置参数**，打包成**元组**
```python
def sum_all(*args):
    print(args)
    return sum(args)
sum_all(1,2,3,4)
```

#### 2.5 可变关键字参数 **kwargs

接收**任意多个关键字参数**，打包成**字典**
```python
def user_info(**kwargs):
    print(kwargs)
user_info(name="小明",sex="男")
```

#### 2.6 参数顺序规范

**位置参数 > 默认参数 > *args > **kwargs**

---

### 3. 变量作用域

#### 3.1 局部变量

定义在函数内部，**仅函数内生效**，函数执行结束销毁

#### 3.2 全局变量

定义在函数外部，**全局都可访问**
```python
num = 100  # 全局变量
def test():
    print(num)
```

#### 3.3 global 关键字

函数内**修改全局变量**必须声明
```python
a = 10
def change():
    global a
    a = 20
change()
print(a)
```

#### 3.4 nonlocal 关键字

**嵌套函数**中，修改外层函数局部变量（非全局）
```python
def outer():
    x = 10
    def inner():
        nonlocal x
        x = 20
    inner()
    print(x)
outer()
```

---

### 4. 匿名函数 lambda

极简单行函数，适合简单逻辑，无需def定义
#### 语法

```python
lambda 形参: 表达式
```
#### 示例

```python
f = lambda x,y:x+y
print(f(2,3))

# 条件表达式
lambda x:"偶数" if x%2==0 else "奇数"
```

---

### 5. 三大高阶函数

#### 5.1 map() 映射处理

对序列每个元素统一执行函数逻辑
```python
lst = [1,2,3]
res = map(lambda x:x*2, lst)
print(list(res))
```

#### 5.2 filter() 过滤筛选

按条件筛选序列元素，保留结果为True的数据
```python
lst = [1,2,3,4,5]
res = filter(lambda x:x%2==0, lst)
print(list(res))
```

#### 5.3 reduce() 累积运算

从左到右依次对元素两两运算
```python
from functools import reduce
lst = [1,2,3,4]
res = reduce(lambda a,b:a+b, lst)
print(res)
```

---

### 6. 闭包

#### 定义

1. 函数**嵌套**
2. 内层函数引用外层函数变量
3. 外层函数返回内层函数
#### 特点

外层变量常驻内存，不会随函数结束销毁
```python
def outer(msg):
    def inner():
        print(msg)
    return inner

f = outer("hello")
f()
```

---

### 7. 装饰器（函数进阶核心）

#### 作用

**不修改原函数源码、不改变调用方式**，动态给函数新增功能
常用场景：日志记录、权限校验、计时统计、缓存、异常捕获

#### 基础无参装饰器

```python
def decorator(func):
    def wrapper():
        print("新增前置功能")
        func()
        print("新增后置功能")
    return wrapper

@decorator
def say_hello():
    print("原函数逻辑")

say_hello()
```

#### 通用装饰器（适配任意参数函数）

```python
def timer(func):
    def wrapper(*args,**kwargs):
        print("开始执行")
        res = func(*args,**kwargs)
        print("执行结束")
        return res
    return wrapper
```

#### 带参数装饰器

外层再多套一层函数，接收装饰器参数

---

### 8. 函数式编程核心思想

1. 尽量用**纯函数**：相同输入必出相同输出，无外部依赖
2. 少用全局变量，多用参数传值
3. 优先使用 lambda + 高阶函数简化代码
4. 利用闭包保存状态，装饰器统一切面功能

---

### 9. 实际应用场景

1. **通用工具封装**：日期处理、数据校验、文件读写通用函数
2. **业务逻辑拆分**：把复杂业务拆分成多个独立函数
3. **代码解耦**：功能独立，修改一处不影响全局
4. **接口开发**：接口逻辑封装、参数统一校验
5. **日志/权限统一管控**：用装饰器全局统一加日志、登录校验
6. **数据批量处理**：map/filter快速批量清洗数据

需要我给你整理**函数专项必刷题（含装饰器、闭包、参数练习题）**吗？



---

## 四、面向对象编程

Python是**面向对象语言**，核心是**万物皆对象**，适合开发中大型项目。
### 核心知识点
1. 基础概念：类（`class`）、对象、属性、方法
2. 构造函数：`__init__()` 初始化对象
3. 三大特性：封装、继承、多态
4. 高级特性：魔术方法（`__str__`/`__repr__`）、类方法/静态方法、属性装饰器（`@property`）
5. 访问控制：公有属性、私有属性（`__变量`）

### 应用场景
Web开发、游戏开发、桌面软件、大型后端项目



### 核心思想

**万物皆对象**，把事物抽象成**类**，实例化为**对象**，用属性描述特征、方法描述行为，是中大型项目架构核心。

---

### 1. 基础概念

- **类 class**：事物的抽象模板，统一属性与行为
- **对象/实例**：根据类创建出来的具体实体
- **属性**：对象拥有的特征（变量）
- **方法**：对象具备的行为（函数）

#### 类与对象创建

```python
# 定义类
class Person:
    # 初始化构造方法
    def __init__(self, name, age):
        # 实例属性
        self.name = name
        self.age = age

    # 实例方法
    def talk(self):
        print(f"{self.name}在说话，今年{self.age}岁")

# 创建对象（实例化）
p1 = Person("张三", 18)
# 调用属性
print(p1.name)
# 调用方法
p1.talk()
```
- `self`：代表当前实例对象，必须写在实例方法首位

---

### 2. 构造方法 `__init__`

1. 创建对象**自动执行**，用于初始化属性
2. 可自定义参数，给对象赋初始值
3. 一个类**只能有一个**`__init__`
4. 无自定义构造方法，默认空构造

---

### 3. OOP 三大核心特性

#### 3.1 封装

- 含义：隐藏内部细节，对外暴露简单使用接口
- 作用：保护数据安全、简化调用
- **私有属性/私有方法**：变量/方法前加双下划线 `__`
```python
class People:
    def __init__(self):
        self.name = "小明"       # 公有属性
        self.__money = 1000     # 私有属性

    def get_money(self):
        # 对外提供访问接口
        return self.__money

p = People()
print(p.name)
# print(p.__money)  外部无法直接访问
print(p.get_money())
```

#### 3.2 继承

- 子类拥有父类所有属性和方法，实现**代码复用**
- 语法：`class 子类名(父类名):`
- `super()`：调用父类方法

```python
# 父类
class Animal:
    def eat(self):
        print("动物需要进食")

# 子类继承
class Dog(Animal):
    def bark(self):
        print("小狗汪汪叫")

dog = Dog()
dog.eat()   # 继承父类方法
dog.bark()  # 自有方法
```
- 多继承：`class A(B,C)` 同时继承多个父类
- 方法重写：子类重新定义和父类同名方法，覆盖父类逻辑

#### 3.3 多态

- 不同子类调用同一个父类方法，表现出不同行为
- 前提：继承 + 方法重写
- 作用：统一接口，适配不同子类

```python
class Animal:
    def speak(self):
        pass

class Cat(Animal):
    def speak(self):
        print("喵喵")

class Dog(Animal):
    def speak(self):
        print("汪汪")

def animal_speak(obj):
    obj.speak()

animal_speak(Cat())
animal_speak(Dog())
```

---

### 4. 三类方法

#### 4.1 实例方法

- 自带`self`，只能由**对象**调用
- 操作实例属性，最常用

#### 4.2 类方法 `@classmethod`

- 自带`cls`，由**类名直接调用**
- 操作类属性
```python
class Tool:
    num = 0
    @classmethod
    def count(cls):
        cls.num +=1
Tool.count()
```

#### 4.3 静态方法 `@staticmethod`

- 无`self`无`cls`，和类、对象无关
- 当作普通工具函数使用
```python
class MathUtil:
    @staticmethod
    def add(a,b):
        return a+b
print(MathUtil.add(1,2))
```

---

### 5. 属性装饰器 @property

把**方法伪装成属性**，简化调用，优雅获取/修改私有属性
```python
class Student:
    def __init__(self, score):
        self.__score = score

    # 读取属性
    @property
    def score(self):
        return self.__score

    # 修改属性
    @score.setter
    def score(self, val):
        if 0<=val<=100:
            self.__score = val

s = Student(80)
print(s.score)    # 像属性一样调用
s.score = 95
```

---

### 6. 常用魔术方法（内置方法）

| 方法       | 作用                       |
| ---------- | -------------------------- |
| `__init__` | 构造初始化                 |
| `__str__`  | print(对象) 自定义打印内容 |
| `__repr__` | 交互式命令行对象显示       |
| `__del__`  | 对象销毁自动执行           |
| `__len__`  | 支持 len(对象)             |

示例
```python
class Book:
    def __str__(self):
        return "这是一本书对象"
b = Book()
print(b)
```

---

### 7. 访问控制总结

1. **公有**：直接定义，内外均可访问
2. **私有 `__xxx`**：仅类内部可用，外部不能直接访问
3. 伪私有：只是名字改写，并非绝对加密

---

### 8. 应用场景

1. **Web后端**：视图类、模型类、业务实体类
2. **游戏开发**：角色、怪物、道具统一抽象类
3. **桌面程序**：窗口、按钮、组件面向对象设计
4. **大型项目**：分层架构、实体封装、业务解耦
5. **爬虫项目**：请求类、解析类、存储类拆分
6. **数据分析**：封装数据处理实体类

---

### 学习要点总结

1. 先分清**类与对象**、self作用
2. 吃透**封装、继承、多态**三大特性
3. 分清**实例方法、类方法、静态方法**使用场景
4. 掌握私有属性 + @property 规范数据读写
5. 熟练使用继承减少重复代码，用多态统一业务接口

需要我给你一套**面向对象入门实战案例（学生管理系统极简版）**吗？

---

## 五、模块与包管理（工程化开发）
解决**代码分文件管理**问题，让项目结构清晰、可维护。
### 核心知识点
1. 模块导入：`import` / `from ... import`
2. 自定义模块：拆分代码到多个`.py`文件
3. 包（Package）：`__init__.py`、分层包结构
4. 标准库模块：`os`/`sys`/`datetime`/`json`/`random`
5. 第三方库：pip安装、`requirements.txt` 依赖管理

### 应用场景
中型/大型项目开发、团队协作、代码复用

### 核心作用

拆分臃肿代码，**分文件分层级管理**，实现代码复用、项目结构化、团队协同开发，是从脚本编程走向工程化开发的必备能力。

---

### 1. 模块基础

#### 1.1 什么是模块

- 后缀为 `.py` 的Python文件就是**模块**
- 作用：功能拆分、代码隔离、重复调用

#### 1.2 两种导入方式

##### ① 整体导入

```python
# 导入整个模块
import demo
# 调用：模块名.功能
demo.fun()
```

##### ② 局部导入

```python
# 只导入指定函数/变量
from demo import fun, name
fun()

# 导入全部（不推荐）
from demo import *
```

#### 1.3 起别名简化调用

```python
import numpy as np
from demo import fun as f
```

#### 1.4 执行优先级

1. 先找**当前目录**
2. 再找系统环境变量路径
3. 最后找Python内置库

#### 1.5 主程序入口 `if __name__ == '__main__'`

```python
# demo.py
def test():
    print("测试函数")

# 只有当前文件直接运行才执行，被导入时不执行
if __name__ == '__main__':
    test()
```

---

### 2. 自定义模块开发

1. 新建多个独立 `.py` 文件，按功能拆分
2. 公共工具放统一工具模块
3. 业务逻辑拆分不同业务模块
4. 主文件统一调用

**目录示例**
```
project/
  main.py      # 主程序入口
  tools.py     # 工具函数模块
  user.py      # 用户业务模块
```

---

### 3. 包 Package（多层级管理）

#### 3.1 什么是包

包含`__init__.py`文件的**文件夹**，用来管理多个模块，实现分层架构。
- Python3 新版可省略`__init__.py`，**正式项目建议必加**

#### 3.2 标准包结构

```
mypackage/
    __init__.py
    module1.py
    module2.py
```

#### 3.3 包内导入

```python
# 导入包下指定模块
from mypackage import module1
# 导入模块内功能
from mypackage.module1 import func
```

#### 3.4 相对导入（包内部使用）

`.` 当前目录  `..` 上级目录
```python
from . import module2
from ..utils import tool
```
**注意：相对导入不能直接运行脚本，只能作为包导入使用**

#### 3.5 __init__.py 作用

1. 标识文件夹为Python包
2. 批量对外暴露接口，简化外部导入
```python
# __init__.py
from .module1 import add
from .module2 import info
```
外部直接使用：`from mypackage import add`

---

### 4. 常用内置标准库（必掌握）

| 模块     | 核心用途                           |
| -------- | ---------------------------------- |
| os       | 系统路径、文件目录、执行系统命令   |
| sys      | 程序参数、解释器信息、退出程序     |
| datetime | 日期时间处理、时间格式化           |
| json     | 字典与JSON字符串互转，接口数据解析 |
| random   | 随机数、随机抽取、打乱序列         |
| time     | 时间戳、程序休眠                   |
| math     | 数学运算、圆周率、三角函数         |

#### 常用示例

```python
import os
os.getcwd()          # 获取当前路径
os.mkdir("test")     # 创建文件夹

import json
json.dumps(字典)     # 转json字符串
json.loads(字符串)   # 转字典

from datetime import datetime
datetime.now()       # 获取当前时间
```

---

### 5. 第三方库管理

#### 5.1 pip 常用命令

```bash
# 安装库
pip install 库名
# 指定版本安装
pip install 库名==1.2.0
# 卸载
pip uninstall 库名
# 查看已安装
pip list
# 查看库信息
pip show 库名
```

#### 5.2 国内镜像加速（必备）

```bash
# 清华源
pip install 库名 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 5.3 项目依赖导出与导入

##### 1）导出环境依赖

```bash
pip freeze > requirements.txt
```

##### 2）新项目一键安装所有依赖

```bash
pip install -r requirements.txt
```
作用：团队协作、项目部署统一环境版本

---

### 6. 项目规范目录结构（企业通用）

```
python_project/
├── main.py               # 程序入口
├── requirements.txt      # 依赖清单
├── config/               # 配置包
│   ├── __init__.py
│   └── settings.py       # 全局配置
├── utils/                # 工具包
│   ├── __init__.py
│   ├── file_tool.py
│   └── time_tool.py
├── core/                 # 核心业务包
│   ├── __init__.py
│   └── service.py
└── logs/                 # 日志文件夹
```

---

### 7. 常见报错解决

1. **ModuleNotFoundError**
   - 模块名写错、文件不在同级目录、未加入环境变量
2. **相对导入报错**
   - 不要直接运行包内py文件，从顶层入口启动
3. **版本冲突**
   - 使用虚拟环境隔离项目依赖

---

### 8. 应用场景

1. **中型/大型项目**：分层解耦，结构清晰易维护
2. **团队协作开发**：每人开发独立模块，互不冲突
3. **框架开发**：Flask/Django 全靠包与模块分层
4. **工具封装**：通用代码做成模块，全项目复用
5. **服务器部署**：通过`requirements.txt`快速搭建环境
6. **开源项目**：规范包结构，方便他人调用

---

### 学习重点总结

1. 吃透**模块导入**三种写法
2. 分清**包与文件夹**区别，掌握`__init__.py`用法
3. 熟练使用**os、datetime、json**三大标准库
4. 精通`pip`命令与**requirements.txt**依赖管理
5. 学会搭建企业级规范项目目录结构

需要我给你整理**标准项目空架构模板**，直接复制就能用吗？

---

## 六、文件IO与数据持久化（数据存储）
程序和**外部文件交互**的核心，读取/写入数据必备。
### 核心知识点
1. 文件操作：`open()` 函数、读/写模式（`r`/`w`/`a`/`r+`）、上下文管理器（`with`）
2. 常见文件类型：文本文件、CSV、Excel、JSON、XML
3. 路径处理：`os.path`/`pathlib` 跨平台路径管理
4. 数据库基础：SQLite（内置轻量数据库）、MySQL连接

### 应用场景
日志记录、数据导出、配置文件、本地数据存储

### 核心作用

实现程序与本地文件、数据库的数据读写，把运行数据**长久保存**，断电不丢失，是业务数据落地必备技能。

---

### 1. 基础文件操作 open()

#### 1.1 基础语法

```python
open(file, mode='r', encoding='utf-8')
```
- `file`：文件路径
- `mode`：读写模式
- `encoding`：指定编码，文本文件必写防乱码

#### 1.2 常用读写模式

| 模式    | 含义       | 特点                         |
| ------- | ---------- | ---------------------------- |
| `r`     | 只读       | 默认模式，文件不存在报错     |
| `w`     | 只写       | 清空原有内容，不存在则新建   |
| `a`     | 追加写入   | 末尾追加，不清空，不存在新建 |
| `r+`    | 读写       | 可读可写，不会清空原内容     |
| `rb/wb` | 二进制读写 | 图片、视频、音频             |

#### 1.3 基础读写

```python
# 读取文件
f = open("test.txt","r",encoding="utf-8")
content = f.read()
f.close()

# 写入文件
f = open("test.txt","w",encoding="utf-8")
f.write("写入内容")
f.close()
```

#### 1.4 上下文管理器 with（推荐）

**自动关闭文件，无需手动 close**
```python
# 读
with open("test.txt","r",encoding="utf-8") as f:
    text = f.read()

# 追加写入
with open("test.txt","a",encoding="utf-8") as f:
    f.write("\n新增一行数据")
```

#### 1.5 分行读取

```python
# 按行读取所有
with open("test.txt","r",encoding="utf-8") as f:
    lines = f.readlines()

# 逐行遍历
for line in f:
    print(line.strip())
```

---

### 2. 路径处理

#### 2.1 os.path 传统路径

```python
import os
# 获取当前脚本路径
print(os.getcwd())
# 拼接路径（跨平台通用）
path = os.path.join("data","info.txt")
# 判断文件是否存在
os.path.exists(path)
# 创建文件夹
os.mkdir("data")
```

#### 2.2 pathlib 新式优雅路径（Python3.4+）

```python
from pathlib import Path
p = Path("data/test.txt")
p.write_text("内容",encoding="utf-8")
text = p.read_text(encoding="utf-8")
# 判断是否存在
print(p.exists())
```

---

### 3. 主流文件类型读写

#### 3.1 JSON 文件（最常用配置/接口数据）

```python
import json

# 写入json
data = {"name":"张三","age":18}
with open("data.json","w",encoding="utf-8") as f:
    json.dump(data,f,ensure_ascii=False,indent=2)

# 读取json
with open("data.json","r",encoding="utf-8") as f:
    res = json.load(f)
```

#### 3.2 CSV 表格数据

```python
import csv
# 写入
with open("data.csv","w",newline="",encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerow(["姓名","年龄"])
    writer.writerow(["李四",20])

# 读取
with open("data.csv","r",encoding="utf-8") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

#### 3.3 Excel 读写

- 库：`openpyxl`（读写xlsx）
```bash
pip install openpyxl
```
实现表格批量数据导入导出、报表生成。

#### 3.4 二进制文件

图片、视频、压缩包使用 `rb / wb` 模式读写，多用于文件复制、上传下载。

---

### 4. 数据持久化 — 数据库基础

#### 4.1 SQLite 内置轻量数据库

**无需安装服务，本地文件数据库**，小型项目首选
```python
import sqlite3
# 连接数据库，不存在自动创建
conn = sqlite3.connect("test.db")
cur = conn.cursor()

# 建表
cur.execute("create table if not exists user(name text,age int)")
# 插入数据
cur.execute("insert into user values(?,?)",("王五",22))
conn.commit()  # 提交事务

# 查询
res = cur.execute("select * from user").fetchall()
print(res)

cur.close()
conn.close()
```

#### 4.2 MySQL 数据库连接

依赖库：`pymysql`
```bash
pip install pymysql
```
可实现企业级项目增删改查、业务数据持久存储。

---

### 5. 常见IO异常

1. 文件路径错误 → 用绝对路径+路径拼接解决
2. 中文乱码 → 统一指定 `encoding="utf-8"`
3. 文件被占用 → 确保文件关闭，优先使用`with`
4. 超大文件读取 → 禁止`read()`一次性读完，改用逐行读取

---

### 6. 实际应用场景

1. **日志记录**：程序运行日志、错误日志写入本地文件
2. **配置文件**：用JSON/TXT存放账号、地址、阈值等配置
3. **数据导出**：爬虫数据、统计数据导出CSV/Excel报表
4. **本地缓存**：临时数据存入文件，减少重复请求
5. **小型系统**：学生管理、进销存用SQLite存数据
6. **自动化办公**：批量读写文档、整理办公数据

---

### 学习重点

1. 熟练掌握 **with上下文 + 四大文件模式**
2. 必会 **JSON 读写**（工作最高频）
3. 掌握路径拼接，适配 Windows / Mac / Linux
4. 学会 SQLite 基础增删改查，实现无服务数据存储
5. 分清文本文件与二进制文件使用场景

需要我给你整理**文件IO常用工具类（读写JSON/CSV/日志封装）**吗？



---

## 七、高级编程特性（进阶核心）
提升代码效率、性能、优雅度的必备技能，是**初中级→高级程序员**的分水岭。
### 核心知识点
1. 并发编程：多线程（`threading`）、多进程（`multiprocessing`）、协程（`asyncio`）
2. 正则表达式：`re` 模块——字符串匹配、提取、替换
3. 反射与元编程：动态获取/修改类和对象属性
4. 上下文管理器：自定义`with`语句
5. 性能优化：时间复杂度、生成器、缓存（`lru_cache`）

### 应用场景
爬虫加速、高并发服务、文本处理、大数据处理



### 核心定位

从**业务实现**走向**性能优化、架构设计、高效开发**，是中级进阶高级程序员核心分水岭，主打提速、精简、灵活扩展。

---

### 1. 并发编程（重中之重）

#### 1.1 三大并发方案对比

- **多线程 threading**
  - 适用：**IO密集型**（爬虫、文件读写、网络请求、等待休眠）
  - 特点：轻量、开销小、受GIL全局解释器锁限制，**无法利用多核CPU**
  - 常用：线程池 `ThreadPoolExecutor`

- **多进程 multiprocessing**
  - 适用：**CPU密集型**（数据分析、运算、解压、大批量计算）
  - 特点：绕过GIL、利用多核、开销大、进程间通信麻烦
  - 常用：进程池 `ProcessPoolExecutor`

- **协程 asyncio**
  - 适用：**超高IO并发**（异步爬虫、异步接口、高并发服务）
  - 特点：单线程实现高并发、开销极小、效率最高、语法异步
  - 关键字：`async / await`

#### 1.2 极简使用

```python
# 线程池快速并发
from concurrent.futures import ThreadPoolExecutor

def task(x):
    return x*2

with ThreadPoolExecutor(5) as t:
    res = list(t.map(task, range(10)))
```

#### 1.3 GIL锁总结

Python同一时刻**只有一个线程执行CPU代码**，IO阻塞时会释放锁，所以IO用线程，计算用进程。

---

### 2. 正则表达式 re 模块

#### 核心作用

快速匹配、提取、清洗、替换复杂字符串，日志解析、数据提取、表单校验必备。

#### 常用元字符

```
. 任意字符  ^开头  $结尾  \d数字  \w字母数字下划线
\s空白符  *任意次数  +至少一次  ?非贪婪  ()分组
```

#### 常用方法

```python
import re

re.match()      # 从头匹配
re.search()     # 全局找第一个
re.findall()    # 提取所有匹配结果（最常用）
re.sub()        # 正则替换
re.split()      # 正则分割字符串
```

#### 实战场景

手机号、邮箱、身份证、URL提取、HTML文本清洗、日志关键字筛选

---

### 3. 反射与元编程

#### 3.1 反射（动态操作对象）

运行时**动态获取、判断、调用、修改**属性和方法，不用写死代码。
```python
class User:
    name = "张三"
    def run(self):
        print("跑步")

u = User()
hasattr(u,"name")       # 判断是否有属性
getattr(u,"name")       # 获取属性
setattr(u,"age",18)     # 设置属性
delattr(u,"age")        # 删除属性
getattr(u,"run")()      # 动态调用方法
```
**用途**：通用框架、路由分发、插件机制、自动适配参数

#### 3.2 元编程 metaclass

- `type` 是所有类的父类
- 自定义元类 `metaclass` 可**控制类的创建过程**
- 用途：ORM框架、字段校验、全局类拦截、代码自动注册

---

### 4. 自定义上下文管理器 with

#### 作用

统一实现**进入前置逻辑 + 退出后置逻辑**，自动释放资源，替代手动开关。
```python
class MyFile:
    def __enter__(self):
        print("打开资源")
        return self
    def __exit__(self,exc_type,exc_val,exc_tb):
        print("关闭资源")
    def do_work(self):
        print("执行业务")

with MyFile() as f:
    f.do_work()
```
**业务场景**：数据库自动关闭、连接自动释放、锁自动释放、事务自动提交回滚

---

### 5. 代码性能优化

#### 5.1 基础优化思想

- 减少循环嵌套、降低时间复杂度
- 优先内置函数/推导式，比for循环更快
- 大数据优先**生成器**，少用列表存全量数据

#### 5.2 缓存装饰器 lru_cache

缓存函数执行结果，重复调用直接读取缓存，极大提速
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fib(n):
    if n<=2:
        return 1
    return fib(n-1)+fib(n-2)
```
适用：递归计算、重复查询、静态数据获取

#### 5.3 其他优化手段

1. 用`join`拼接字符串，少用`+`
2. 局部变量访问快于全局变量
3. 海量数据迭代遍历，不一次性载入内存
4. 使用内置模块替代手写逻辑

---

### 6. 高级语法补充

1. **深浅拷贝**
   - 浅拷贝`copy()`：只拷贝外层
   - 深拷贝`deepcopy()`：递归全拷贝，独立互不影响
2. **枚举类 enum**
   固定常量状态统一管理，替代魔法数字
3. **walrus海象运算符 :=**
   一行完成赋值+判断，精简代码

---

### 7. 三大技术选型口诀

1. **IO密集** → 优先线程 / 协程
2. **CPU密集** → 优先多进程
3. **字符串处理** → 优先正则
4. **资源自动管理** → 优先with上下文
5. **动态灵活架构** → 优先反射
6. **重复高耗时函数** → 优先lru_cache缓存

---

### 应用场景

1. **爬虫加速**：线程池/异步协程批量爬取
2. **高并发接口**：FastAPI异步、协程支撑高吞吐
3. **文本大数据处理**：正则清洗、生成器流式读取
4. **分布式任务**：多进程并行计算
5. **框架二次开发**：反射+元编程实现插件化架构
6. **运维自动化**：批量并发执行服务器脚本

---

### 进阶学习路线

1. 吃透**协程asyncio**与异步请求
2. 精通**线程锁、进程通信、队列**
3. 熟练正则批量提取清洗数据
4. 掌握反射实现通用工具框架
5. 养成**性能优化**编码习惯

需要我给你整理：**并发编程全套实战代码（线程/进程/协程）+ 正则常用表达式大全**吗？

---

## 八、实战应用方向（核心编程落地）
学完核心语法后，Python最常用的4大实战方向，直接对接工作需求：
1. **网络编程**
   爬虫（`requests`/`BeautifulSoup`）、接口开发（`Flask`/`FastAPI`）、socket通信
2. **数据处理与分析**
   `Pandas`/`NumPy`数据清洗、数据可视化（`Matplotlib`）
3. **自动化办公**
   Excel/Word/PPT自动化、邮件发送、定时任务
4. **测试与运维**
   自动化测试（`Selenium`）、服务器脚本、监控工具

### 整体说明

前面七大核心语法全部学完，直接切入**职场主流4大就业方向**，每方向包含：技术栈、核心知识点、实战项目、就业岗位、学习顺序

---

### 1. 网络编程方向（最热门）

#### 细分路线

##### 1）网络爬虫

**核心库**
- 网络请求：`requests`、`urllib`
- 网页解析：`BeautifulSoup`、`lxml`、`pyquery`
- 异步爬虫：`aiohttp`
- 反爬应对：请求头、代理IP、Cookie、验证码识别
- 动态网页：`Selenium`、`Playwright`

**核心技能**
- 静态页面数据抓取
- AJAX接口抓包爬取
- 多线程/协程批量爬取
- 数据入库MySQL/CSV
- 防封禁、IP代理、登录态维持

**实战项目**
- 电商商品数据采集
- 新闻资讯全站爬虫
- 短视频标题/文案采集
- 招聘岗位信息汇总

##### 2）Web后端开发

**轻量框架**
- Flask：小型项目、快速接口
- FastAPI：高性能异步接口、现在主流

**必备知识点**
- 路由、请求参数、响应、JSON接口
- 数据库CURD、分页、鉴权
- 跨域、日志、异常统一处理
- 接口文档自动生成

**就业岗位**
Python后端开发、接口开发、API工程师

##### 3）Socket网络通信

- TCP/UDP套接字编程
- 客户端服务端通信
- 即时通讯简易架构

**应用场景**
内网通信、设备对接、简易消息推送

---

#### 2. 数据处理与数据分析方向（数据分析岗主力）

##### 核心技术栈

- 数值计算：**NumPy**
- 数据分析清洗：**Pandas**（重中之重）
- 数据可视化：Matplotlib、Seaborn、Pyecharts
- 高级：Pandas透视表、分组聚合、缺失值处理、时间序列

##### 核心学习内容

1. NumPy：数组运算、矩阵、批量数值处理
2. Pandas
   - DataFrame表格创建、读取Excel/CSV
   - 数据筛选、去重、填充缺失值
   - 分组统计、聚合计算、合并表格
   - 时间数据处理、业务报表统计
3. 可视化：折线图、柱状图、饼图、热力图

##### 实战项目

- 销售业绩数据分析
- 用户行为数据统计
- 财务报表自动汇总
- 疫情/天气数据可视化分析

##### 就业岗位

数据分析师、数据运营、商业数据分析、BI报表开发

---

#### 3. 自动化办公方向（零基础最快变现）

##### 核心用途

解放办公重复劳力，职场白领、行政、财务必备
##### 主流库

- Excel自动化：`openpyxl`、`pandas`、`xlwings`
- Word自动化：`python-docx`
- PPT自动化：`python-pptx`
- 邮件自动化：`smtplib`
- 定时任务：`schedule`、`APScheduler`
- 系统自动化：文件批量处理、批量重命名

##### 常用实战功能

1. 批量读写Excel、批量汇总表格
2. 自动生成周报/月报/报表
3. 自动发送工作邮件、群发通知
4. 电脑文件分类整理、批量解压
5. 定时执行脚本、自动打卡

##### 适用人群

职场办公人员、行政、财务、文员、副业接单

---

#### 4. 自动化测试 & 运维开发方向（互联网刚需）

##### 1）自动化测试

**核心库**
- 网页自动化：**Selenium**
- APP自动化：Appium
- 接口自动化：requests+unittest/pytest
- 测试框架：pytest、allure测试报告

**工作内容**
- 网页功能自动化点击、输入、登录
- 接口批量自动化测试
- 自动生成测试报告
- 回归测试脚本

##### 2）运维自动化

**核心技能**
- Linux系统命令调用
- 服务器批量管理
- 日志分析、服务监控
- 批量部署脚本、巡检脚本
- 监控告警、磁盘/内存状态检测

#### 就业岗位

自动化测试工程师、测试开发、运维开发、运维工程师







# Python 异步编程 

Python 异步编程核心是**单线程内实现非阻塞并发**，专门解决 **I/O 密集型任务**（网络请求、文件读写、数据库查询、爬虫等）的效率问题。

我会按照**技术方案、实现方式、核心组件、使用场景**四个维度，把 Python 异步编程体系完整分类讲清楚，新手也能看懂。

---

## 一、按技术方案分类

Python 异步编程主要分 **2 大流派**，也是你最需要区分的：

**1. 原生异步 asyncio（官方标准）**

- Python 3.4+ 内置，**标准库**，无需安装第三方包
- 基于 **协程（coroutine）+ 事件循环（event loop）**
- 语法：`async def` 定义协程，`await` 调用
- 优点：官方支持、稳定、生态越来越完善
- 缺点：部分老库不兼容，需要全套异步生态

**2. 第三方异步框架（基于 asyncio 封装）**

- **aiohttp**：异步 HTTP 客户端/服务端（最常用）
- **FastAPI**：现代高性能 Web 框架（异步）
- **httpx**：支持同步+异步的 HTTP 客户端
- **aiomysql、aioredis**：异步数据库驱动

这是 Python 异步最关键、最实用的分类，**直接决定你写代码的方式**，我给你掰开揉碎讲清楚。

---

### 一、原生异步：asyncio

#### 1. 是什么？

Python 官方内置的**异步编程基础框架**，从 3.4 开始正式支持，3.7+ 变得非常好用。
它是**所有第三方异步库的底层基石**。

#### 2. 核心原理（必须懂）

asyncio 靠 3 个东西实现异步：
1. **协程（Coroutine）**：可以暂停/恢复的函数，用 `async def` 定义
2. **事件循环（Event Loop）**：调度器，不停检查哪个任务可以运行
3. **非阻塞 I/O**：等待网络/文件时，程序去做别的事

#### 3. 核心语法

```python
import asyncio

# 1. 定义协程函数
async def do_something():
    await asyncio.sleep(1)  # 模拟非阻塞等待
    return "完成"

# 2. 主协程
async def main():
    result = await do_something()
    print(result)

# 3. 运行异步程序
asyncio.run(main())
```

#### 4. 你必须掌握的 asyncio 功能

- `asyncio.run()`：运行入口（Python 3.7+）
- `asyncio.create_task()`：把协程包装成任务，**并发执行**
- `asyncio.gather()`：批量运行多个任务，等待全部完成
- `asyncio.wait()`：更灵活地等待任务
- `asyncio.sleep()`：异步等待（**千万不要用 time.sleep()**）

#### 5. 优点

- 官方内置，**不用安装**
- 稳定、兼容性好
- 所有现代异步库都基于它
- 性能极高，单线程可处理万级并发

#### 6. 缺点

- 只能和**异步兼容代码**一起用
- 同步函数（如 requests、pymysql）不能直接 await
- 学习曲线比同步代码陡

---

### 二、第三方异步框架/库（基于 asyncio 封装）

这是**实际工作中用得最多**的部分，它们让 asyncio 变得好用、能直接做项目。

我按**用途分类**给你详解：

---

#### 1. 异步 HTTP 请求：aiohttp（最常用）

**地位**：Python 异步 HTTP 事实标准
**作用**：发送异步网络请求、写异步 Web 服务

#### 示例代码

```python
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:  # 异步会话
        async with session.get(url) as resp:        # 异步请求
            return await resp.text()                # 异步读取结果

async def main():
    html = await fetch("https://www.baidu.com")
    print(html[:100])

asyncio.run(main())
```

**特点**
- 完全异步，比 requests 快几倍～几十倍
- 支持客户端、服务端
- 爬虫、微服务调用必备

---

#### 2. 现代异步 Web 框架：FastAPI

**地位**：目前最火的 Python 异步 Web 框架
**作用**：写高性能 API 接口、微服务、后端服务

##### 极简示例

```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/")
async def home():
    await asyncio.sleep(1)  # 异步操作
    return {"message": "Hello FastAPI"}
```

**特点**
- 天生支持 async/await
- 速度接近 Go/Node.js
- 自动生成 API 文档
- 大厂大量使用

---

#### 3. 全能 HTTP 客户端：httpx

**地位**：同步/异步双持 HTTP 工具
**作用**：既能像 requests 一样用，也支持异步

示例：
```python
import httpx

async def test():
    async with httpx.AsyncClient() as client:
        resp = await client.get("https://www.baidu.com")
        print(resp.text)
```

**特点**
- API 和 requests 几乎一样
- 支持 HTTP/1.1、HTTP/2
- 同步异步通用，学习成本极低

---

#### 4. 异步数据库驱动

**作用**：让数据库操作也变成异步，避免阻塞

常用库：
- **aiomysql**：异步 MySQL
- **aioredis**：异步 Redis
- **motor**：异步 MongoDB
- **asyncpg**：异步 PostgreSQL（最快）

示例（aiomysql）：
```python
import aiomysql
import asyncio

async def query():
    conn = await aiomysql.connect(host='localhost', user='root', password='123')
    cur = await conn.cursor()
    await cur.execute("SELECT 1")
    result = await cur.fetchone()
    print(result)

asyncio.run(query())
```

---

#### 5. 异步文件操作：aiofiles

**作用**：异步读写文件，不阻塞事件循环

```python
import aiofiles
import asyncio

async def write_file():
    async with aiofiles.open("test.txt", "w") as f:
        await f.write("异步写入")

asyncio.run(write_file())
```

---

### 三、两者关系

```
asyncio（底层地基）
   ↑
aiohttp / FastAPI / aiomysql / aiofiles（上层建筑）
```

- **asyncio** = 地基 + 钢筋
- **第三方异步库** = 房子（你直接住）

---

### 四、最实用的选择建议

#### 你应该怎么选？

1. **写爬虫 / 并发请求** → aiohttp / httpx
2. **写 Web API / 微服务** → FastAPI
3. **操作数据库** → aiomysql / aioredis / motor
4. **底层工具开发** → 直接用 asyncio

#### 最重要的一句话

**所有第三方异步库，底层全都是 asyncio。**
学会 asyncio，所有异步库一通百通。

---

### 五、总结

1. **asyncio**：官方基础库，是所有异步的根
2. **第三方异步框架**：基于 asyncio 封装，用于实战开发
3. 最常用组合：
   - FastAPI + aiohttp + aiomysql
   - 爬虫：aiohttp
4. 它们**全部共用一套语法**：`async def` + `await`

需要我给你整理一份**可直接运行的完整实战代码**（比如异步爬虫、异步接口）吗？

---

## 二、按实现机制分类（底层原理）

**1. 协程（Coroutine）—— 异步核心**

- 用户态轻量级线程，**由程序自己控制切换**，不依赖操作系统
- 开销极小，单线程可跑上万协程
- 遇到 I/O 自动挂起，执行其他任务，I/O 完成后恢复

 **2. 事件循环（Event Loop）**

- 异步程序的**“总指挥”**
- 负责：监听 I/O 事件、调度协程执行、切换任务

**3. 任务（Task）**

- 对协程的包装，让协程**并发运行**
- 一个事件循环可以同时调度成千上万个 Task

**4. 未来对象（Future）**

- 底层数据结构，代表**“未来会完成的操作结果”**
- 日常开发基本不用直接操作

### 1. 协程 Coroutine

#### 核心定义

协程是**用户态轻量级执行单元**，运行在同一线程内，**切换由代码逻辑主动控制**，不经过操作系统内核调度，无线程内核态切换开销。

#### 核心特性

1. **自主可控切换**：不是系统抢占式调度，遇到`await`阻塞点主动让出执行权
2. **资源开销极低**：无独立栈内存开销，单线程轻松支撑数万、十万级协程并发
3. **状态可保存恢复**：阻塞时暂停当前上下文，IO就绪后原地恢复继续执行
4. **串行执行本质**：同一时刻同一线程内**只有一个协程在运行**，并非真正并行

#### 语法标识

- `async def` 定义协程函数，调用后得到**协程对象**，不会立即执行
- 必须通过`await`、事件循环、Task 驱动才能运行

#### 适用场景

仅适配**IO阻塞场景**（网络请求、数据库、文件读写、等待休眠），纯计算密集型无法提速。

---

### 2. 事件循环 EventLoop

#### 核心定位

Python 异步程序**唯一调度中枢、总指挥**，整个异步程序的驱动核心。

#### 核心工作职责

1. 统一监听所有 IO 阻塞事件（网络连接、读写、超时）
2. 遍历就绪协程，分配CPU执行权限
3. 协程阻塞时挂起，IO完成后唤醒重新入队调度
4. 管理所有异步任务队列、定时器、回调函数

#### 运行逻辑

死循环轮询：**检查就绪任务 → 执行 → 遇阻塞切走 → 等待IO → 唤醒重试**

#### 版本简化

- Python3.7+：`asyncio.run()` 自动创建、运行、关闭事件循环，无需手动管理
- 低版本：需手动`get_event_loop()`创建循环

#### 关键点

**一个异步进程默认只有一个主线程事件循环**，所有协程全由它调度。

---

### 3. 任务 Task

#### 核心定义

`asyncio.Task` 是**协程的高层封装对象**，作用是把普通协程加入事件循环，实现**并发调度**。

#### 核心作用

1. 将零散协程注册到事件循环，使其具备被调度资格
2. 独立管理单个协程的运行状态、执行结果、异常
3. 支持取消任务、设置回调、获取返回值

#### 使用方式

```python
# 创建并发任务
task = asyncio.create_task(协程函数())
```
不创建Task时，协程只能**串行等待**；创建多个Task后，事件循环自动交替调度实现并发。

#### 数量上限

仅受内存限制，单事件循环可调度上万Task无压力。

---

### 4. 未来对象 Future

#### 核心定义

**底层最基础的异步结果容器**，代表一个**尚未完成、未来才会得到的执行结果**，是Task的底层父类。

#### 核心特性

1. 只存储状态：未完成、已完成、异常
2. 手动设置结果/异常，触发绑定的回调函数
3. 无业务执行逻辑，纯状态容器

#### 层级关系

`Future（底层结果容器）` → 封装衍生出 `Task（可执行协程任务）`

#### 开发使用

**业务开发几乎不用手动写Future**，仅在封装底层异步组件、自定义调度器时才会接触，日常`await`、Task完全屏蔽该底层细节。

---

### 四者层级联动关系（极简流程）

1. 编写`async def`定义**协程**（业务逻辑）
2. 协程被包装成**Task**，送入队列
3. **事件循环**统一调度所有Task，遇到IO阻塞切换
4. 所有异步执行的结果，底层都由**Future**承载存储

### 一句话总结

协程是干活的工人，Task是工人编号排班，事件循环是工地包工头，Future是存放完工结果的收纳盒。

---

## 三、按代码写法/语法分类
### 1. 现代语法：async/await（Python 3.5+）
**最推荐、最主流**
```python
import asyncio

# 定义异步函数
async def async_task():
    await asyncio.sleep(1)  # 模拟非阻塞 I/O
    return "完成"

# 运行异步主函数
async def main():
    result = await async_task()
    print(result)

asyncio.run(main())
```

---

#### 一、先记住 2 个核心关键字

1. **`async`**：**定义**异步函数（协程）
2. **`await`**：**执行**异步操作，并**等待它完成**

只要记住：**async 定义，await 调用**。

---

#### 二、完整代码逐行精讲

```python
import asyncio  # 导入官方异步库（必须）
```

##### 1. 定义异步函数

```python
# async 修饰 → 这是一个异步函数（协程）
async def async_task():
    # await 后面必须跟 异步操作
    await asyncio.sleep(1)  # 【非阻塞】等待1秒
    return "完成"
```
- 普通函数：`def`
- 异步函数：**`async def`**
- 里面必须用 **`await`** 调用异步操作
- **`asyncio.sleep(1)`** 是**异步等待**，不会卡住整个程序

---

##### 2. 主异步函数

```python
async def main():
    # 调用异步函数，必须加 await
    result = await async_task()
    print(result)
```
- 调用异步函数，**必须加 `await`**
- 不加 await → 只得到一个协程对象，**不会执行**
- await = 等这个异步任务做完，再继续往下走

---

##### 3. 启动异步程序

```python
asyncio.run(main())
```
- **Python 3.7+ 唯一标准启动方式**
- 自动创建事件循环 → 运行任务 → 关闭循环
- 这是**异步程序的入口**

---

#### 三、最关键：await 到底做了什么？

`await` 是异步的灵魂，它做 3 件事：
1. **暂停当前函数**，把执行权交出去
2. 去执行别的就绪任务（实现并发）
3. 等异步操作完成后，**自动回来继续执行**

##### 重点区别（一定要懂）

- `time.sleep(1)` → **阻塞**，整个程序卡死不动
- `asyncio.sleep(1)` → **非阻塞**，程序可以去干别的事

---

#### 四、进阶：并发写法（真正的异步威力）

上面是串行，下面是**同时跑多个任务**：

```python
import asyncio

async def task(name, t):
    print(f"任务 {name} 开始")
    await asyncio.sleep(t)  # 非阻塞等待
    print(f"任务 {name} 结束")
    return name

async def main():
    # 创建 2 个任务，让它们并发执行
    t1 = asyncio.create_task(task("A", 2))
    t2 = asyncio.create_task(task("B", 1))

    # 等待所有任务完成
    res1 = await t1
    res2 = await t2

# 总耗时 ≈ 2秒，不是 3秒！
asyncio.run(main())
```

##### 输出

```
任务 A 开始
任务 B 开始
任务 B 结束
任务 A 结束
```
**真正并发：时间取最长的那个，不是相加！**

---

#### 五、必须遵守的 3 条铁律

1. **只有 async 函数里面，才能用 await**
2. **await 后面只能跟 可等待对象**（协程、Task、Future）
3. **异步函数不能直接调用**，必须 await 或放进 Task

---

#### 六、一句话总结

- **`async def`**：我是异步函数
- **`await`**：我等你，但不卡死程序
- **`asyncio.run()`**：启动异步程序

这套语法就是 Python 异步的**全部现代写法**，学会它，就能写 FastAPI、aiohttp、异步爬虫、异步微服务。

需要我给你演示**异步爬虫 / 异步接口 / 并发下载**的实战代码吗？

### 2. 老式语法：@asyncio.coroutine + yield from（已废弃）

Python 3.4 早期写法，**现在不用学**

---

## 四、按使用场景分类

### 1. 异步网络请求
- 爬虫、API 调用、微服务通信
- 工具：`aiohttp`、`httpx`

### 2. 异步 Web 服务
- 高并发 API 服务
- 框架：`FastAPI`、`Starlette`

### 3. 异步数据库
- MySQL、Redis、MongoDB
- 库：`aiomysql`、`aioredis`、`motor`

### 4. 异步文件 IO
- 大文件读写
- 库：`aiofiles`

### 5. 异步消息队列
- Kafka、RabbitMQ 异步消费

---

## 五、按并发模型对比（异步 vs 多线程 vs 多进程）
| 方式         | 类型         | 切换成本 | 适用场景           |
| ------------ | ------------ | -------- | ------------------ |
| **异步协程** | 单线程并发   | 极低     | I/O 密集型（推荐） |
| 多线程       | 操作系统调度 | 中       | I/O 密集型         |
| 多进程       | 操作系统调度 | 高       | CPU 密集型（计算） |

**重要结论：**
- **CPU 密集** → 用**多进程**
- **I/O 密集** → 用**异步协程**（效率最高）

---

## 六、异步编程核心组件分类
### 1. 异步函数（协程函数）
```python
async def func():
    pass
```

### 2. 异步上下文管理器
```python
async with aiohttp.ClientSession() as session:
    pass
```

### 3. 异步迭代器
```python
async for item in async_iter():
    pass
```

### 4. 同步转异步（run_in_executor）
把不支持异步的同步函数，放到线程池执行：
```python
loop = asyncio.get_event_loop()
await loop.run_in_executor(None, sync_func)
```

---

## 七、完整学习路径

1. 基础：`async` / `await` 语法
2. 核心：`asyncio.run()`、`asyncio.create_task()`
3. 并发：`asyncio.gather()` 并发执行多任务
4. 实战：`aiohttp` 异步请求、`FastAPI` 异步服务
5. 进阶：异步锁、队列、子进程、信号量

