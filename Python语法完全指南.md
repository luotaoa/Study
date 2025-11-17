# Python语法完全指南 - 从入门到精通

## 🎯 目录

1. [基础语法](#part1-基础语法)
2. [数据类型](#part2-数据类型)
3. [运算符](#part3-运算符)
4. [控制流程](#part4-控制流程)
5. [函数](#part5-函数)
6. [面向对象](#part6-面向对象)
7. [常用内置函数](#part7-常用内置函数)
8. [文件操作](#part8-文件操作)
9. [异常处理](#part9-异常处理)
10. [模块和包](#part10-模块和包)

---

## Part1: 基础语法

### 1️⃣ 注释

```python
# 这是单行注释

"""
这是多行注释
可以写很多行
通常用于函数或类的文档说明
"""

'''
这也是多行注释
单引号和双引号都可以
'''
```

---

### 2️⃣ 变量和赋值

```python
# Python不需要声明类型，直接赋值即可
name = "张三"           # 字符串
age = 25               # 整数
height = 1.75          # 浮点数
is_student = True      # 布尔值

# 多个变量同时赋值
x, y, z = 1, 2, 3
print(x, y, z)  # 输出: 1 2 3

# 相同值赋给多个变量
a = b = c = 100
print(a, b, c)  # 输出: 100 100 100

# 交换变量（Python特有的简洁写法）
x, y = 10, 20
x, y = y, x  # 交换
print(x, y)  # 输出: 20 10
```

---

### 3️⃣ 变量命名规则

```python
# ✅ 正确的命名
my_name = "Alice"
age2 = 30
_private_var = 100
firstName = "Bob"  # 驼峰命名

# ❌ 错误的命名
# 2age = 30        # 不能以数字开头
# my-name = "Tom"  # 不能使用连字符
# class = "A"      # 不能使用关键字

# 命名规范（PEP 8）
snake_case_variable = 1    # 推荐：变量和函数用下划线
CONSTANT_VALUE = 100       # 推荐：常量用大写
ClassName = "Example"      # 推荐：类名用驼峰
```

---

### 4️⃣ 缩进（Python的灵魂！）

```python
# Python用缩进表示代码块，不是{}
# 推荐使用4个空格

if True:
    print("这是缩进的代码块")
    print("属于if语句内部")
print("这是外部代码")

# ❌ 错误的缩进
# if True:
# print("没有缩进会报错")

# 嵌套缩进
if True:
    print("第一层")
    if True:
        print("第二层")
        if True:
            print("第三层")
```

---

### 5️⃣ 打印输出

```python
# 基本打印
print("Hello, World!")

# 打印多个值（用逗号分隔，自动加空格）
print("我叫", "张三", "，今年", 25, "岁")
# 输出: 我叫 张三 ，今年 25 岁

# 自定义分隔符
print("A", "B", "C", sep="-")  # 输出: A-B-C

# 不换行打印
print("Hello", end=" ")
print("World")  # 输出: Hello World

# 格式化输出
name = "李四"
age = 30
print(f"我叫{name}，今年{age}岁")  # f-string（推荐）
print("我叫{}，今年{}岁".format(name, age))  # format方法
print("我叫%s，今年%d岁" % (name, age))  # 旧式格式化
```

---

## Part2: 数据类型

### 1️⃣ 数字类型

```python
# 整数 (int)
x = 10
y = -5
z = 0
big_num = 1_000_000  # 可以用下划线分隔，提高可读性

# 浮点数 (float)
pi = 3.14159
price = 99.99
scientific = 1.5e3  # 科学记数法，等于1500

# 复数 (complex)
c = 3 + 4j
print(c.real)  # 实部: 3.0
print(c.imag)  # 虚部: 4.0

# 类型转换
a = int("123")      # 字符串转整数
b = float("3.14")   # 字符串转浮点数
c = str(100)        # 数字转字符串
d = int(3.9)        # 浮点数转整数（截断）: 3
```

---

### 2️⃣ 字符串 (str)

```python
# 创建字符串
s1 = 'Hello'        # 单引号
s2 = "World"        # 双引号
s3 = '''多行
字符串'''            # 三引号
s4 = """也可以用
双引号三个"""

# 字符串拼接
first = "Hello"
last = "World"
full = first + " " + last  # Hello World

# 字符串重复
echo = "Ha" * 3  # HaHaHa

# 字符串索引（从0开始）
text = "Python"
print(text[0])    # P（第一个字符）
print(text[-1])   # n（最后一个字符）
print(text[-2])   # o（倒数第二个）

# 字符串切片 [start:end:step]
s = "Hello, World!"
print(s[0:5])     # Hello（从0到5，不包括5）
print(s[7:])      # World!（从7到结尾）
print(s[:5])      # Hello（从开头到5）
print(s[::2])     # Hlo ol!（每隔一个取一个）
print(s[::-1])    # !dlroW ,olleH（反转字符串）

# 字符串方法
text = "  Hello, Python!  "
print(text.lower())         # 小写: hello, python!
print(text.upper())         # 大写: HELLO, PYTHON!
print(text.strip())         # 去除两端空格
print(text.replace("Python", "World"))  # 替换
print(text.split(","))      # 分割成列表
print("Python" in text)     # 检查是否包含: True
print(text.startswith(" "))  # 是否以空格开头: True
print(text.endswith("!  ")) # 是否以!结尾: True
print(len(text))            # 长度: 19

# 字符串格式化
name = "Alice"
age = 25
score = 95.5

# 方法1: f-string（Python 3.6+，推荐）
print(f"姓名: {name}, 年龄: {age}, 分数: {score:.1f}")

# 方法2: format方法
print("姓名: {}, 年龄: {}, 分数: {:.1f}".format(name, age, score))

# 方法3: 旧式%格式化
print("姓名: %s, 年龄: %d, 分数: %.1f" % (name, age, score))
```

---

### 3️⃣ 列表 (list) - 可变序列

```python
# 创建列表
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]  # 可以混合类型
empty = []  # 空列表

# 访问元素
fruits = ["apple", "banana", "orange"]
print(fruits[0])    # apple
print(fruits[-1])   # orange（最后一个）

# 切片
print(fruits[0:2])  # ['apple', 'banana']
print(fruits[:2])   # ['apple', 'banana']
print(fruits[1:])   # ['banana', 'orange']

# 修改元素
fruits[1] = "grape"
print(fruits)  # ['apple', 'grape', 'orange']

# 添加元素
fruits.append("mango")         # 末尾添加
fruits.insert(1, "banana")     # 指定位置插入
fruits.extend(["kiwi", "pear"])  # 添加多个元素

# 删除元素
fruits.remove("banana")  # 删除指定值
deleted = fruits.pop()   # 删除并返回最后一个
del fruits[0]            # 删除指定索引
fruits.clear()           # 清空列表

# 列表操作
nums = [3, 1, 4, 1, 5, 9, 2]
print(len(nums))        # 长度: 7
print(max(nums))        # 最大值: 9
print(min(nums))        # 最小值: 1
print(sum(nums))        # 求和: 25
print(nums.count(1))    # 统计1出现次数: 2
print(nums.index(4))    # 找到4的索引: 2

nums.sort()             # 排序（改变原列表）
print(nums)             # [1, 1, 2, 3, 4, 5, 9]

nums.reverse()          # 反转
print(nums)             # [9, 5, 4, 3, 2, 1, 1]

# 列表推导式（简洁创建列表）TODO:
squares = [x**2 for x in range(1, 6)]
print(squares)  # [1, 4, 9, 16, 25]

evens = [x for x in range(10) if x % 2 == 0]
print(evens)  # [0, 2, 4, 6, 8]
```

---

### 4️⃣ 元组 (tuple) - 不可变序列

```python
# 创建元组（用小括号或不用括号）
point = (3, 4)
colors = "red", "green", "blue"  # 自动识别为元组
single = (42,)  # 单元素元组必须加逗号
empty = ()      # 空元组

# 访问元素（和列表一样）
print(point[0])   # 3
print(colors[-1]) # blue

# 元组不可修改
# point[0] = 10  # ❌ 报错：不支持修改

# 元组解包
x, y = point
print(x, y)  # 3 4

a, b, c = colors
print(a, b, c)  # red green blue

# 元组的用途
def get_coordinates():
    return 10, 20  # 返回多个值（实际是元组）

x, y = get_coordinates()
print(x, y)  # 10 20
```

---

### 5️⃣ 字典 (dict) - 键值对

```python
# 创建字典
person = {
    "name": "Alice",
    "age": 25,
    "city": "Beijing"
}

# 另一种创建方式
person2 = dict(name="Bob", age=30)

# 访问值
print(person["name"])         # Alice
print(person.get("age"))      # 25
print(person.get("salary", 0))  # 不存在返回默认值0

# 修改/添加
person["age"] = 26           # 修改
person["job"] = "Engineer"   # 添加新键值对

# 删除
del person["city"]           # 删除指定键
removed = person.pop("job")  # 删除并返回值
person.clear()               # 清空字典

# 字典操作
student = {"name": "Tom", "age": 20, "score": 95}

print(student.keys())        # 所有键
print(student.values())      # 所有值
print(student.items())       # 所有键值对

# 遍历字典
for key in student:
    print(key, student[key])

for key, value in student.items():
    print(f"{key}: {value}")

# 检查键是否存在
if "name" in student:
    print("name键存在")

# 字典推导式
squares = {x: x**2 for x in range(1, 6)}
print(squares)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

---

### 6️⃣ 集合 (set) - 无序不重复

```python
# 创建集合
fruits = {"apple", "banana", "orange"}
numbers = set([1, 2, 3, 3, 4, 4])  # 自动去重: {1, 2, 3, 4}
empty = set()  # 空集合（注意：{}是空字典）

# 添加元素
fruits.add("mango")
fruits.update(["kiwi", "pear"])  # 添加多个

# 删除元素
fruits.remove("banana")  # 不存在会报错
fruits.discard("xxx")    # 不存在不报错
deleted = fruits.pop()   # 随机删除一个

# 集合运算
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

print(a | b)   # 并集: {1, 2, 3, 4, 5, 6}
print(a & b)   # 交集: {3, 4}
print(a - b)   # 差集: {1, 2}
print(a ^ b)   # 对称差: {1, 2, 5, 6}

# 集合推导式
evens = {x for x in range(10) if x % 2 == 0}
print(evens)  # {0, 2, 4, 6, 8}
```

---

## Part3: 运算符

### 1️⃣ 算术运算符

```python
a, b = 10, 3

print(a + b)   # 加法: 13
print(a - b)   # 减法: 7
print(a * b)   # 乘法: 30
print(a / b)   # 除法: 3.3333...
print(a // b)  # 整除: 3
print(a % b)   # 取余: 1
print(a ** b)  # 幂运算: 1000（10的3次方）

# 复合赋值
x = 5
x += 3   # x = x + 3, 结果: 8
x -= 2   # x = x - 2, 结果: 6
x *= 4   # x = x * 4, 结果: 24
x //= 5  # x = x // 5, 结果: 4
```

---

### 2️⃣ 比较运算符

```python
a, b = 10, 20

print(a == b)   # 等于: False
print(a != b)   # 不等于: True
print(a > b)    # 大于: False
print(a < b)    # 小于: True
print(a >= b)   # 大于等于: False
print(a <= b)   # 小于等于: True

# 链式比较（Python特有）
x = 15
print(10 < x < 20)  # True（相当于 10 < x and x < 20）
```

---

### 3️⃣ 逻辑运算符

```python
# and（与）、or（或）、not（非）
a, b = True, False

print(a and b)   # False（都为True才为True）
print(a or b)    # True（有一个True就为True）
print(not a)     # False（取反）

# 实际应用
age = 25
has_license = True

if age >= 18 and has_license:
    print("可以开车")

# 短路求值
x = 0
# y = 10 / x  # 会报错
y = x != 0 and 10 / x  # 不会报错，因为x != 0为False就不会执行后面
```

---

### 4️⃣ 成员运算符

```python
# in、not in
fruits = ["apple", "banana", "orange"]

print("apple" in fruits)      # True
print("grape" not in fruits)  # True

# 字符串中也可以用
text = "Hello, World!"
print("Hello" in text)        # True
print("Python" not in text)   # True
```

---

### 5️⃣ 身份运算符

```python
# is、is not（判断是否是同一个对象）
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)   # True（值相同）
print(a is b)   # False（不是同一个对象）
print(a is c)   # True（是同一个对象）

# None的判断
x = None
print(x is None)      # True（推荐）
print(x == None)      # True（不推荐）
```

---

## Part4: 控制流程

### 1️⃣ if条件语句

```python
# 基本if
age = 18
if age >= 18:
    print("成年人")

# if-else
score = 75
if score >= 60:
    print("及格")
else:
    print("不及格")

# if-elif-else
score = 85
if score >= 90:
    print("优秀")
elif score >= 80:
    print("良好")
elif score >= 60:
    print("及格")
else:
    print("不及格")

# 三元表达式（简化的if-else）
age = 20
status = "成年" if age >= 18 else "未成年"
print(status)  # 成年
```

---

### 2️⃣ for循环

```python
# 遍历列表
fruits = ["apple", "banana", "orange"]
for fruit in fruits:
    print(fruit)

# 遍历字符串
for char in "Python":
    print(char)

# range()函数
for i in range(5):           # 0到4
    print(i)

for i in range(1, 6):        # 1到5
    print(i)

for i in range(0, 10, 2):    # 0到9，步长2
    print(i)  # 0, 2, 4, 6, 8

# enumerate()（获取索引和值）
fruits = ["apple", "banana", "orange"]
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
# 0: apple
# 1: banana
# 2: orange

# zip()（同时遍历多个列表）
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name}今年{age}岁")

# 列表推导式（简洁创建列表）
squares = [x**2 for x in range(1, 6)]
print(squares)  # [1, 4, 9, 16, 25]

# 带条件的列表推导式
evens = [x for x in range(10) if x % 2 == 0]
print(evens)  # [0, 2, 4, 6, 8]
```

---

### 3️⃣ while循环

```python
# 基本while
count = 0
while count < 5:
    print(count)
    count += 1

# 无限循环（需要break跳出）
while True:
    user_input = input("输入'quit'退出: ")
    if user_input == "quit":
        break
    print(f"你输入了: {user_input}")

# 猜数字游戏
import random
secret = random.randint(1, 100)
guess_count = 0

while True:
    guess = int(input("猜一个1-100的数字: "))
    guess_count += 1
    
    if guess < secret:
        print("太小了")
    elif guess > secret:
        print("太大了")
    else:
        print(f"恭喜！猜对了！你猜了{guess_count}次")
        break

while TRUE:
    gue = int(input('请输入数字'))
    if gue>10:
			print("太大了")
		else if gue<10:
			print('太小了')
		if gue==10:
			print('刚好')
```

---

### 4️⃣ break、continue、pass

```python
# break：跳出循环
for i in range(10):
    if i == 5:
        break  # 遇到5就停止
    print(i)  # 输出: 0 1 2 3 4

# continue：跳过本次循环
for i in range(10):
    if i % 2 == 0:
        continue  # 跳过偶数
    print(i)  # 输出: 1 3 5 7 9

# pass：占位符（什么都不做）
for i in range(5):
    if i == 2:
        pass  # 暂时不写代码，但不报错
    else:
        print(i)

# pass的实际用途
def my_function():
    pass  # 函数体暂时为空，先占位

class MyClass:
    pass  # 类暂时为空
```

---

### 5️⃣ else子句（特殊用法）

```python
# for-else（循环正常结束时执行）
for i in range(5):
    print(i)
else:
    print("循环正常结束")  # 会执行

# 如果用break跳出，else不会执行
for i in range(5):
    if i == 3:
        break
    print(i)
else:
    print("循环正常结束")  # 不会执行

# while-else
count = 0
while count < 3:
    print(count)
    count += 1
else:
    print("while循环结束")  # 会执行
```

---

## Part5: 函数

### 1️⃣ 定义和调用函数

```python
# 基本函数
def greet():
    print("Hello!")

greet()  # 调用函数

# 带参数的函数
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")  # Hello, Alice!

# 多个参数
def add(a, b):
    return a + b

result = add(3, 5)
print(result)  # 8

# 默认参数
def greet(name, greeting="Hello"):
    print(f"{greeting}, {name}!")

greet("Bob")                    # Hello, Bob!
greet("Alice", "Hi")           # Hi, Alice!
greet("Tom", greeting="Hey")   # Hey, Tom!（关键字参数）

# 返回多个值（实际是元组）
def get_info():
    name = "Alice"
    age = 25
    return name, age

name, age = get_info()
print(name, age)  # Alice 25
```

---

### 2️⃣ 参数类型

```python
# 位置参数
def power(base, exponent):
    return base ** exponent

print(power(2, 3))  # 8

# 关键字参数
print(power(exponent=3, base=2))  # 8（顺序无关）

# 默认参数
def greet(name, msg="Good morning"):
    print(f"{msg}, {name}")

greet("Alice")              # Good morning, Alice
greet("Bob", "Good night")  # Good night, Bob

# 可变参数 *args（接收任意数量的位置参数）
def sum_all(*numbers):
    total = 0
    for num in numbers:
        total += num
    return total

print(sum_all(1, 2, 3))        # 6
print(sum_all(1, 2, 3, 4, 5))  # 15

# 关键字可变参数 **kwargs（接收任意数量的关键字参数）
def print_info(**info):
    for key, value in info.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=25, city="Beijing")
# name: Alice
# age: 25
# city: Beijing

# 混合使用
def example(a, b, *args, **kwargs):
    print(f"a={a}, b={b}")
    print(f"args={args}")
    print(f"kwargs={kwargs}")

example(1, 2, 3, 4, 5, x=10, y=20)
# a=1, b=2
# args=(3, 4, 5)
# kwargs={'x': 10, 'y': 20}
```

---

### 3️⃣ 匿名函数 (lambda)

```python
# 普通函数
def square(x):
    return x ** 2

print(square(5))  # 25

# lambda表达式（匿名函数）
square = lambda x: x ** 2
print(square(5))  # 25

# 多个参数
add = lambda a, b: a + b
print(add(3, 5))  # 8

# 常用于高阶函数
numbers = [1, 2, 3, 4, 5]

# map()：对每个元素应用函数
squared = list(map(lambda x: x**2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# filter()：过滤元素
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4]

# sorted()：自定义排序
students = [("Alice", 85), ("Bob", 75), ("Charlie", 95)]
sorted_students = sorted(students, key=lambda x: x[1])
print(sorted_students)
# [('Bob', 75), ('Alice', 85), ('Charlie', 95)]
```

---

### 4️⃣ 作用域

```python
# 全局变量
x = 10

def func():
    # 局部变量
    y = 20
    print(x)  # 可以访问全局变量: 10
    print(y)  # 可以访问局部变量: 20

func()
# print(y)  # ❌ 报错：局部变量在外部不可访问

# 修改全局变量
count = 0

def increment():
    global count  # 声明使用全局变量
    count += 1

increment()
print(count)  # 1

# nonlocal（嵌套函数中修改外层变量）
def outer():
    x = 10
    
    def inner():
        nonlocal x  # 修改外层函数的变量
        x += 5
    
    inner()
    print(x)  # 15

outer()
```

---

### 5️⃣ 文档字符串 (docstring)

```python
def calculate_area(radius):
    """
    计算圆的面积
    
    参数:
        radius (float): 圆的半径
    
    返回:
        float: 圆的面积
    """
    return 3.14159 * radius ** 2

# 查看文档
print(calculate_area.__doc__)
help(calculate_area)
```

---

## Part6: 面向对象

### 1️⃣ 类和对象

```python
# 定义类
class Dog:
    # 类属性（所有实例共享）
    species = "Canis familiaris"
    
    # 初始化方法（构造函数）
    def __init__(self, name, age):
        # 实例属性
        self.name = name
        self.age = age
    
    # 实例方法
    def bark(self):
        print(f"{self.name}在叫: 汪汪汪!")
    
    def get_info(self):
        return f"{self.name}今年{self.age}岁"

# 创建对象
dog1 = Dog("小白", 3)
dog2 = Dog("小黑", 5)

# 访问属性
print(dog1.name)      # 小白
print(dog1.age)       # 3
print(Dog.species)    # Canis familiaris

# 调用方法
dog1.bark()           # 小白在叫: 汪汪汪!
print(dog1.get_info())  # 小白今年3岁
```

---

### 2️⃣ 继承

```python
# 父类
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        pass  # 抽象方法，由子类实现

# 子类
class Dog(Animal):
    def speak(self):
        return f"{self.name}说: 汪汪!"

class Cat(Animal):
    def speak(self):
        return f"{self.name}说: 喵喵!"

# 使用
dog = Dog("小白")
cat = Cat("小花")

print(dog.speak())  # 小白说: 汪汪!
print(cat.speak())  # 小花说: 喵喵!

# 多继承
class Flyable:
    def fly(self):
        print("我会飞!")

class Swimmable:
    def swim(self):
        print("我会游泳!")

class Duck(Animal, Flyable, Swimmable):
    def speak(self):
        return f"{self.name}说: 嘎嘎!"

duck = Duck("唐老鸭")
print(duck.speak())  # 唐老鸭说: 嘎嘎!
duck.fly()           # 我会飞!
duck.swim()          # 我会游泳!
```

---

### 3️⃣ 封装

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.__balance = balance  # 私有属性（双下划线）
    
    def deposit(self, amount):
        """存款"""
        if amount > 0:
            self.__balance += amount
            print(f"存入{amount}元，余额{self.__balance}元")
        else:
            print("存款金额必须大于0")
    
    def withdraw(self, amount):
        """取款"""
        if amount > self.__balance:
            print("余额不足")
        else:
            self.__balance -= amount
            print(f"取出{amount}元，余额{self.__balance}元")
    
    def get_balance(self):
        """查询余额"""
        return self.__balance

# 使用
account = BankAccount("Alice", 1000)
account.deposit(500)        # 存入500元，余额1500元
account.withdraw(200)       # 取出200元，余额1300元
print(account.get_balance())  # 1300

# print(account.__balance)  # ❌ 无法直接访问私有属性
```

---

### 4️⃣ 特殊方法（魔法方法）

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # 字符串表示
    def __str__(self):
        return f"Point({self.x}, {self.y})"
    
    def __repr__(self):
        return f"Point({self.x}, {self.y})"
    
    # 运算符重载
    def __add__(self, other):
        return Point(self.x + other.x, self.y + other.y)
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    
    # 长度
    def __len__(self):
        return 2

p1 = Point(1, 2)
p2 = Point(3, 4)

print(p1)            # Point(1, 2)
print(p1 + p2)       # Point(4, 6)
print(p1 == p2)      # False
print(len(p1))       # 2
```

---

### 5️⃣ 类方法和静态方法

```python
class MathUtils:
    pi = 3.14159
    
    # 实例方法
    def instance_method(self):
        print("这是实例方法")
    
    # 类方法（第一个参数是类本身）
    @classmethod
    def class_method(cls):
        print(f"这是类方法，pi={cls.pi}")
    
    # 静态方法（不需要self或cls）
    @staticmethod
    def static_method(x, y):
        return x + y

# 调用
obj = MathUtils()
obj.instance_method()        # 这是实例方法

MathUtils.class_method()     # 这是类方法，pi=3.14159

result = MathUtils.static_method(3, 5)
print(result)                # 8
```

---

## Part7: 常用内置函数

```python
# 数学函数
print(abs(-10))         # 绝对值: 10
print(pow(2, 3))        # 幂: 8
print(round(3.14159, 2))  # 四舍五入: 3.14
print(max(1, 5, 3))     # 最大值: 5
print(min(1, 5, 3))     # 最小值: 1
print(sum([1, 2, 3, 4]))  # 求和: 10

# 类型转换
print(int("123"))       # 字符串转整数: 123
print(float("3.14"))    # 字符串转浮点数: 3.14
print(str(100))         # 数字转字符串: "100"
print(list("abc"))      # 字符串转列表: ['a', 'b', 'c']
print(tuple([1, 2, 3]))  # 列表转元组: (1, 2, 3)
print(set([1, 2, 2, 3]))  # 列表转集合: {1, 2, 3}

# 序列函数
numbers = [3, 1, 4, 1, 5]
print(len(numbers))     # 长度: 5
print(sorted(numbers))  # 排序: [1, 1, 3, 4, 5]
print(reversed(numbers))  # 反转（返回迭代器）

# range()
print(list(range(5)))        # [0, 1, 2, 3, 4]
print(list(range(1, 6)))     # [1, 2, 3, 4, 5]
print(list(range(0, 10, 2))) # [0, 2, 4, 6, 8]

# enumerate()
fruits = ["apple", "banana", "orange"]
for i, fruit in enumerate(fruits):
    print(i, fruit)

# zip()
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name}: {age}")

# map()
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# filter()
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4]

# any() 和 all()
print(any([False, False, True]))   # True（有一个True）
print(all([True, True, True]))     # True（全部True）
print(all([True, False, True]))    # False（有一个False）

# input()
name = input("请输入你的名字: ")
print(f"你好，{name}!")
```

---

## Part8: 文件操作

### 1️⃣ 读取文件

```python
# 方法1：手动关闭
file = open("example.txt", "r", encoding="utf-8")
content = file.read()  # 读取全部内容
print(content)
file.close()  # 必须关闭

# 方法2：with语句（推荐，自动关闭）
with open("example.txt", "r", encoding="utf-8") as file:
    content = file.read()
    print(content)
# 文件自动关闭

# 逐行读取
with open("example.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line.strip())  # strip()去除换行符

# 读取所有行到列表
with open("example.txt", "r", encoding="utf-8") as file:
    lines = file.readlines()
    print(lines)
```

---

### 2️⃣ 写入文件

```python
# 写入（覆盖）
with open("output.txt", "w", encoding="utf-8") as file:
    file.write("Hello, World!\n")
    file.write("第二行内容\n")

# 追加
with open("output.txt", "a", encoding="utf-8") as file:
    file.write("追加的内容\n")

# 写入多行
lines = ["第一行\n", "第二行\n", "第三行\n"]
with open("output.txt", "w", encoding="utf-8") as file:
    file.writelines(lines)
```

---

### 3️⃣ 文件模式

```python
# "r"  - 只读（默认）
# "w"  - 写入（覆盖）
# "a"  - 追加
# "r+" - 读写
# "rb" - 二进制读
# "wb" - 二进制写

# 示例：复制文件
with open("source.txt", "r", encoding="utf-8") as src:
    with open("dest.txt", "w", encoding="utf-8") as dst:
        dst.write(src.read())
```

---

## Part9: 异常处理

### 1️⃣ try-except

```python
# 基本异常处理
try:
    result = 10 / 0  # 会引发异常
except ZeroDivisionError:
    print("除数不能为0")

# 捕获多种异常
try:
    x = int("abc")  # ValueError
except ValueError:
    print("值错误")
except ZeroDivisionError:
    print("除数不能为0")

# 捕获所有异常
try:
    x = 10 / 0
except Exception as e:
    print(f"发生错误: {e}")

# try-except-else-finally
try:
    result = 10 / 2
except ZeroDivisionError:
    print("除数不能为0")
else:
    print(f"结果是{result}")  # 没有异常时执行
finally:
    print("无论如何都会执行")  # 总是执行
```

---

### 2️⃣ 抛出异常

```python
# 手动抛出异常
def check_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数")
    elif age > 150:
        raise ValueError("年龄不合理")
    else:
        print(f"年龄: {age}")

try:
    check_age(-5)
except ValueError as e:
    print(f"错误: {e}")

# 自定义异常
class MyError(Exception):
    pass

def my_function():
    raise MyError("这是自定义异常")

try:
    my_function()
except MyError as e:
    print(f"捕获到: {e}")
```

---

## Part10: 模块和包

### 1️⃣ 导入模块

```python
# 导入整个模块
import math
print(math.pi)        # 3.14159...
print(math.sqrt(16))  # 4.0

# 导入特定函数
from math import pi, sqrt
print(pi)             # 3.14159...
print(sqrt(16))       # 4.0

# 导入所有（不推荐）
from math import *
print(pi)

# 重命名
import numpy as np
import pandas as pd

# 导入自己的模块
# 假设有文件 mymodule.py
# import mymodule
# mymodule.my_function()
```

---

### 2️⃣ 常用标准库

```python
# datetime - 日期时间
from datetime import datetime, timedelta

now = datetime.now()
print(now)  # 2024-01-15 10:30:00

tomorrow = now + timedelta(days=1)
print(tomorrow)

# random - 随机数
import random

print(random.randint(1, 10))     # 1-10的随机整数
print(random.random())           # 0-1的随机浮点数
print(random.choice([1, 2, 3]))  # 随机选择一个

# os - 操作系统
import os

print(os.getcwd())           # 当前工作目录
print(os.listdir())          # 列出文件
os.mkdir("new_folder")       # 创建文件夹
os.remove("file.txt")        # 删除文件

# json - JSON处理
import json

data = {"name": "Alice", "age": 25}
json_str = json.dumps(data)  # 字典转JSON字符串
print(json_str)

data2 = json.loads(json_str)  # JSON字符串转字典
print(data2)
```

---

### 3️⃣ 第三方库（需要安装）

```bash
# 使用pip安装
pip install numpy
pip install pandas
pip install matplotlib
pip install requests
```

```python
# numpy - 数值计算
import numpy as np

arr = np.array([1, 2, 3, 4, 5])
print(arr * 2)  # [2 4 6 8 10]

# pandas - 数据分析
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35]
})
print(df)

# matplotlib - 数据可视化
import matplotlib.pyplot as plt

x = [1, 2, 3, 4, 5]
y = [2, 4, 6, 8, 10]
plt.plot(x, y)
plt.show()

# requests - HTTP请求
import requests

response = requests.get("https://api.github.com")
print(response.status_code)
print(response.json())
```

---

## 🎓 总结

### Python核心特点

```
1. 简洁易读：用缩进表示代码块
2. 动态类型：不需要声明变量类型
3. 强大的数据结构：list、dict、set、tuple
4. 面向对象：类和继承
5. 丰富的库：标准库 + 第三方库
```

### 学习路径

```
1. 基础语法 → 数据类型 → 控制流程
2. 函数 → 面向对象
3. 文件操作 → 异常处理
4. 模块和包 → 第三方库
5. 实战项目
```

### 常用快速参考

```python
# 列表推导式
[x**2 for x in range(10) if x % 2 == 0]

# 字典推导式
{x: x**2 for x in range(5)}

# lambda表达式
lambda x: x**2

# with语句
with open("file.txt") as f:
    content = f.read()

# 解包
a, b = (1, 2)
*rest, last = [1, 2, 3, 4]

# 三元表达式
result = "成年" if age >= 18 else "未成年"
```

---

**恭喜！你已经掌握了Python的核心语法！接下来多写代码，多实践！** 🚀
