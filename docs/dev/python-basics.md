# Python 基础速查

!!! tip "适用版本"
    本笔记基于 Python 3.10+，有 C 语言基础，重点记录与 C 不同的地方。
    参考：[廖雪峰 Python 教程](https://liaoxuefeng.com/books/python)

---

## 一、数据类型

=== "列表 List"
    ```python
    fruits = ["apple", "banana", "cherry"]
    fruits.append("mango")        # 追加末尾
    fruits.insert(1, "grape")     # 插入指定位置
    fruits.pop()                  # 删除末尾
    fruits.remove("banana")       # 删除指定值
    fruits[0]                     # 索引
    fruits[-1]                    # 倒数第一个
    fruits[1:3]                   # 切片 [1, 3)
    fruits[::-1]                  # 反转
    len(fruits)                   # 长度

    # 排序
    fruits.sort()                 # 原地排序
    sorted(fruits, reverse=True)  # 返回新列表
    ```

=== "字典 Dict"
    ```python
    person = {"name": "Alice", "age": 26}
    person["name"]                    # 取值，不存在报 KeyError
    person.get("email", "unknown")    # 安全取值，不存在返回默认值
    person["city"] = "Wuhan"          # 新增/更新
    del person["age"]                 # 删除
    "name" in person                  # 判断 key 是否存在

    # 遍历
    for k, v in person.items():
        print(f"{k}: {v}")

    # 合并（Python 3.9+）
    merged = {**dict1, **dict2}
    merged = dict1 | dict2            # 更简洁
    ```

=== "集合 Set"
    ```python
    tags = {"python", "ai", "code"}
    tags.add("agent")
    tags.discard("ai")               # 删除，不存在不报错
    "python" in tags                 # 成员判断 O(1)

    a = {1, 2, 3}
    b = {2, 3, 4}
    a & b   # 交集 {2, 3}
    a | b   # 并集 {1, 2, 3, 4}
    a - b   # 差集 {1}
    ```

=== "元组 Tuple"
    ```python
    # 不可变序列，比 list 快，常用于多返回值
    point = (3, 4)
    x, y = point           # 解包

    # 命名元组（比普通 tuple 可读性强）
    from collections import namedtuple
    Point = namedtuple("Point", ["x", "y"])
    p = Point(3, 4)
    print(p.x, p.y)
    ```

---

## 二、推导式与生成器

```python
# 列表推导式（比 map/filter 更 Pythonic）
squares = [x**2 for x in range(10)]
evens   = [x for x in range(20) if x % 2 == 0]

# 字典推导式
word_len = {word: len(word) for word in ["apple", "banana"]}

# 集合推导式
unique_lens = {len(w) for w in ["apple", "banana", "fig"]}

# 生成器表达式（不立即计算，省内存）
gen = (x**2 for x in range(1000000))  # 不占内存
total = sum(x**2 for x in range(100)) # 直接传入函数

# 生成器函数
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci()
print([next(fib) for _ in range(8)])  # [0, 1, 1, 2, 3, 5, 8, 13]
```

---

## 三、函数

```python
# 类型注解（Python 3.10+ 写法）
def greet(name: str, *, formal: bool = False) -> str:
    prefix = "Dear" if formal else "Hey"
    return f"{prefix}, {name}!"

# * 后面的参数只能用关键字传入
greet("Alice", formal=True)

# 可变参数
def func(*args, **kwargs):
    print(args)    # tuple
    print(kwargs)  # dict

# Lambda（适合简单单行函数）
double = lambda x: x * 2
sorted(items, key=lambda x: x["price"])

# 解包传参
params = {"name": "Alice", "formal": True}
greet(**params)
```

### 装饰器

```python
import functools
import time

# 计时装饰器
def timer(func):
    @functools.wraps(func)   # 保留原函数的 __name__ 等信息
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} 耗时 {time.time()-start:.3f}s")
        return result
    return wrapper

# 带参数的装饰器
def retry(times=3, delay=1):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if i == times - 1:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(times=3, delay=2)
def call_api():
    ...
```

---

## 四、面向对象

```python
class Animal:
    # 类变量（所有实例共享）
    count = 0

    def __init__(self, name: str, age: int):
        # 实例变量
        self.name = name
        self.age  = age
        Animal.count += 1

    def speak(self) -> str:
        raise NotImplementedError  # 类似抽象方法

    def __repr__(self) -> str:
        return f"Animal(name={self.name!r}, age={self.age})"

    def __str__(self) -> str:
        return self.name

class Dog(Animal):
    def __init__(self, name: str, age: int, breed: str):
        super().__init__(name, age)  # 调用父类 __init__
        self.breed = breed

    def speak(self) -> str:
        return f"{self.name}: Woof!"

    @classmethod
    def from_dict(cls, data: dict) -> "Dog":
        """类方法：替代构造器"""
        return cls(data["name"], data["age"], data["breed"])

    @staticmethod
    def is_valid_age(age: int) -> bool:
        """静态方法：不依赖实例或类"""
        return 0 <= age <= 20

    @property
    def info(self) -> str:
        """属性装饰器：当作属性访问，不加括号"""
        return f"{self.name} ({self.breed})"

dog = Dog("Buddy", 3, "Labrador")
print(dog.info)          # Buddy (Labrador)
print(Dog.is_valid_age(3))  # True
```

### dataclass（推荐替代手写 __init__）

```python
from dataclasses import dataclass, field

@dataclass
class ExpenseRecord:
    id: int
    title: str
    amount: float
    tags: list[str] = field(default_factory=list)  # 可变默认值必须用 field
    status: str = "pending"

    def is_approved(self) -> bool:
        return self.status == "approved"

# 自动生成 __init__、__repr__、__eq__
record = ExpenseRecord(id=1, title="高铁票", amount=580.0)
print(record)  # ExpenseRecord(id=1, title='高铁票', ...)
```

---

## 五、异常处理

```python
# 基础结构
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"除零错误: {e}")
except (TypeError, ValueError) as e:
    print(f"类型或值错误: {e}")
except Exception as e:
    print(f"未知错误: {e}")
    raise                   # 重新抛出
else:
    print("没有异常才执行")
finally:
    print("无论如何都执行（适合关闭连接/文件）")

# 自定义异常
class AppError(Exception):
    def __init__(self, message: str, code: int = 500):
        super().__init__(message)
        self.code = code

raise AppError("报销单不存在", code=404)

# 上下文管理器（自动资源释放）
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()
# 退出 with 块后自动关闭文件，即使发生异常
```

---

## 六、文件与路径

```python
from pathlib import Path

# 推荐用 pathlib，比 os.path 更直观
p = Path("/Users/yang_yinmiao/Documents")
p / "my-notes" / "docs"    # 路径拼接
p.exists()                  # 是否存在
p.is_file()
p.is_dir()
p.suffix                    # '.md'
p.stem                      # 文件名不含扩展名
p.parent                    # 父目录

# 读写文件
Path("output.txt").write_text("hello", encoding="utf-8")
content = Path("output.txt").read_text(encoding="utf-8")

# 遍历目录
for md_file in Path("docs").rglob("*.md"):
    print(md_file)

# JSON
import json
data = {"name": "Alice", "age": 26}
json_str = json.dumps(data, ensure_ascii=False, indent=2)
parsed  = json.loads(json_str)

Path("data.json").write_text(json_str, encoding="utf-8")
```

---

## 七、异步编程（async/await）

Agent 开发中大量使用异步，重点掌握。

```python
import asyncio
import httpx

# 基本语法
async def fetch(url: str) -> str:
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.text

# 并发执行多个任务（比串行快 N 倍）
async def main():
    urls = ["https://api.example.com/1", "https://api.example.com/2"]

    # 并发执行
    results = await asyncio.gather(*[fetch(url) for url in urls])

    # 带超时
    try:
        result = await asyncio.wait_for(fetch(urls[0]), timeout=5.0)
    except asyncio.TimeoutError:
        print("请求超时")

asyncio.run(main())

# 异步生成器（流式输出常用）
async def stream_response():
    async for chunk in llm.astream("解释一下 RAG"):
        yield chunk

async def print_stream():
    async for chunk in stream_response():
        print(chunk, end="", flush=True)
```

**同步 vs 异步选择：**
- 脚本/一次性任务 → 同步够用
- Web 服务（FastAPI）/ 并发调用 API / Agent 多工具并行 → 必须用异步

---

## 八、类型注解

```python
from typing import Optional, Union, Any
from collections.abc import Callable, Generator

# 基础
name: str = "Alice"
age:  int = 26
data: dict[str, Any] = {}

# Optional = 可以是 None
def find_user(user_id: int) -> Optional[dict]:
    ...

# Union（Python 3.10+ 可用 |）
def process(value: int | str) -> str:
    return str(value)

# 泛型容器
def get_names(users: list[dict[str, Any]]) -> list[str]:
    return [u["name"] for u in users]

# Callable
def apply(func: Callable[[int], int], value: int) -> int:
    return func(value)

# TypedDict（比 dict 更严格）
from typing import TypedDict

class UserInfo(TypedDict):
    name: str
    age:  int
    email: str | None
```

---

## 九、常用标准库速查

| 库 | 常用功能 | 示例 |
|----|---------|------|
| `pathlib` | 文件路径 | `Path("docs").glob("*.md")` |
| `json` | JSON 读写 | `json.dumps(data, ensure_ascii=False)` |
| `datetime` | 日期时间 | `datetime.now().strftime("%Y-%m-%d")` |
| `re` | 正则表达式 | `re.findall(r"\d+", text)` |
| `collections` | 特殊容器 | `Counter`, `defaultdict`, `deque` |
| `itertools` | 迭代工具 | `chain`, `product`, `groupby` |
| `functools` | 函数工具 | `lru_cache`, `partial`, `reduce` |
| `dataclasses` | 数据类 | `@dataclass` 自动生成 `__init__` |
| `typing` | 类型注解 | `Optional`, `Union`, `TypedDict` |
| `asyncio` | 异步编程 | `gather`, `wait_for`, `run` |
| `logging` | 日志 | 比 print 更规范的日志输出 |
| `os` / `sys` | 系统操作 | 环境变量、进程参数 |

### collections 实用示例

```python
from collections import Counter, defaultdict, deque

# Counter：统计频次
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
c = Counter(words)
print(c.most_common(2))  # [('apple', 3), ('banana', 2)]

# defaultdict：访问不存在的 key 自动初始化
graph = defaultdict(list)
graph["A"].append("B")   # 不需要先判断 "A" 是否存在

# deque：双端队列，appendleft/popleft 都是 O(1)
dq = deque(maxlen=5)     # 固定大小，超出自动丢弃最旧的
dq.appendleft(0)
```

### logging 基础配置

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s"
)
logger = logging.getLogger(__name__)

