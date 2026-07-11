**解决问题：任务还没有完成，但是我要先继续干别的**

*生命周期*
PENDING
(等待中)

      |
      |
      v

RUNNING
(执行中)

      |
      |
      v

FINISHED
(完成)

      |
      |
      v

RESULT
(结果)
一出租车为例Future:

等待出租车回来


出租车出发


出租车到达


Future保存结果

*代码模拟：*
from concurrent.futures import ThreadPoolExecutor
import time


def taxi(order):

    print(
        f"{order} 出发"
    )

    time.sleep(3)

    print(
        f"{order} 到达"
    )

    return f"{order} 完成"


with ThreadPoolExecutor(max_workers=3) as executor:

    future1 = executor.submit(
        taxi,
        "机场订单"
    )

    future2 = executor.submit(
        taxi,
        "火车站订单"
    )

    future3 = executor.submit(
        taxi,
        "码头订单"
    )


    print("调度中心继续处理其他事情")


    print(
        future1.result()
    )

    print(
        future2.result()
    )

    print(
        future3.result()
    )

*执行结果*
机场订单 出发
火车站订单 出发
码头订单 出发


调度中心继续处理其他事情


3秒后:

机场订单 到达
火车站订单 到达
码头订单 到达


机场订单完成
火车站订单完成
码头订单完成

**核心总结**
Future 是并发编程中的“结果占位符”。
它解决的问题："任务现在开始执行，但结果以后才知道，我先拿一个对象代表这个未来结果。"

python 中：

ThreadPoolExecutor.submit() 返回 Future  //不是立即得到结果，得到的是Future对象，类似：
Future1
 |
 |
等出租车回来
 
Future.result() 获取结果//结果未完成时一直处于阻塞状态
asyncio.Task 基于 Future
await 本质是在等待 Future 完成

如果把协程比作出租车自己决定什么时候停车，那么 Future 就是调度中心给每辆出租车发的订单回执，回来后把结果填进去。
