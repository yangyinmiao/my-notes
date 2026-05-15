# Python 基础速查

!!! tip "适用版本"
    本笔记基于 Python 3.10+，有 C 语言基础，重点记录与 C 不同的地方。

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
    - [Real Python](https://realpython.com) — 中高级 Python 教程
    - [Python 官方文档](https://docs.python.org/zh-cn/3/) — 标准库必备
    - [Python Cookbook](https://python3-cookbook.readthedocs.io/zh_CN/latest/) — 中文版实用技巧
