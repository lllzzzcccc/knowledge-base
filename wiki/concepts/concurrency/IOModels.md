---
title: "IO 模型 (BIO / NIO / AIO)"
type: concept
tags: [IO, 并发, Java, 操作系统]
sources: [claude-concurrency-io-session]
last_updated: 2026-09-26
---

# IO 模型 (BIO / NIO / AIO)

BIO / NIO / AIO 是 Java 中三种 I/O 模型(也是操作系统 I/O 模型的分层)。理解它们的关键是先立住两个**正交**的维度,很多人把这两维混在一起,导致 NIO 和 AIO 分不清([[claude-concurrency-io-session]])。

## 两个正交维度

| 维度 | 问的是 | 关键区别 |
|---|---|---|
| **阻塞 / 非阻塞** | "调用后,线程能不能先去干别的?" | 阻塞 = 线程被挂起必须等;非阻塞 = 立即返回 |
| **同步 / 异步** | "数据从内核拷到用户空间,这一步谁来做?" | 同步 = 应用自己调 read() 主动拷;异步 = 操作系统拷好再通知 |

## 三种模型对比

| 模型 | 阻塞/非阻塞 | 同步/异步 | 一句话 | 模式 |
|---|---|---|---|---|
| **BIO** | 阻塞 | 同步 | 一直等,数据到了才返回 | — |
| **NIO** | 非阻塞 | **同步** | 先问"好了吗",好了你再亲自去拿 | Reactor |
| **AIO** | 非阻塞 | 异步 | 留个回调,内核拷好数据通知你 | Proactor |

> 关键认知:**NIO 是"非阻塞"的,但仍然是"同步"的**——它只是"不用傻等",数据搬运那一步还是应用自己 read() 完成的。AIO 才把拷贝也交给内核。

## BIO:同步阻塞

一个连接占用一个线程,线程在 `read()` 上挂起直到有数据。逻辑简单、执行流线性好调试;但**伸缩性差**——线程占内存(默认 1MB 栈)、切换开销大,大量线程其实在空等 I/O。这是 **C10K 问题**的根源。典型场景:早期 Tomcat、JDBC 驱动。

## NIO:同步非阻塞

核心武器是 **Selector 多路复用**([[IOMultiplexing]]):一个线程 + 一个 Selector 同时监听成千上万个 Channel,`select()` 返回哪些通道就绪,线程只对就绪的 `read()`。底层对应 OS 的 select/poll/**epoll**(Linux),即 Reactor 模式。典型场景:Netty、Tomcat NIO Connector、Kafka 客户端。

## AIO:异步非阻塞

发起请求后立即返回,操作系统负责把数据从内核拷到用户缓冲区,完成后回调通知(`CompletionHandler`)或返回 `Future`([[FutureAndPromise]]),应用线程从头到尾不参与搬运。典型场景:`AsynchronousFileChannel` 等。

**实现细节**:Windows 的 IOCP 是真正的内核级异步;**Linux 上 JDK 的 AIO 长期用 epoll + 线程池"模拟"**(见 [[EventLoop]]),数据拷贝仍由池线程同步执行——这是 AIO 在 Linux 上性能优势不明显、普及度不如 NIO 的原因之一。真正内核级异步是近年的 **`io_uring`**。

## 电话类比

| | 你等一个电话 |
|---|---|
| BIO | 拿起电话一直听,期间啥也不干 |
| NIO | 雇个秘书(Selector)同时盯 N 个电话,哪个响了叫你,你再过去接 |
| AIO | 留个号码,对方打来转接,还帮你把话抄好送到手上 |

## 与线程模型的关系

- BIO 的"一连接一线程"让 [[Thread]] 成为瓶颈,倒逼出多路复用与异步。
- 单线程事件循环([[EventLoop]])本质就是 NIO/Reactor 思想的延伸:一个线程发起 N 个网络请求,谁的数据回来了处理谁的回调——这就是"单线程为什么能扛高并发"的底层逻辑。
- Java 21 虚拟线程(Project Loom)用调度器 + epoll 把虚拟线程映射到少量真实线程,让阻塞式代码重获高并发能力。

## 来源

- [[claude-concurrency-io-session]]
