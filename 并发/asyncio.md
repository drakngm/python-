# asyncio 异步并发框架

> **asyncio = Python 官方异步并发框架**

`asyncio` 基于**事件循环（Event Loop）**实现异步并发，通过协程（Coroutine）、Task 和 Future 协同工作，实现高效处理大量 IO 等待任务。

---

# 一、asyncio 核心组成

整体结构：

```text
                 asyncio
                    |
    --------------------------------
    |              |               |
  协程           Task            Future
    |
async / await
                    |
              Event Loop
```

## 核心概念

| 组件 | 作用 |
| --- | --- |
| Coroutine（协程） | 描述异步任务 |
| async/await | 定义和暂停协程 |
| Event Loop（事件循环） | 调度所有异步任务 |
| Task | 包装协程并负责执行 |
| Future | 保存未来产生的结果 |

---

# 二、简单 asyncio 并发

## 示例

```python
import asyncio


async def taxi(name):
    """
    模拟出租车任务
    """

    print(f"{name} 出发")

    # 模拟等待，例如：
    # 等红灯、等待乘客、网络请求
    await asyncio.sleep(3)

    print(f"{name} 到达")

    return f"{name}完成"


async def main():

    result = await taxi(
        "Taxi A"
    )

    print(result)


asyncio.run(main())
```

---

## 执行过程

```text
main()

  |
  |
  v

执行 taxi()

  |
  |
  v

Taxi A 出发

  |
  |
 await asyncio.sleep(3)

  |
  |
暂停当前协程

  |
  |
3秒后恢复

  |
  |
Taxi A 到达
```

---

## 运行结果

```text
Taxi A 出发

3秒后

Taxi A 到达

Taxi A完成
```

---

# 三、真正并发：asyncio.create_task()

如果有多个出租车任务：

```python
async def main():

    task1 = asyncio.create_task(
        taxi("Taxi A")
    )

    task2 = asyncio.create_task(
        taxi("Taxi B")
    )

    task3 = asyncio.create_task(
        taxi("Taxi C")
    )


    await task1
    await task2
    await task3


asyncio.run(main())
```

---

## 执行过程

```text
Event Loop

      |
      |
 --------------------
 |        |          |
Taxi A  Taxi B    Taxi C

 |        |          |
await    await     await

 |        |          |

等待期间交出控制权

      |
      |
继续调度其他任务
```

---

## 运行结果

```text
Taxi A 出发
Taxi B 出发
Taxi C 出发


3秒后:


Taxi A 到达
Taxi B 到达
Taxi C 到达
```

---

# 四、Task 与 Future 的关系

## Task 本质

```text
Task = Future + 执行协程
```

关系：

```text
Coroutine

    |
    |
    v

 Task

    |
    |
    v

Future

    |
    |
    v

最终结果
```

例如：

```python
task = asyncio.create_task(
    taxi("Taxi A")
)
```

执行过程：

```text
创建 Task

↓

加入 Event Loop

↓

执行协程

↓

保存执行结果到 Future
```

---

# 五、asyncio.gather 批量并发

当多个任务需要同时执行时，可以使用：

```python
asyncio.gather()
```

---

## 示例

```python
async def main():

    results = await asyncio.gather(

        taxi("A"),

        taxi("B"),

        taxi("C")

    )


    print(results)


asyncio.run(main())
```

---

## 执行结果

```text
A 出发
B 出发
C 出发


3秒后:


A 到达
B 到达
C 到达
```

---

返回结果：

```python
[
    'A完成',
    'B完成',
    'C完成'
]
```

---

# 六、asyncio 并发模型

asyncio 不是多线程。

它采用：

> **单线程 + 事件循环 + 协作式调度**

运行模型：

```text
             Event Loop

                  |
        --------------------
        |        |         |
      Task1    Task2     Task3

        |
        |
      await

        |
        |
主动释放执行权

        |
        |
Event Loop 调度其他任务
```

---

# 七、asyncio 工作流程

```text
async def

    |
    |
创建协程

    |
    |
create_task()

    |
    |
生成 Task

    |
    |
Event Loop 调度

    |
    |
await 等待 IO

    |
    |
Future 保存结果

    |
    |
任务完成
```

---

# 八、企业应用场景

asyncio 非常适合：

- HTTP 请求
- 数据库访问
- Redis 操作
- 消息队列
- 文件 IO
- 网络爬虫
- WebSocket

例如：

```python
await database.save()

await redis.set()

await send_message()
```

多个 IO 操作可以：

```python
await asyncio.gather(
    database.save(),
    redis.set(),
    send_message()
)
```

---

# 九、asyncio 总结

> **asyncio 是一个基于事件循环的单线程异步并发框架。**

它通过：

- `async` 定义协程
- `await` 暂停并释放控制权
- `Event Loop` 调度任务
- `Task` 执行协程
- `Future` 保存未来结果

实现：

> **在单线程环境下，高效处理大量 IO 等待任务。**
