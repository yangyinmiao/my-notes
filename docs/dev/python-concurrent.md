# Python 并发编程

> 涵盖多进程、多线程、协程三种并发模型，以及各自的适用场景。
> 参考：[廖雪峰 Python 教程 · 进程和线程](https://liaoxuefeng.com/books/python/process-thread/index.html)

---

## 一、并发模型选择

```
任务类型
  ├── CPU 密集型（计算、图像处理、加密）
  │     → multiprocessing（绕过 GIL，真并行）
  │
  ├── IO 密集型（网络请求、文件读写、数据库）
  │     ├── 并发量不大 → threading（简单）
  │     └── 大量并发   → asyncio（高效，推荐）
  │
  └── 混合型 → ProcessPoolExecutor + asyncio
```

---

## 二、多进程

```python
from multiprocessing import Process, Pool, Queue, Pipe
import os

# 基础用法
def worker(name):
    print(f"进程 {name}，PID={os.getpid()}")

if __name__ == "__main__":    # 多进程必须在这个保护下运行
    p = Process(target=worker, args=("子进程",))
    p.start()
    p.join()   # 等待子进程结束

# 进程池（推荐，自动管理进程数量）
def square(n):
    return n * n

if __name__ == "__main__":
    with Pool(processes=4) as pool:
        results = pool.map(square, range(10))    # 同步，等全部完成
        print(results)   # [0, 1, 4, 9, 16, ...]

        # 异步提交
        result = pool.apply_async(square, (5,))
        print(result.get(timeout=3))   # 25

# 进程间通信 - Queue
def producer(q):
    for i in range(5):
        q.put(i)
        print(f"生产: {i}")
    q.put(None)   # 结束信号

def consumer(q):
    while True:
        item = q.get()
        if item is None:
            break
        print(f"消费: {item}")

if __name__ == "__main__":
    q = Queue()
    p1 = Process(target=producer, args=(q,))
    p2 = Process(target=consumer, args=(q,))
    p1.start(); p2.start()
    p1.join(); p2.join()

# 进程间通信 - Pipe（双向管道）
def sender(conn):
    conn.send({"msg": "hello"})
    conn.close()

if __name__ == "__main__":
    parent_conn, child_conn = Pipe()
    p = Process(target=sender, args=(child_conn,))
    p.start()
    print(parent_conn.recv())   # {'msg': 'hello'}
    p.join()
```

---

## 三、多线程

```python
import threading
import time

# 基础用法
def task(name, delay):
    print(f"线程 {name} 开始")
    time.sleep(delay)
    print(f"线程 {name} 结束")

t1 = threading.Thread(target=task, args=("A", 2))
t2 = threading.Thread(target=task, args=("B", 1))
t1.start()
t2.start()
t1.join()
t2.join()

# Lock（互斥锁）—— 防止竞态条件
counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:          # 自动 acquire/release，推荐
            counter += 1

# RLock（可重入锁）—— 同一线程可以多次 acquire
rlock = threading.RLock()

# Event —— 线程间信号
event = threading.Event()

def waiter():
    print("等待信号...")
    event.wait()          # 阻塞直到 event.set()
    print("收到信号，继续")

def setter():
    time.sleep(2)
    event.set()

# Semaphore —— 限制并发数量
sem = threading.Semaphore(3)   # 最多 3 个线程同时执行

def limited_task(n):
    with sem:
        print(f"任务 {n} 执行中")
        time.sleep(1)

# ThreadLocal —— 线程私有数据，不同线程互不干扰
local = threading.local()

def thread_func(name):
    local.name = name            # 每个线程有自己的 local.name
    time.sleep(0.1)
    print(local.name)            # 不会被其他线程修改

# 线程池（推荐替代手动管理线程）
from concurrent.futures import ThreadPoolExecutor

def fetch(url):
    # 模拟 IO 操作
    time.sleep(0.5)
    return f"结果: {url}"

with ThreadPoolExecutor(max_workers=5) as executor:
    urls = ["url1", "url2", "url3"]
    futures = [executor.submit(fetch, url) for url in urls]
    results = [f.result() for f in futures]

# 或者用 map
with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(fetch, urls))
```

---

## 四、进程池 vs 线程池（统一接口）

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def cpu_task(n):
    """CPU 密集型"""
    return sum(i * i for i in range(n))

def io_task(url):
    """IO 密集型"""
    import time; time.sleep(0.5)
    return url

# CPU 密集型 → 进程池
with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(cpu_task, [10**6] * 8))

