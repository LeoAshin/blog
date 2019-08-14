---
title: Java 线程池标准 ThreadPoolExecutor
date: 2019-05-10 13:27:21
thumbnail: /gallery/six.png
tags: 
- 线程池
- 笔记
categories: Code
---

面试的时候问了我线程池的问题. 比如说用的哪个类, 里面的参数是怎样的.

```java
    ThreadPoolExecutor pool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, timeUnit, BlockingQueue)
```

<!-- more -->

- corePoolSize: 核心池大小, 也就是核心线程数

    线程池刚开始启动的时候, 默认为0, 如果当前工作线程数少于该值, 则创建新线程

- BlockingQueue: 工作队列
    若在添加新任务的时候, 线程池worker线程已经达到了corePoolSize的值,则将新任务放入workQueue当中等待

- maximumPoolSize: 最大池大小, 也就是最大线程数
    若BlockingQueue被沾满, 则判断该值申请新的线程开始消费workQueue中的任务, 新创线程不超过该值

- keepAliveTime: 线程存活时间
    如果我maximumPoolSize - corePoolSize > 0 的线程中有闲置线程, 则线程在该时间后进行销毁

- timeUnit: 存活时间的单位

这里还有另外一个参数: `handler`

我们来看看jdk8中关于Execute()方法的注释

    * Proceed in 3 steps:
    *
    * 1. If fewer than corePoolSize threads are running, try to
    * start a new thread with the given command as its first
    * task.  The call to addWorker atomically checks runState and
    * workerCount, and so prevents false alarms that would add
    * threads when it shouldn't, by returning false.
    *
    * 2. If a task can be successfully queued, then we still need
    * to double-check whether we should have added a thread
    * (because existing ones died since last checking) or that
    * the pool shut down since entry into this method. So we
    * recheck state and if necessary roll back the enqueuing if
    * stopped, or start a new thread if there are none.
    *
    * 3. If we cannot queue task, then we try to add a new
    * thread.  If it fails, we know we are shut down or saturated
    * and so reject the task.
    */

如果当前运行线程数少于corePoolSize数目, 那么就会尝试创建一个新线程来执行任务这里的计算使用了Atom类来保证其运行在状态和count的原子性

```java
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

```java
    int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
```

如果不能在在队列中安排task, 则会尝试新增加一个线程来处理新任务. 而如果创建失败, 则会拒绝新任务.

整体而言线程的工作做流程分为以下步骤:

- 添加新任务
- 线程核心数没有达到指定数目, 创建新线程
- 达到核心线程数, 安排进Queue中
- Queue中线程数满, 且核心线程数没有空余, 创建新线程来对Queue进行处理, 以FIFO原则处理

- 若Queue满, 核心线程数满, 最大线程数也满, 如果此时定义了`handler` 则会以handler中的方法来处理新任务. 否则直接拒绝新任务的加入
