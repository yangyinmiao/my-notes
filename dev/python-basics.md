# Python 基础速查

!!! tip "适用版本"
    本笔记基于 Python 3.9+

## 数据类型

=== "列表 List"
    ```python
    fruits = ["apple", "banana", "cherry"]
    fruits.append("mango")      # 追加
    fruits[0]                   # 索引
    fruits[1:3]                 # 切片
    ```

=== "字典 Dict"
    ```python
    person = {"name": "Alice", "age": 26}
    person.get("name", "unknown")  # 安全取值
    person.items()                  # 遍历
    ```

=== "集合 Set"
    ```python
    tags = {"python", "ai", "code"}
    tags.add("agent")
    tags & {"python", "rust"}    # 交集
    ```

## 函数与装饰器

```python
def greet(name: str, *, formal: bool = False) -> str:
    prefix = "Dear" if formal else "Hey"
    return f"{prefix}, {name}!"

# 装饰器示例
import functools

def retry(times=3):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if i == times - 1:
                        raise
        return wrapper
    return decorator
```

## 常用标准库

| 库 | 用途 |
|----|------|
| `pathlib` | 文件路径操作 |
| `dataclasses` | 数据类 |
| `typing` | 类型注解 |
| `asyncio` | 异步编程 |
| `json` | JSON 序列化 |

!!! note "推荐阅读"
    [Real Python](https://realpython.com) 是非常好的中高级 Python 学习资源。
