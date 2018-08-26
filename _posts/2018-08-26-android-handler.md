---
layout:     post
title:      "Android 消息机制详解"
date:       2018-08-26 12:32:18 +0800
summary:    "深度介绍Android的消息机制和多线程开发"
categories: Android
---

# 1. Handler 是什么？

Handler 是 Android 平台的异步消息分发的框架

# 2. Handler 能做什么？

可以从一个Thread向另一个Thread发送消息。

# 3. Handler 核心机制

整个handler机制的核心是如下三个类：`Handler` `Looper` `MessageQueue`

Looper 和 MessageQueue 是 Hander 的成员变量

其中 Handler 中的 MessageQueue 来源于 Looper， Looper 通过 ThreadLocal 确保每个线程有一份自己的对象。

因而，无论有多少个handler，对于一个线程来说 Looper 和 MessageQueue只有一个。

Handler消息机制的简单流程如下：

Handler --> MessageQueue --> Looper --> Message --> handler

Handler 将消息存储到MessageQueue，Looper从MessageQueue中取出Message，交由对应的handler处理

MessageQueue是一个单向链表结构，插入需要查找响应的位置。
