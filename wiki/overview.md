---
title: "Overview"
type: synthesis
tags: [操作系统, 并发, IO, MCP, AI]
sources: [process-vs-thread, six-ipc-mechanisms, mcp-explained, claude-concurrency-io-session]
last_updated: 2026-09-28
---

# Overview

*This page is maintained by the LLM. It is updated on every ingest to reflect the current synthesis across all sources.*

## 知识领域一：操作系统基础（进程、线程与 IPC）

知识库现有两篇知乎操作系统专栏文章，构成互补的整体：[[process-vs-thread]] 回答"[[Process]] 与 [[Thread]] 是什么、为什么区分"，[[six-ipc-mechanisms]] 回答"隔离的进程之间如何通信（[[IPC]]）"。

### 进程与线程模型

- **执行环境**：[[UserModeVsKernelMode]]——CPU 特权级划分用户态/内核态，系统调用即陷入；也给出"判断一行代码是否陷入内核"的判据与 strace 验证法。
- **[[Process]]** = 资源分配基本单位：程序在专属地址空间（代码区/数据区/堆栈区）内的动态执行实例，地址空间相互隔离（一个崩溃不影响其他）。
- **[[Thread]]** = CPU 调度执行基本单位：仅私有运行栈、PC、部分寄存器，共享进程的五类资源（代码/数据/堆/文件描述符/信号处理）。
- **[[ContextSwitch]]**：进程切换开销是线程切换的 10~100 倍（进程上下文含地址空间信息）——线程存在的核心原因。
- **[[ConcurrencyAndParallelism]]**：单核靠时间片轮转实现并发，多核实现并行；CPU 密集型宜多进程/少线程，IO 密集型宜多线程。

### 进程通信六机制（Linux 内核）

进程用户地址空间隔离、内核空间共享 → 通信必经内核。演进主线：**[[Pipe]]（无格式字节流）→ [[MessageQueue]]（有格式但拷贝开销大）→ [[SharedMemory]]（零拷贝最快但需同步）**，配套 **[[Semaphore]]**（PV 操作实现互斥与同步）、**[[Signal]]**（唯一异步机制，≠ 信号量）、**[[Socket]]**（唯一可跨网络）。

### 贯穿性主线

"隔离换安全，共享换效率"：进程间用地址空间隔离换取稳定性（崩溃隔离），线程与共享内存用共享换取性能；所有同步机制（信号量/PV、互斥锁）都是在共享前提下重建秩序的补丁。用户态同步原语的全家福（互斥锁/条件变量/读写锁/自旋锁）见 [[ThreadSynchronization]]。

## 知识领域二：应用层并发与 IO 模型

[[claude-concurrency-io-session]] 把 OS 概念延伸到 Java/Node 工程实践，主线是"**用极少的线程扛住海量 I/O**"：

- **[[IOModels]]**：BIO（一连接一线程，C10K 根源）→ NIO（非阻塞但同步，Reactor）→ AIO（内核代劳拷贝，Proactor）；两个正交维度是阻塞/非阻塞 × 同步/异步。
- **[[IOMultiplexing]]**：让一个线程同时阻塞在 N 个连接上；epoll 以红黑树注册 + 就绪链表回调实现 O(1) 就绪查询。
- **[[EventLoop]]**：单线程串行消费就绪事件队列（Node/libuv）；Java AIO 则是就绪队列 + 线程池并发消费——回调是否要加锁由此决定。
- **[[FutureAndPromise]]**：Future 是阻塞句柄，Promise 是回调协议；CompletableFuture 才是 Java 版 Promise。
- **[[AddressSpaceLayout]]**：每个进程地址空间分用户空间（私有）与内核空间（共享同一内核）两半，页表权限位 + CPU 特权级硬件强制；I/O、IPC、系统调用本质都是跨这个边界。
- **Java 内存三层视角**：[[JVM]]（进程内的字节码运行时，租户）→ [[JvmRuntimeMemory]]（JVM 对用户空间的逻辑分区：堆/方法区共享 + 每线程私有栈/PC）→ [[JavaMemoryModel]]（其上的并发规范：主内存/工作内存是物理内存与 CPU 缓存的抽象，happens-before 管可见性与有序性）。
- **协程主线**：[[Goroutine]]（GMP + 动态栈 + netpoller）把调度决策和等待挂起搬回用户态，让"过边界"的频率和代价最小化——写同步代码享受 NIO 性能。
- **实证**：[[ClaudeCode]] 的并行子 agent 即 EventLoop 模型——单线程 async/await 协程并发（in-process）+ 多进程（swarm/tmux），全程无 worker_threads；Java 21 虚拟线程则把 epoll 多路复用做进运行时。
- **铁律**：别在单线程事件循环里跑阻塞/CPU 密集任务；别在非阻塞框架里写阻塞代码；永远用连接池、设超时。

## 知识领域三：AI 应用基础设施（MCP）

[[mcp-explained]] 将知识库扩展到 AI 工程领域：[[ModelContextProtocol]]（MCP，[[Anthropic]] 推出的开放协议）为 LLM 与外部工具/数据源建立标准化连接，被类比为"AI 应用的 USB-C 接口"。

- **架构**：MCP Host（Cursor、Claude Desktop、Cline 等）→ MCP Client（调度与通信层）→ MCP Server（能力网关）。
- **三大原语**：[[MCPTool]]（可执行、改变状态）、[[MCPResource]]（只读上下文，URI 定位）、[[MCPPrompt]]（标准化任务模板）。
- **通信机制**：统一 JSON-RPC；传输层 STDIO（本地进程级，默认）→ SSE（已弃用）→ Streamable HTTP（官方推荐）。
- **工程价值**：解耦模型与工具的交互复杂度，而非替代现有架构。

### 跨领域呼应

三大领域形成"基础机制 → 上层应用"的纵向贯通：MCP 的 STDIO 本质是 [[Pipe]]（匿名管道），SSE/Streamable HTTP 建立在 [[Socket]] 的跨网络能力之上；应用层并发复用 OS 的等待队列与 [[UserModeVsKernelMode]] 系统调用（epoll_wait 即陷入内核），又用 [[ClaudeCode]] 的实例印证 [[Thread]]/[[Process]] 的取舍。线程本质的三层拆解见综合页 [[thread-essence]]。
