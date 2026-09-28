---
title: "Claude 会话：多 Agent 并发模型与 Java/Node IO 模型深谈"
type: source
tags: [并发, IO模型, 多路复用, 事件循环, 线程, Claude Code, Java, Node.js]
date: 2026-09-28
source_file: raw/sessions/claude-concurrency-io-session.md
---

## Summary

Claude Code 会话(2026-09-26 ~ 09-28,8 轮问答)。以一次源码级分析开篇——[[ClaudeCode]] 并行运行多个子 agent 时是否用多线程(答案:否),随后顺势展开应用层并发与 IO 的体系化讲解:[[IOModels]](BIO/NIO/AIO 及阻塞×同步/异步二维矩阵)、[[FutureAndPromise]]、[[IOMultiplexing]] 内核原理(select/poll/epoll)、日常开发的框架地图与铁律、同步/异步的命名词源、线程阻塞与挂起的术语辨析。全会话以"单线程事件循环 + 多路复用"为主线,把 [[Thread]]/[[Process]] 等 OS 概念自然延伸到 Java/Node 工程实践。

## Key Claims

- **Claude Code 并行子 agent 不用多线程**:全仓库无 `worker_threads`;in-process 路径是单线程事件循环上的 async/await 协程并发(`StreamingToolExecutor` 对工具调用不 await、`Promise.race` 等待;Agent 工具标记 `isConcurrencySafe`);swarm/tmux 队友走**多进程**;workflow 是 `LocalWorkflowTask`,复用同一 `runAgent()` 路径——同为单线程 async 并发。
- **BIO/NIO/AIO 由两个正交维度组合**:阻塞/非阻塞(线程能不能先去干别的)× 同步/异步(数据拷贝谁来做)。NIO 非阻塞但仍是**同步**(就绪后应用亲自 read 拷贝);AIO 才把拷贝也交给内核(Proactor);BIO 一连接一线程是 C10K 根源。
- **Future ≠ Promise**:`Future.get()` 是阻塞取结果,`Promise.then()` 是非阻塞注册回调;真正对应 Promise 的是 `CompletableFuture`(thenApply/thenCompose/exceptionally),AIO 的 `CompletionHandler` 回调在语义上最贴 Promise.then。
- **Java AIO 不是单线程事件循环**:底层是 epoll/IOCP 就绪队列 + 线程池并发消费,回调在不同池线程执行需线程安全;Linux 上 JDK AIO 长期是 epoll+线程池模拟,真正内核异步是 `io_uring`。
- **IO 多路复用原理**:阻塞的线程被内核挂到 fd 的等待队列;多路复用让一个线程同时挂到 N 个 fd 上等,epoll 用红黑树(注册一次)+就绪链表(回调挂入)实现 O(1) 获取就绪事件。
- **日常开发铁律**:别在单线程事件循环里用同步 API/跑 CPU 密集任务;别在非阻塞框架(WebFlux/Netty)里写阻塞代码;永远用连接池;所有外部调用设超时。Java 21 虚拟线程把 epoll 多路复用做进运行时,让阻塞式代码也能扛高并发。
- **同步/异步命名**=希腊语 syn(一起)+chronos(时间):同步=两动作时间步调绑死(打电话),异步=各走各的时间线(发消息)。
- **线程阻塞 ≠ 挂起**:阻塞是线程自己等资源而睡眠(OS 等待队列);Java `suspend()` 是被废弃的他人强暂停(不释放锁,会死锁);OS 七状态模型里"挂起"指进程映像换出内存(swap),与阻塞正交。

## Key Quotes

> "并行子 agent = 单线程 async/await 协程并发(in-process 路径)+ 独立 OS 进程(swarm/tmux 路径),没有用到多线程。" —— 会话开篇源码分析的结论

> "多路复用其实就一句话能点破:'阻塞'本身没错,错的是'每个连接都占一个线程去阻塞'。" —— IO 多路复用讲解的开宗明义

> "Future ≈'未来值的阻塞句柄',Promise ≈'未来值的回调协议'。前者问你'现在好了没?没好我原地等着';后者说'好了叫我'。" —— Future 与 Promise 的本质区别

> "底层原理你不需要手写,但必须知道'我这套框架是单线程事件循环,还是线程池,还是非阻塞 Reactor',然后遵守一条铁律——别在你所在的模型上干阻塞/耗时的事。" —— 日常开发建议的总结

## Connections

- [[IOModels]] — BIO/NIO/AIO 二维矩阵与 Reactor/Proactor
- [[IOMultiplexing]] — select/poll/epoll 内核原理
- [[EventLoop]] — 单线程事件循环与 Java AIO 线程池消费的对比
- [[FutureAndPromise]] — Java Future / CompletableFuture / JS Promise
- [[ClaudeCode]] — 会话开篇的源码分析对象
- [[Thread]] — 阻塞/挂起辨析、BIO 一连接一线程的代价
- [[Process]] — swarm 队友走多进程
- [[Socket]] — epoll 监听的对象即 socket fd
- [[ConcurrencyAndParallelism]] — 单线程并发 vs 多核并行的工程印证

## Contradictions

- 无。会话内容与 [[process-vs-thread]]、[[six-ipc-mechanisms]] 的 OS 基础互补:彼处讲概念原理,此处讲这些原理在 Java/Node 框架与真实 agent 系统中的落地。
