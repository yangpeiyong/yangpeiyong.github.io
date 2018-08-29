---
layout:     post
title:      "Android 消息机制详解"
date:       2018-08-26 12:32:18 +0800
summary:    "深度介绍Android的消息机制和多线程开发"
categories: Android
---


#### 1. Handler 是什么？

Handler是Android系统设计的消息框架，主要用来做两件事：

+ 在未来某个时刻执行某件事
+ 将某件事交由另一个线程执行

> There are two main uses for a Handler:
> 1. to schedule messages and runnables to be executed as some point in the future;
> 2. to enqueue an action to be performed on a different thread than your own.

为了完成这两件事，Handler消息机制需要三个问题：1）消息如何发送？2）消息如何存储？ 3）消息是如何被处理？

Handler消息框架通过三个核心类：`Handler` `Looper` `MessageQueue` 来完成整个核心框架的功能。


#### 2. Handler 核心机制

整个handler机制的核心是如下三个类：`Handler` `Looper` `MessageQueue`


简单来说Handler消息机制的流程如下：

> 1. Handler 将消息存储到 MessageQueue
> 2. Looper 循环从 MessageQueue 中取出消息交由handler处理

下面我们根据源码分析一下 `Looper` `MessageQueue`的原理。注意这里的源码是部分代码，只抽取出核心逻辑的相关代码。

#### 3. Handler 消息发送

Handler消息发送有如下方法：

```java
post(Runnable r)
postDelayed(Runnable r, long delayMillis)
postAtTime(Runnable r, long uptimeMillis)

sendMessage(Message msg)
sendMessageDelayed(Message msg, long delayMillis)
sendMessageAtTime(Message msg, long uptimeMillis)
```

最终都会构造一个message，通过`Hanlder.enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis)`存储到`MessageQueue`中。

#### 4. Looper 运行原理

```java
public static void loop() {

    final Looper me = myLooper();

    final MessageQueue queue = me.mQueue;

    for (;;) {
        Message msg = queue.next(); // might block
        msg.target.dispatchMessage(msg);
    }

}
```
当Looper执行loop后，就会进入一个无限循环，不停从`MessageQueue`中取出`Message`交由对应的target(Handler)处理，事件处理运行在looper所在的线程中。

#### 5. MessageQueue 运行原理

MessageQueue的核心方法有两个: `enqueueMessage()`和`next()`, enqueueMessage是向队列插入数据，next是从队列取出数据。

注意MessageQueue实际存储的数据格式不是队列，而是单向链表

我们先看看enqueueMessage的核心代码

```java
boolean enqueueMessage(Message msg, long when) {
    synchronized (this) {
        msg.when = when;
        Message p = mMessages;
        if (p == null || when == 0 || when < p.when) {
            msg.next = p;
            mMessages = msg;
        } else {
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
            }
            msg.next = p;
            prev.next = msg;
        }
    }
    return true;
}
```

这段代码就是将Message插入到链表里面，插入位置根据when从小到大的顺序判断。when指的是message的执行时间，越小越靠前。

再看看next的核心代码

```java
    Message next() {
        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            nativePollOnce(ptr, nextPollTimeoutMillis);//important

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

            }
        }
    }
```

next的大体逻辑就是将MessageQueue链表中取出第一个元素，然后将链表头设置为下一个元素。这里调用了native方法`nativePollOnce`, 这是利用Linux的epoll机制来实现MessageQueue为空时，线程阻塞，等待新的message加入MessageQueue。


#### 6. 主线程消息循环模型

waiting to write


#### 7. 一些需要注意的问题

waiting to write

## Draft

Looper 和 MessageQueue 是 Hander 的成员变量

其中 Handler 中的 MessageQueue 来源于 Looper， Looper 通过 ThreadLocal 确保每个线程有一份自己的对象。

因而，无论有多少个handler，对于一个线程来说 Looper 和 MessageQueue只有一个。


This article is being written, if you have any questions or suggestions, pelease contact me with wechat(raccoon-yang) or email(yctx.yang@gmail.com)