logger.info("服务启动")
logger.warning("连接超时，正在重试")
logger.error("数据库连接失败", exc_info=True)  # 自动附上堆栈信息
```

---

## 十、环境与包管理

```bash
# 创建虚拟环境
python3.11 -m venv venv
source venv/bin/activate      # macOS/Linux
venv\Scripts\activate         # Windows

# 包管理
pip install requests
pip install -r requirements.txt
pip freeze > requirements.txt

# 查看已安装包
pip list
pip show requests

# 环境变量（开发常用）
from dotenv import load_dotenv
import os

load_dotenv()                         # 读取 .env 文件
api_key = os.getenv("OPENAI_API_KEY", "default_value")
```

---

!!! note "推荐资源"
    - [廖雪峰 Python 教程](https://liaoxuefeng.com/books/python) — 中文系统教程
    - [Real Python](https://realpython.com) — 中高级 Python 教程
    - [Python 官方文档](https://docs.python.org/zh-cn/3/) — 标准库必备
    - [Python Cookbook](https://python3-cookbook.readthedocs.io/zh_CN/latest/) — 中文版实用技巧

---

## 十一、字符串与编码

```python
# Python 3 中 str 是 Unicode，bytes 是字节序列
s = "你好"                      # str，Unicode
b = s.encode("utf-8")           # str → bytes
s2 = b.decode("utf-8")          # bytes → str

