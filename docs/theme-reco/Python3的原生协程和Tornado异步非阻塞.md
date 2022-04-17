---
title: Python3的原生协程和Tornado异步非阻塞
date: 2020-11-23 18:30:38
tags: [协程,异步非阻塞]
---



 我们知道在程序在执行 IO 密集型任务的时候，程序会因为等待 IO 而阻塞，而协程作为一种用户态的轻量级线程，可以帮我们解决这个问题。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存，在调度回来的时候，恢复先前保存的寄存器上下文和栈。因此协程能保留上一次调用时的状态，即所有局部状态的一个特定组合

​    说人话：说白了就是，当协程遇到io操作而阻塞时，立即切换到别的任务，如果操作完成则进行回调返回执行结果，提高了效率，同时这样也可以充分利用 CPU 和其他资源，这就是异步协程的优势，并且协程本质上是个单进程，相对于多进程来说，无需进程间上下文切换的开销，无需原子操作锁定及同步的开销，编程模型也非常简单。

​    在python2以及python3.3时代，人们使用协程还得基于greenlet或者gevent，greenlet机制的主要思想是：生成器函数或者协程函数中的yield语句挂起函数的执行，直到稍后使用next()或send()操作进行恢复为止。可以使用一个调度器循环在一组生成器函数之间协作多个任务，它的缺点是必须通过安装三方库进行使用，使用时由于封装特性导致性能有一定的流失。

​    终于在python3.4中，我们迎来了python的原生协程关键字:Async和Await，它们的底层基于生成器函数，使得协程的实现更加方便。

​    Async 用来声明一个函数为异步函数，异步函数的特点是能在函数执行过程中挂起，去执行其他异步函数，等到挂起条件（假设挂起条件是sleep(5)）消失后，也就是5秒到了再回来执行。
    Await 用来用来声明程序挂起,比如异步程序执行到某一步时需要等待的时间很长，就将此挂起，去执行其他的异步程序

​    首先我们先来看一个不使用协程的程序

​    

```
import time
def job(t):
    time.sleep(t) 
    print('用了%s' % t)
def main():
    [job(t) for t in range(1,3)]
start = time.time()
main()
print(time.time()-start)
```

![img](https://v3u.cn/v3u/Public/js/editor/attached/image/20190920/20190920070853_68425.png)    

​    从运行结果可以看出，我们的 job 是按顺序执行的。必须执行完 job 1 才能开始执行 job 2， job 1 需要 1 秒的执行时间，job 2 需要 2 秒的执行时间，所以总时间是 3 秒多。

​    

​    如果我们使用协程的方式，job 1 在等待 time.sleep(t) 执行结束的时候,是可以切换到 job 2 执行的。

​    

```
import time
import asyncio
async def job(t):  # 使用 async 关键字将一个函数定义为协程
    await asyncio.sleep(t)  # 等待 t 秒, 期间切换执行其他任务
    print('用了%s秒' % t)
async def main(loop):  # 使用 async 关键字将一个函数定义为协程
    tasks = [loop.create_task(job(t)) for t in range(1,3)]  # 创建任务, 不立即执行
    await asyncio.wait(tasks)  # 执行并等待所有任务完成
start = time.time()
loop = asyncio.get_event_loop()  # 建立 loop
loop.run_until_complete(main(loop))  # 执行 loop
loop.close()  # 关闭 loop

print(time.time()-start)
```

​     ![img](https://v3u.cn/v3u/Public/js/editor/attached/image/20190920/20190920071108_26974.png)

​    从运行结果可以看出，我们没有等待 job 1 执行结束再开始执行 job 2，而是 job 1 触发 await 的时候切换到了 job 2 。 这时 job 1 和 job 2 同时在执行 await asyncio.sleep(t)，所以最终程序的执行时间取决于执行时间最长的那个 job，也就是 job 2 的执行时间：2 秒

​    由此可见，效率提高非常明显。

​    同理，在之前一篇文章中：[关于Tornado:真实的异步和虚假的异步](https://v3u.cn/a_id_107) 提到了tornado默认是同步阻塞机制，如果要激活异步非阻塞的特性，需要使用异步写法，在那篇文章我使用的装饰器的形式来声明异步方法，而在这里，我们同样可以使用async和await来进行协程的异步非阻塞任务

​    

```
import tornado.web
from tornado import gen
class IndexHandler(tornado.web.RequestHandler):
    def get(self):
        self.write('index')
async def doing():
    await gen.sleep(10)  # here are doing some things
    return 'Non-Blocking'
class NonBlockingHandler(tornado.web.RequestHandler):
    async def get(self):
        result = await doing()
        self.write(result)
application = tornado.web.Application([
    (r"/", IndexHandler),
    (r"/nonblocking", NonBlockingHandler),
])
if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

​     可以看到，虽然代码可读性下降了一点，但是性能和效率却实实在在的提升了