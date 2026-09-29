---
title: "事件循环 (Event Loop)"
type: concept
tags: [并发, Node.js, 事件循环, 异步]
sources: [claude-concurrency-io-session]
last_updated: 2026-09-26
---

# 事件循环 (Event Loop)

事件循环 = **一个线程循环消费"就绪事件队列"**:I/O 多路复用([[IOMultiplexing]])返回就绪事件后,把对应回调丢进队列,由单线程一个个串行执行([[claude-concurrency-io-session]])。它是"单线程为什么能扛高并发"的架构答案。

## Node.js(libuv)模型

- 底层多路复用:epoll(Linux)/ kqueue(macOS)/ IOCP(Windows)
- 消费执行体:**1 个事件循环线程**,回调串行执行
- 回调里基本不用加锁(单线程),这是与 Java AIO 的关键区别

## 对比:Node 事件循环 vs Java AIO

| 维度 | Node.js(libuv) | Java AIO(NIO.2) |
|---|---|---|
| 底层多路复用 | epoll / kqueue / IOCP | epoll(Linux)/ IOCP(Windows) |
| 消费事件的执行体 | **1 个事件循环线程** | **线程池(N 个线程)** |
| 回调执行 | 串行,一次一个 | 并发,可能同时多个 |
| 回调共享数据 | 基本不用加锁 | 要线程安全 |
| 数据拷贝 | 主线程同步 read | 池线程同步 read |

一句话:**Node 是"一个 loop 串行消费事件";Java AIO 是"一个就绪队列 + 多线程并发消费"。** 生产者-消费者视角:两者都是"事件生产者→队列",Node 只有一个消费者,Java AIO 有多个。注意 Linux 上 JDK AIO 的"异步"是 epoll+线程池模拟出来的语义异步([[IOModels]])。

## 铁律:别堵住唯一的那根线程

- ❌ 在请求处理里用同步 API:`fs.readFileSync`、`execSync`、同步 `crypto`——冻结**整个服务**,所有请求一起卡
- ❌ CPU 密集操作跑主线程:大 `JSON.parse`、大数组循环、图片处理——拖慢事件循环;重活丢给 `worker_threads` / `child_process`
- ✅ 一律 `async/await`(别忘了 `await`,漏了就是 unhandled rejection)
- ✅ 任何 HTTP/DB 调用都设超时,否则一个挂起连接饿死事件循环

[[ClaudeCode]] 的 in-process 子 agent 并发正是这个模型:多个 agent 是事件循环上交错推进的 async/await 协程,每个 agent 大部分时间在 await LLM 网络 I/O,await 期间线程去跑其他 agent 的回调。

## 关联概念

- [[IOMultiplexing]] — 事件循环的底层就绪事件来源
- [[IOModels]] — NIO/Reactor 的延伸形态
- [[FutureAndPromise]] — 事件循环上组织异步逻辑的抽象(async/await、Promise)
- [[Goroutine]] — 同目标的不同路线：Go 把多路复用藏进运行时，程序员照写同步代码
- [[Thread]] — 单线程模型 vs 多线程模型的取舍

## 来源

- [[claude-concurrency-io-session]]