# 常用字符串方法
s = "  Hello, World!  "
s.strip()                        # 去首尾空白
s.lower() / s.upper()
s.replace("Hello", "Hi")
s.split(", ")                    # 分割 → list
", ".join(["a", "b", "c"])       # 拼接
s.startswith("Hello")
s.find("World")                  # 返回索引，不存在返回 -1
s.count("l")

# f-string（推荐）
name, score = "Alice", 98.5
print(f"{name} 得了 {score:.1f} 分")   # 保留1位小数
print(f"{score:>10.2f}")               # 右对齐，宽度10

# 多行字符串
text = """
第一行
第二行
"""

# bytes 操作
data = b"\x48\x65\x6c\x6c\x6f"
print(data.decode("ascii"))      # Hello
```

---

## 十二、条件判断与模式匹配

```python
# 常规 if/elif/else
x = 10
if x > 0:
    print("正数")
elif x == 0:
    print("零")
else:
    print("负数")

# 三元表达式
result = "正" if x > 0 else "非正"

# match-case（Python 3.10+，类似 C 的 switch 但更强）
command = "quit"
match command:
    case "quit":
        print("退出")
    case "go north" | "go south":   # 多值匹配
        print("移动")
    case _:                          # 默认分支
        print("未知命令")

# 匹配数据结构
point = (1, 0)
match point:
    case (0, 0):
        print("原点")
    case (x, 0):
        print(f"X轴，x={x}")
    case (0, y):
        print(f"Y轴，y={y}")
    case (x, y):
        print(f"任意点，x={x}, y={y}")

