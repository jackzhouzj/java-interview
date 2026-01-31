# Python基础 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Python版本**：3.9+
- **最新稳定版**：3.12
- **推荐版本**：3.10 或 3.11

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：25-35小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- 计算机基础知识
- 基本的编程概念
- 无需其他编程语言基础

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 掌握Python基础语法和数据类型
- [ ] 理解Python的控制流和函数
- [ ] 熟练使用Python的数据结构
- [ ] 掌握面向对象编程
- [ ] 能够进行文件操作和异常处理
- [ ] 理解Python模块和包的使用
- [ ] 能够独立编写Python程序

## 📖 目录

1. [Python简介](#1-python简介)
2. [环境搭建](#2-环境搭建)
3. [基础语法](#3-基础语法)
4. [数据类型](#4-数据类型)
5. [控制流](#5-控制流)
6. [函数](#6-函数)
7. [数据结构](#7-数据结构)
8. [面向对象编程](#8-面向对象编程)
9. [模块和包](#9-模块和包)
10. [文件操作](#10-文件操作)
11. [异常处理](#11-异常处理)
12. [最佳实践](#12-最佳实践)

---

## 1. Python简介

### 1.1 什么是Python

Python是一种高级、解释型、面向对象的编程语言，由Guido van Rossum于1991年创建。

**Python的特点**：
- 🔥 **简洁易读**：语法简洁，接近自然语言
- 🔥 **跨平台**：支持Windows、Linux、macOS
- 🔥 **丰富的库**：拥有强大的标准库和第三方库
- 🔥 **应用广泛**：Web开发、数据科学、AI、自动化等

### 1.2 Python的应用领域

- **Web开发**：Django、Flask
- **数据科学**：NumPy、Pandas、Matplotlib
- **人工智能**：TensorFlow、PyTorch、Scikit-learn
- **自动化运维**：Ansible、SaltStack
- **爬虫**：Scrapy、BeautifulSoup

---

## 2. 环境搭建

### 2.1 安装Python

#### Windows
```bash
# 下载Python安装包
# 访问 https://www.python.org/downloads/
# 下载并安装，记得勾选"Add Python to PATH"
```

#### macOS
```bash
# 使用Homebrew安装
brew install python@3.11
```

#### Linux
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3.11

# CentOS/RHEL
sudo yum install python3.11
```

### 2.2 验证安装

```bash
# 检查Python版本
python --version
# 或
python3 --version

# 检查pip版本
pip --version
```

### 2.3 配置开发环境

#### 推荐IDE
- **PyCharm**：功能强大的Python IDE
- **VS Code**：轻量级编辑器，配合Python插件
- **Jupyter Notebook**：交互式开发环境

#### 虚拟环境
```bash
# 创建虚拟环境
python -m venv myenv

# 激活虚拟环境
# Windows
myenv\Scripts\activate
# macOS/Linux
source myenv/bin/activate

# 退出虚拟环境
deactivate
```

---

## 3. 基础语法

### 3.1 Hello World

```python
# 最简单的Python程序
print("Hello, World!")
```

### 3.2 注释

```python
# 这是单行注释

"""
这是多行注释
可以写多行内容
"""

'''
这也是多行注释
使用单引号
'''
```

### 3.3 变量和赋值

```python
# 变量赋值
name = "Alice"
age = 25
height = 1.68
is_student = True

# 多重赋值
x, y, z = 1, 2, 3

# 链式赋值
a = b = c = 0

# 交换变量
x, y = y, x
```

### 3.4 命名规范

```python
# 🔥 推荐的命名方式
user_name = "Alice"  # 变量：小写+下划线
MAX_SIZE = 100       # 常量：全大写+下划线

class UserProfile:   # 类名：大驼峰
    pass

def calculate_sum(): # 函数：小写+下划线
    pass

# ❌ 不推荐的命名
userName = "Alice"   # 避免小驼峰（除非是类属性）
MAXSIZE = 100        # 常量应该用下划线分隔
```

---

## 4. 数据类型

### 4.1 数值类型

#### 整数（int）
```python
# 整数
x = 10
y = -5
z = 0

# 大整数（Python3支持任意大小的整数）
big_num = 123456789012345678901234567890

# 不同进制
binary = 0b1010      # 二进制：10
octal = 0o12         # 八进制：10
hexadecimal = 0xa    # 十六进制：10
```

#### 浮点数（float）
```python
# 浮点数
pi = 3.14159
e = 2.71828

# 科学计数法
speed_of_light = 3e8  # 3 * 10^8
```

#### 复数（complex）
```python
# 复数
z = 3 + 4j
print(z.real)  # 实部：3.0
print(z.imag)  # 虚部：4.0
```

### 4.2 字符串（str）

```python
# 字符串定义
name = "Alice"
message = 'Hello'
multiline = """
这是多行字符串
可以包含多行内容
"""

# 🔥 字符串操作
# 拼接
full_name = "Alice" + " " + "Smith"

# 重复
repeat = "Ha" * 3  # "HaHaHa"

# 索引和切片
text = "Python"
print(text[0])      # 'P'
print(text[-1])     # 'n'
print(text[0:3])    # 'Pyt'
print(text[::-1])   # 'nohtyP' (反转)

# 字符串方法
text = "  Hello World  "
print(text.strip())       # 去除空格
print(text.lower())       # 转小写
print(text.upper())       # 转大写
print(text.replace("World", "Python"))  # 替换
print(text.split())       # 分割

# 🔥 字符串格式化
name = "Alice"
age = 25

# f-string (推荐)
message = f"My name is {name}, I'm {age} years old"

# format方法
message = "My name is {}, I'm {} years old".format(name, age)

# % 格式化（旧式）
message = "My name is %s, I'm %d years old" % (name, age)
```

### 4.3 布尔类型（bool）

```python
# 布尔值
is_active = True
is_deleted = False

# 布尔运算
print(True and False)  # False
print(True or False)   # True
print(not True)        # False

# 比较运算
print(5 > 3)   # True
print(5 == 5)  # True
print(5 != 3)  # True
```

### 4.4 类型转换

```python
# 🔥 类型转换
# 转整数
x = int("10")      # 10
y = int(3.14)      # 3

# 转浮点数
a = float("3.14")  # 3.14
b = float(10)      # 10.0

# 转字符串
s = str(123)       # "123"
t = str(3.14)      # "3.14"

# 转布尔值
print(bool(0))     # False
print(bool(1))     # True
print(bool(""))    # False
print(bool("Hi"))  # True
```

---

## 5. 控制流

### 5.1 条件语句

```python
# if语句
age = 18
if age >= 18:
    print("成年人")

# if-else语句
score = 85
if score >= 60:
    print("及格")
else:
    print("不及格")

# if-elif-else语句
score = 85
if score >= 90:
    print("优秀")
elif score >= 80:
    print("良好")
elif score >= 60:
    print("及格")
else:
    print("不及格")

# 🔥 三元表达式
result = "及格" if score >= 60 else "不及格"
```

### 5.2 循环语句

#### for循环
```python
# 遍历列表
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# 遍历字符串
for char in "Python":
    print(char)

# 🔥 range函数
for i in range(5):        # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):     # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2): # 0, 2, 4, 6, 8
    print(i)

# 🔥 enumerate（获取索引和值）
fruits = ["apple", "banana", "cherry"]
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
```

#### while循环
```python
# while循环
count = 0
while count < 5:
    print(count)
    count += 1

# 无限循环（需要break退出）
while True:
    user_input = input("输入'quit'退出: ")
    if user_input == "quit":
        break
    print(f"你输入了: {user_input}")
```

### 5.3 循环控制

```python
# break：跳出循环
for i in range(10):
    if i == 5:
        break
    print(i)  # 0, 1, 2, 3, 4

# continue：跳过本次循环
for i in range(5):
    if i == 2:
        continue
    print(i)  # 0, 1, 3, 4

# else：循环正常结束时执行
for i in range(5):
    print(i)
else:
    print("循环结束")
```

---

## 6. 函数

### 6.1 函数定义

```python
# 基本函数
def greet():
    """打招呼函数"""
    print("Hello!")

greet()  # 调用函数

# 带参数的函数
def greet_person(name):
    """
    向指定人打招呼
    
    Args:
        name: 人名
    """
    print(f"Hello, {name}!")

greet_person("Alice")

# 带返回值的函数
def add(a, b):
    """
    计算两数之和
    
    Args:
        a: 第一个数
        b: 第二个数
    
    Returns:
        两数之和
    """
    return a + b

result = add(3, 5)
print(result)  # 8
```

### 6.2 参数类型

```python
# 🔥 默认参数
def greet(name, message="Hello"):
    print(f"{message}, {name}!")

greet("Alice")              # Hello, Alice!
greet("Bob", "Hi")          # Hi, Bob!

# 🔥 关键字参数
def create_user(name, age, city):
    print(f"{name}, {age}, {city}")

create_user(name="Alice", age=25, city="Beijing")
create_user(age=30, name="Bob", city="Shanghai")

# 🔥 可变参数
def sum_all(*numbers):
    """接受任意数量的参数"""
    return sum(numbers)

print(sum_all(1, 2, 3))        # 6
print(sum_all(1, 2, 3, 4, 5))  # 15

# 🔥 关键字可变参数
def print_info(**kwargs):
    """接受任意数量的关键字参数"""
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=25, city="Beijing")
```

### 6.3 Lambda函数

```python
# lambda函数（匿名函数）
add = lambda x, y: x + y
print(add(3, 5))  # 8

# 常用于排序
students = [
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 20},
    {"name": "Charlie", "age": 23}
]
students.sort(key=lambda x: x["age"])
print(students)
```

---

## 7. 数据结构

### 7.1 列表（List）

```python
# 🔥 列表创建
fruits = ["apple", "banana", "cherry"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]

# 列表操作
fruits.append("orange")      # 添加元素
fruits.insert(1, "grape")    # 插入元素
fruits.remove("banana")      # 删除元素
popped = fruits.pop()        # 弹出最后一个元素
fruits.extend(["kiwi", "mango"])  # 扩展列表

# 列表索引和切片
print(fruits[0])       # 第一个元素
print(fruits[-1])      # 最后一个元素
print(fruits[1:3])     # 切片
print(fruits[::-1])    # 反转

# 🔥 列表推导式
squares = [x**2 for x in range(10)]
even_squares = [x**2 for x in range(10) if x % 2 == 0]
```

### 7.2 元组（Tuple）

```python
# 元组（不可变）
point = (3, 4)
person = ("Alice", 25, "Beijing")

# 元组解包
x, y = point
name, age, city = person

# 单元素元组
single = (1,)  # 注意逗号
```

### 7.3 字典（Dict）

```python
# 🔥 字典创建
person = {
    "name": "Alice",
    "age": 25,
    "city": "Beijing"
}

# 字典操作
print(person["name"])           # 访问
person["email"] = "alice@example.com"  # 添加
person["age"] = 26              # 修改
del person["city"]              # 删除

# 安全访问
print(person.get("phone", "N/A"))  # 不存在返回默认值

# 遍历字典
for key, value in person.items():
    print(f"{key}: {value}")

# 🔥 字典推导式
squares = {x: x**2 for x in range(5)}
```

### 7.4 集合（Set）

```python
# 集合（无序、不重复）
fruits = {"apple", "banana", "cherry"}

# 集合操作
fruits.add("orange")       # 添加
fruits.remove("banana")    # 删除
fruits.discard("grape")    # 安全删除（不存在不报错）

# 集合运算
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
print(a | b)  # 并集
print(a & b)  # 交集
print(a - b)  # 差集
print(a ^ b)  # 对称差集
```

---

## 8. 面向对象编程

### 8.1 类和对象

```python
# 🔥 定义类
class Person:
    """人类"""
    
    def __init__(self, name, age):
        """构造函数"""
        self.name = name
        self.age = age
    
    def greet(self):
        """打招呼方法"""
        print(f"Hello, I'm {self.name}, {self.age} years old")
    
    def birthday(self):
        """过生日"""
        self.age += 1

# 创建对象
person = Person("Alice", 25)
person.greet()
person.birthday()
print(person.age)  # 26
```

### 8.2 继承

```python
# 继承
class Student(Person):
    """学生类，继承Person"""
    
    def __init__(self, name, age, student_id):
        super().__init__(name, age)  # 调用父类构造函数
        self.student_id = student_id
    
    def study(self):
        """学习方法"""
        print(f"{self.name} is studying")

student = Student("Bob", 20, "S001")
student.greet()   # 继承的方法
student.study()   # 自己的方法
```

---

## 9. 模块和包

### 9.1 导入模块

```python
# 导入整个模块
import math
print(math.pi)

# 导入特定函数
from math import sqrt, pi
print(sqrt(16))

# 导入并重命名
import numpy as np
```

---

## 10. 文件操作

```python
# 🔥 读取文件
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()

# 写入文件
with open("file.txt", "w", encoding="utf-8") as f:
    f.write("Hello, World!")

# 追加文件
with open("file.txt", "a", encoding="utf-8") as f:
    f.write("\nNew line")
```

---

## 11. 异常处理

```python
# 🔥 异常处理
try:
    result = 10 / 0
except ZeroDivisionError:
    print("除数不能为0")
except Exception as e:
    print(f"发生错误: {e}")
finally:
    print("无论如何都会执行")
```

---

## 12. 最佳实践

### 12.1 代码规范

- 遵循PEP 8规范
- 使用有意义的变量名
- 添加注释和文档字符串
- 保持函数简洁

### 12.2 性能优化

- 使用列表推导式
- 避免不必要的循环
- 使用生成器处理大数据

---

## 📝 学习检查清单

- [ ] 能够独立编写Python程序
- [ ] 理解Python的数据类型和数据结构
- [ ] 掌握函数和面向对象编程
- [ ] 能够进行文件操作和异常处理
- [ ] 了解Python的最佳实践

---

## 🔗 相关资源

- [Python官方文档](https://docs.python.org/zh-cn/3/)
- [Python教程 - 廖雪峰](https://www.liaoxuefeng.com/wiki/1016959663602400)
- [Real Python](https://realpython.com/)

---

@author erik.zhou