# IO 密集型 → 线程池
with ThreadPoolExecutor(max_workers=20) as executor:
    results = list(executor.map(io_task, urls))

# 两者 API 完全一致，切换只需换类名
```

---

## 五、协程与 asyncio

```python
import asyncio
import time

# 基础协程
async def say_hello(name, delay):
    await asyncio.sleep(delay)    # 非阻塞等待
    print(f"Hello, {name}!")

# 运行单个协程
asyncio.run(say_hello("Alice", 1))

# 并发执行多个协程
async def main():
    start = time.time()

    # gather：并发执行，等全部完成
    await asyncio.gather(
        say_hello("Alice", 2),
        say_hello("Bob", 1),
        say_hello("Charlie", 3),
    )
    # 总耗时约 3s，而不是 6s

    print(f"耗时 {time.time()-start:.1f}s")

asyncio.run(main())

# 获取返回值
async def fetch(url: str) -> str:
    await asyncio.sleep(0.5)
    return f"data from {url}"

async def main():
    results = await asyncio.gather(
        fetch("url1"),
        fetch("url2"),
        fetch("url3"),
    )
    print(results)  # ['data from url1', 'data from url2', 'data from url3']

# 超时控制
async def main():
    try:
        result = await asyncio.wait_for(fetch("slow_url"), timeout=1.0)
    except asyncio.TimeoutError:
        print("超时了")

# Task —— 让协程并发运行，不等待
async def main():
    task1 = asyncio.create_task(say_hello("Alice", 2))
    task2 = asyncio.create_task(say_hello("Bob", 1))
    # 这里可以做其他事情
    await task1
    await task2

# 异步上下文管理器
async def main():
    async with aiohttp.ClientSession() as session:
        async with session.get("https://api.example.com") as resp:
            data = await resp.json()

# 异步迭代器
async def async_range(n):
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async def main():
    async for i in async_range(5):
        print(i)
```

---

## 六、协程 vs 线程 vs 进程

| 维度 | 协程（asyncio）| 多线程 | 多进程 |
|------|--------------|--------|--------|
| 并发方式 | 单线程，用户态切换 | 多线程，OS 调度 | 多进程，真正并行 |
| 切换开销 | 极小 | 中 | 大 |
| 内存占用 | 极小 | 中（每线程约 8MB 栈）| 大（每进程独立内存）|
| 共享数据 | 天然安全（单线程）| 需要加锁 | 需要 IPC |
| GIL 影响 | 无（单线程）| 受限（IO 密集可用）| 不受限 |
| 适合 | 大量并发 IO | 少量并发 IO | CPU 密集计算 |
| 调试难度 | 中 | 高（竞态/死锁）| 中 |

---

## 七、常见并发陷阱

```python
# ❌ 竞态条件（Race Condition）
counter = 0
def bad_increment():
    global counter
    counter += 1   # 读-改-写三步，非原子操作

# ✅ 用锁解决
lock = threading.Lock()
def safe_increment():
    global counter
    with lock:
        counter += 1

# ❌ 死锁（Deadlock）
lock_a = threading.Lock()
lock_b = threading.Lock()

def thread1():
    with lock_a:
        time.sleep(0.1)
        with lock_b:    # 等 B，但 thread2 拿着 B 等 A → 死锁
            pass

# ✅ 统一加锁顺序，或用 acquire(timeout) 设置超时
def thread1():
    with lock_a:
        with lock_b:    # 和 thread2 加锁顺序一致
            pass

# ❌ 在同步代码中调用异步函数
async def async_func():
    return await some_async_op()

# 不能直接调用
result = async_func()   # 返回协程对象，不执行

# ✅ 正确方式
result = asyncio.run(async_func())   # 在新事件循环中运行
# 或在已有事件循环中：
result = await async_func()          # 在 async 函数内
```