# 匹配字典
response = {"status": 200, "data": "ok"}
match response:
    case {"status": 200, "data": data}:
        print(f"成功：{data}")
    case {"status": 404}:
        print("未找到")
```

---

## 十三、函数参数详解

```python
# 位置参数
def add(x, y):
    return x + y

# 默认参数（默认值必须是不可变对象！）
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}"

# 可变位置参数 *args → tuple
def total(*nums):
    return sum(nums)
total(1, 2, 3, 4)  # 10

# 可变关键字参数 **kwargs → dict
def show(**info):
    for k, v in info.items():
        print(f"{k}: {v}")
show(name="Alice", age=26)

# * 强制关键字参数（* 后面的参数只能用关键字传入）
def config(host, *, port=8080, debug=False):
    pass
config("localhost", port=3000)     # ✅
config("localhost", 3000)          # ❌ TypeError

# / 强制位置参数（Python 3.8+，/ 前面的只能位置传入）
def func(x, y, /, z=0):
    pass
func(1, 2, z=3)    # ✅
func(x=1, y=2)     # ❌

# 参数顺序：位置参数 / 默认参数 / *args / * / 关键字参数 / **kwargs
def full(a, b=1, *args, key, **kwargs):
    pass

# 解包传参
args = [1, 2, 3]
kwargs = {"sep": "-"}
print(*args)            # 1 2 3
"_".join(map(str, args))
```

---

## 十四、函数式编程

```python
# map：对每个元素应用函数
nums = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, nums))   # [1, 4, 9, 16, 25]
# 推荐用推导式替代：[x**2 for x in nums]

# filter：过滤元素
evens = list(filter(lambda x: x % 2 == 0, nums))  # [2, 4]

