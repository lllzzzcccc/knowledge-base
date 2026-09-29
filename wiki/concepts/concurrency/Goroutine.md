---
title: "协程与 Goroutine (Coroutine & Goroutine)"
type: concept
tags: [并发, Go, 协程, 操作系统]
sources: [claude-concurrency-io-session]
last_updated: 2026-09-28
---

# 协程与 Goroutine (Coroutine & Goroutine)

**协程是用户态调度的执行流，OS 内核看不见它**——内核只看到一群普通线程在跑，协程的调度完全由语言运行时（用户空间代码）完成。Goroutine 是这一抽象最成熟的工业实现（Go），与 [[EventLoop]]（Node）、虚拟线程（Java 21）同属"把调度权从内核拿回用户态"的路线。

## GMP 调度模型

```
        全局队列(GRQ)
            │
   ┌────────┼────────┐
   P0       P1       P2     ← P = 逻辑处理器，持有本地队列(LRQ, 容量256)
   │        │        │
  M0       M1       M2      ← M = OS 线程（内核看得见的实体）
```

- **G**：协程，初始栈 **2KB**，创建/销毁在用户态完成
- **M**：OS 线程封装，数量动态伸缩
- **P**：调度上下文，数量 = `GOMAXPROCS`（默认 CPU 核数），决定最大并行度（[[ConcurrencyAndParallelism]]）

调度策略：**work stealing**——队列空了的 P 从其他 P 的本地队列偷一半 G，或从全局队列取。内核调度器在进程层面做的事，被运行时在用户态重做了一遍。

## 四大机制

**1. 动态栈**：初始 2KB，函数序言检查栈空间，不够则 `morestack` 分配更大栈并拷贝（Go 1.3 起连续栈）。对比 [[Thread]] 的 1MB 固定栈——单机百万级 goroutine 内存可行，同等线程数量光栈就要 ~1TB。

**2. 用户态切换**：只保存/恢复几个寄存器 + 栈指针 + g 指针，不陷入内核，~100ns 以内；线程切换 ~μs 级（[[ContextSwitch]]）。比一次系统调用的"陷入/返回"（[[UserModeVsKernelMode]]）还轻。

**3. 阻塞的三条出路**（没有一种浪费 M）：

| 阻塞类型 | 处理 | M 状态 |
|---|---|---|
| channel/mutex/sleep | `gopark` 挂入运行时等待队列（用户态记账） | 继续跑别的 G |
| 网络 I/O | 底层走 **netpoller**（epoll/kqueue/IOCP，见 [[IOMultiplexing]]），就绪后唤醒回队列 | 继续跑别的 G |
| 系统调用 | G 与 M 一起陷入，但 P **hand off** 给空闲 M | 换绑继续 |

**4. 同步阻塞的魔法**：`http.Get(...)` 这类代码看起来阻塞，运行时就把它变成 epoll 多路复用——写同步代码享受 NIO 性能（[[IOModels]]）。Java 21 虚拟线程同理（mount/unmount carrier 线程）。

## 在用户/内核边界上的优化

协程没有消除 [[AddressSpaceLayout]] 的边界，但把过边界的频率和代价压到最低：

| 动作 | 线程模型 | 协程模型 | 边界 crossings |
|---|---|---|---|
| 调度决策 | 内核调度器 | 用户态 GMP | **消除** |
| 切换 | ~μs，内核参与 | ~100ns，纯用户态 | **消除** |
| 网络 I/O 等待 | 每线程睡内核等待队列 | G 用户态挂起，netpoller 批量轮询 | N 个线程 → ~核数个 |
| 阻塞 syscall | 整个线程进去 | M 进去，P 换绑隔离 | 次数不减，代价隔离 |
| 用户↔内核数据拷贝 | 每次 I/O 一次 | 仍在（零拷贝是另一门技术） | **未消除** |

清醒剂：协程优化的是**控制和等待**，不是**数据通路**；拷贝要靠 zero-copy，与协程正交。

## 与线程对比

| 维度 | 线程（OS） | goroutine |
|---|---|---|
| 调度者 | 内核 | 用户态运行时 |
| 栈 | 固定 ~1MB | 2KB 起步动态增长 |
| 单机量级 | 几千~几万 | 百万级 |
| 切换 | ~μs，陷内核 | ~100ns，用户态 |
| 阻塞代价 | 线程整体让出 | G 挂起，M 照跑 |
| 同步需求 | 共享内存加锁 | **一样要**（channel/mutex/atomic） |

最后一条是常见误解："go 关键字一开就线程安全"是错的——[[Thread]] 的共享数据需 [[Semaphore]] 保护，对 goroutine 完全成立。

## 抽象定位

在 [[thread-essence]] 的阶梯上：`协程 = 现场`（调度权在用户）→ `线程 = 现场 + 内核调度` → `进程 = 现场 + 资源 + 调度`。goroutine 是"协程"层的标准答案。

## 关联概念

- [[AddressSpaceLayout]] — 协程优化的那个"边界"本身
- [[IOMultiplexing]] — netpoller 的 epoll 底牌
- [[EventLoop]] / [[IOModels]] — 同目标的不同路线（Node 把异步暴露给程序员）
- [[Thread]] / [[Process]] / [[ContextSwitch]]

## 来源

- 由 query 工作流提炼（Go 运行时机制 + 知识库并发主线的延伸）
