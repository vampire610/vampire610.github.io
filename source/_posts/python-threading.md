---
title: python线程学习记录
date: 2023-06-27 10:38:07
tags:
  - python
  - 线程
categories:
  - 学习笔记
  - threading
  - python
archives:
  - python
---

# python threading的一些使用记录

开发python串口上位机的时候需要保证串口一直接收数据的同时主ui界面可以运行操作，学习python线程操作，学习过程中有一些疑问，在此记录。

## 疑问1：线程运行后关闭主界面线程仍在运行

由于使用创建线程来让串口持续接收，在线程函数内写为死循环，但发现该方法在程序线程启动后关闭主程序线程会残留在后台不会停止。

解决办法：设置为守护线程，当主进程停止的时候，会强制关闭守护线程

``` python
# 在启动线程之前，将线程设置为守护线程
my_thread.daemon = True
my_thread.start()
```

## 疑问2：线程结束

编写线程代码时，需要在运行时主动停止线程，目前在网上看到两个方法

1. 使用线程的事件`Event()`

   实例化`Thread.Event()`实质上是定义一个全局变量Flag，程序执行`event.wait()`方法时会阻塞，当Flag值为`True`时，程序执行。

   在编程时可以创建一个event实例，在主进程内改变event内flag状态，在线程内使用`event.is_set()`获取当前状态，来决定是否退出线程内死循环，让线程自然结束后退出。

   > event方法：
   >
   > wait(): 是否阻塞是看event对象内部的Flag值
   >
   > set(): 将Flag值改成True
   >
   > clear(): 将Flag值改成False
   >
   > is_set(): 判断当前的Flag值

2. 可能能用普通变量实现？或者信号量？

## 疑问3：是否.join()？

在用`.start()`启动线程后，是否使用`.join([timeout])`？

`timeout`参数可选，功能是指定线程最多可以霸占CPU资源时间（秒为单位），若省略，则默认直到线程执行结束（进入死亡状态）才释放CPU资源。

## 疑问3：`threading.Thread`还是重写`run()`

在网上看到有两种方法来实现线程，一种是写一个线程函数，使用`my_thread = threading.Thread(target=function, args=(i,x))`来创建线程实例，target指定线程函数，args指定传入参数。

另一种方式是用threading模块实现一个新的线程，需要下面3步：

- 继承于Thread 类
- 重写`__init__`方法，可以添加额外的参数
- 最后，需要重写run()方法实现线程功能

run()方法默认就是创建线程threading时，传递的target function。start方法会自动调用run()。

示例：

``` python
class myThread(threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
        # 以上参数可自定义

    def run(self):
        print("Starting " + self.name)
        print_time(self.name, self.counter, 2)
        print("Exiting " + self.name)
```