# reduce：累积计算
from functools import reduce
product = reduce(lambda x, y: x * y, nums)  # 120

# sorted 高级用法
students = [{"name": "Bob", "score": 85}, {"name": "Alice", "score": 92}]
sorted(students, key=lambda s: s["score"], reverse=True)
sorted(students, key=lambda s: (-s["score"], s["name"]))  # 多字段排序

# 返回函数（闭包）
def make_adder(n):
    def adder(x):
        return x + n      # 捕获外部变量 n
    return adder

add5 = make_adder(5)
add5(3)   # 8

# 注意闭包陷阱
def make_funcs():
    return [lambda: i for i in range(3)]   # ❌ 都返回 2

def make_funcs():
    return [lambda i=i: i for i in range(3)]  # ✅ 用默认参数固定值

# partial：固定部分参数
from functools import partial
int16 = partial(int, base=16)
int16("ff")   # 255

# 高阶函数组合
from functools import lru_cache

@lru_cache(maxsize=128)    # 缓存结果，避免重复计算
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

---

## 十五、迭代器与生成器进阶

```python
# 可迭代对象 vs 迭代器
# 可迭代对象：实现了 __iter__（list、dict、str 等）
# 迭代器：实现了 __iter__ 和 __next__，只能向前，用完即止

from collections.abc import Iterator, Iterable

isinstance([], Iterable)           # True
isinstance([], Iterator)           # False
isinstance(iter([]), Iterator)     # True

# 手动迭代
it = iter([1, 2, 3])
next(it)   # 1
next(it)   # 2
next(it)   # 3
next(it)   # StopIteration

# 自定义迭代器
class Countdown:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        return self

    def __next__(self):
        if self.n <= 0:
            raise StopIteration
        self.n -= 1
        return self.n + 1

list(Countdown(3))   # [3, 2, 1]

# 生成器 send（双向通信）
def accumulator():
    total = 0
    while True:
        value = yield total    # yield 既输出又接收
        total += value

gen = accumulator()
next(gen)           # 启动生成器，返回 0
gen.send(10)        # 返回 10
gen.send(20)        # 返回 30

# itertools 常用工具
import itertools

list(itertools.chain([1,2], [3,4], [5]))     # [1,2,3,4,5]
list(itertools.islice(range(100), 5))        # [0,1,2,3,4]
list(itertools.product("AB", repeat=2))      # AA AB BA BB
list(itertools.combinations([1,2,3], 2))     # (1,2) (1,3) (2,3)
list(itertools.permutations([1,2,3], 2))     # (1,2) (1,3) (2,1) ...

for key, group in itertools.groupby("AAABBBCC"):
    print(key, list(group))   # A ['A','A','A'] ...
```

---

## 十六、模块与包

```python
# 导入方式
import os                        # 使用时 os.path.join(...)
from os import path              # 使用时 path.join(...)
from os.path import join, exists # 直接用 join(...)
import numpy as np               # 别名

# __name__ 的作用
# 直接运行时 __name__ == "__main__"
# 被导入时 __name__ == 模块名
if __name__ == "__main__":
    main()   # 只有直接运行才执行

# 包结构（目录需要 __init__.py）
# mypackage/
#   __init__.py      ← 包的初始化文件，可以为空
#   module_a.py
#   sub/
#     __init__.py
#     module_b.py

# 相对导入（包内部使用）
from . import module_a           # 同级
from ..sub import module_b       # 上级

# __init__.py 控制对外接口
# mypackage/__init__.py
from .module_a import ClassA     # 让用户可以直接 from mypackage import ClassA

# 查看模块内容
import os
dir(os)           # 列出所有属性和方法
help(os.path)     # 查看文档
```

---

## 十七、OOP 访问控制

```python
class Student:
    def __init__(self, name, score):
        self.name = name         # 公开
        self._score = score      # 约定私有（仍可访问，只是暗示）
        self.__grade = "A"       # 名称改写为 _Student__grade，真正限制外部访问

    def get_grade(self):
        return self.__grade

s = Student("Alice", 95)
s.name                 # ✅
s._score               # ✅（可以访问，但不建议）
s.__grade              # ❌ AttributeError
s._Student__grade      # ✅ 绕过限制（不推荐）

# 使用 @property 替代 getter/setter
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):            # getter
        return self._radius

    @radius.setter
    def radius(self, value):     # setter
        if value < 0:
            raise ValueError("半径不能为负")
        self._radius = value

    @property
    def area(self):              # 只读属性
        import math
        return math.pi * self._radius ** 2

c = Circle(5)
c.radius        # 调用 getter
c.radius = 10   # 调用 setter
c.area          # 只读
```

---

## 十八、OOP 定制类（魔法方法）

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):          # 开发者用，eval(repr(obj)) 应能还原对象
        return f"Vector({self.x}, {self.y})"

    def __str__(self):           # 用户友好的字符串，print() 调用
        return f"({self.x}, {self.y})"

    def __len__(self):           # len(obj)
        return 2

    def __add__(self, other):    # obj + other
        return Vector(self.x + other.x, self.y + other.y)

    def __eq__(self, other):     # obj == other
        return self.x == other.x and self.y == other.y

    def __lt__(self, other):     # obj < other（配合 sorted 使用）
        return (self.x**2 + self.y**2) < (other.x**2 + other.y**2)

    def __bool__(self):          # bool(obj)
        return self.x != 0 or self.y != 0

    def __getitem__(self, index):  # obj[index]
        return (self.x, self.y)[index]

    def __iter__(self):          # for x in obj
        yield self.x
        yield self.y

    def __contains__(self, item): # item in obj
        return item in (self.x, self.y)

    def __call__(self, scale):   # obj(args) — 让实例可调用
        return Vector(self.x * scale, self.y * scale)

# __slots__：限制实例属性，节省内存（适合大量创建的简单对象）
class Point:
    __slots__ = ("x", "y")      # 只允许 x 和 y 属性
    def __init__(self, x, y):
        self.x = x
        self.y = y
# p.z = 1  ❌ AttributeError
```

---

## 十九、枚举类

```python
from enum import Enum, IntEnum, auto

class Color(Enum):
    RED   = 1
    GREEN = 2
    BLUE  = 3

Color.RED           # <Color.RED: 1>
Color.RED.name      # 'RED'
Color.RED.value     # 1
Color(1)            # <Color.RED: 1>  ← 通过值获取成员
list(Color)         # 所有成员

# auto() 自动赋值
class Direction(Enum):
    NORTH = auto()  # 1
    SOUTH = auto()  # 2
    EAST  = auto()  # 3
    WEST  = auto()  # 4

# IntEnum：可以和 int 比较
class Status(IntEnum):
    PENDING  = 0
    APPROVED = 1
    REJECTED = 2

Status.APPROVED == 1   # True
```

---

## 二十、单元测试

```python
import unittest

def add(a, b):
    return a + b

class TestAdd(unittest.TestCase):

    def setUp(self):
        """每个测试方法前执行"""
        pass

    def tearDown(self):
        """每个测试方法后执行"""
        pass

    def test_positive(self):
        self.assertEqual(add(1, 2), 3)

    def test_negative(self):
        self.assertEqual(add(-1, -2), -3)

    def test_zero(self):
        self.assertEqual(add(0, 0), 0)

    def test_type_error(self):
        with self.assertRaises(TypeError):
            add("1", 2)

    def test_float(self):
        self.assertAlmostEqual(add(0.1, 0.2), 0.3, places=10)

# 常用断言
# assertEqual / assertNotEqual
# assertTrue / assertFalse
# assertIsNone / assertIsNotNone
# assertIn / assertNotIn
# assertRaises(ExceptionType)
# assertAlmostEqual(a, b, places=7)

# 运行：python -m unittest test_module.py
# 或：python -m pytest（更推荐）
```

---

## 二十一、正则表达式

```python
import re

text = "2024-03-15 报销金额：¥580.00，类别：差旅费"

# 基础匹配
re.match(r"\d+", text)           # 从头匹配，返回 Match 或 None
re.search(r"\d+", text)          # 全文搜索，返回第一个 Match
re.findall(r"\d+", text)         # 返回所有匹配列表：['2024', '03', '15', '580', '00']
re.finditer(r"\d+", text)        # 返回 Match 迭代器

# 常用模式
# \d  数字，\D 非数字
# \w  字母数字下划线，\W 反
# \s  空白，\S 反
# .   任意字符（不含换行）
# ^   开头，$ 结尾
# *   0次+，+  1次+，? 0或1次
# {n} n次，{n,m} n到m次
# []  字符集 [a-z0-9]，[^a-z] 取反
# ()  分组捕获，(?:) 非捕获组
# |   或

# 分组捕获
m = re.search(r"(\d{4})-(\d{2})-(\d{2})", text)
if m:
    print(m.group(0))   # 2024-03-15（完整匹配）
    print(m.group(1))   # 2024
    print(m.groups())   # ('2024', '03', '15')

# 命名分组
m = re.search(r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})", text)
m.group("year")   # 2024

# 替换
re.sub(r"\d+", "N", text)        # 把所有数字替换为 N
re.sub(r"(\w+)@(\w+)", r"\2@\1", email)  # 分组引用

# 分割
re.split(r"[,，；;]\s*", "a, b，c; d")  # ['a', 'b', 'c', 'd']

# 编译（多次使用时提高性能）
pattern = re.compile(r"\d{4}-\d{2}-\d{2}", re.IGNORECASE)
pattern.findall(text)

# flags
re.IGNORECASE  # 忽略大小写
re.MULTILINE   # ^ $ 匹配每行开头结尾
re.DOTALL      # . 匹配换行符
```

---

## 二十二、序列化

```python
# JSON（最常用）
import json

data = {"name": "Alice", "scores": [95, 87, 92], "active": True}

# 序列化
json_str = json.dumps(data, ensure_ascii=False, indent=2)
# 写文件
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# 反序列化
obj = json.loads(json_str)
with open("data.json", encoding="utf-8") as f:
    obj = json.load(f)

# 自定义类的 JSON 序列化
from datetime import datetime

class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

json.dumps({"time": datetime.now()}, cls=DateTimeEncoder)

# pickle（Python 特有，可序列化任意对象，但不跨语言）
import pickle

data = {"key": [1, 2, 3]}
b = pickle.dumps(data)           # 序列化为 bytes
obj = pickle.loads(b)            # 反序列化

with open("data.pkl", "wb") as f:
    pickle.dump(data, f)
with open("data.pkl", "rb") as f:
    obj = pickle.load(f)

# 注意：pickle 有安全风险，不要反序列化不信任的数据
```

---

## 二十三、调试技巧

```python
# 1. print 调试（最简单）
print(f"DEBUG: x={x}, type={type(x)}")

# 2. assert 断言
assert len(data) > 0, f"data 不能为空，当前值：{data}"
# 运行时加 -O 参数可以禁用所有断言

# 3. logging（推荐替代 print）
import logging
logging.basicConfig(level=logging.DEBUG,
                    format="%(asctime)s %(levelname)s: %(message)s")
logger = logging.getLogger(__name__)

logger.debug("详细调试信息")
logger.info("普通信息")
logger.warning("警告")
logger.error("错误", exc_info=True)   # 附上堆栈

# 4. pdb 交互式调试
import pdb
pdb.set_trace()   # 程序暂停，进入调试控制台
# Python 3.7+ 可以直接用 breakpoint()

# pdb 常用命令
# n (next)      执行下一行
# s (step)      进入函数
# c (continue)  继续执行
# p <var>       打印变量
# l (list)      显示当前代码
# q (quit)      退出
```

