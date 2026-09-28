# Wiki Index

This file is maintained by the LLM. Updated on every ingest.

## Overview
- [Overview](overview.md) — living synthesis across all sources

## Sources
- [进程 vs 线程：从原理到区别，一次讲清楚](sources/process-vs-thread.md) — 进程/线程本质、上下文切换开销 10~100 倍、并发并行与任务选型
- [一文搞懂六大进程通信机制原理(全网最详细)](sources/six-ipc-mechanisms.md) — Linux 六大 IPC 机制：管道/消息队列/共享内存/信号量/信号/Socket
- [MCP 到底是什么？一文带你彻底搞懂(附:从零构建MCP实例)](sources/mcp-explained.md) — MCP 架构、三大原语、STDIO/SSE/Streamable HTTP 通信机制
- [Claude 会话：多 Agent 并发模型与 Java/Node IO 模型深谈](sources/claude-concurrency-io-session.md) — Claude Code 单线程并发源码分析、BIO/NIO/AIO、epoll 原理、Future vs Promise
- [深入理解Java内存区域与内存模型](sources/java-memory-and-jmm.md) — JVM 运行时数据区、永久代→元空间演进、JMM 主内存/工作内存与 happens-before
- [同步(Synchronous)和异步(Asynchronous)的理解和区别讲解](sources/sync-vs-async.md) — 同步/异步的消息通信机制定义；与阻塞/非阻塞划界
- [Linux IO模型:阻塞、非阻塞、多路复用、异步](sources/linux-io-models.md) — 两阶段框架：没数据时(阻塞/非阻塞)× 有数据后(同步/异步)
- [线程同步与互斥(pthread 原语详解)](sources/pthread-sync-primitives.md) — 互斥锁/条件变量/读写锁/自旋锁/信号量的特点、API 与 demo

## Entities
- [Anthropic](entities/Anthropic.md) — Claude 开发者，MCP 协议的推出方
- [Claude Code](entities/ClaudeCode.md) — Anthropic 的 AI 编程 CLI；子 agent 单线程 async 并发 + swarm 多进程

## Concepts
- [进程 (Process)](concepts/Process.md) — 资源分配基本单位；地址空间结构与生命周期
- [线程 (Thread)](concepts/Thread.md) — CPU 调度执行基本单位；三类私有资源与五类共享资源
- [上下文切换 (Context Switch)](concepts/ContextSwitch.md) — 进程/线程上下文构成与 10~100 倍开销差
- [用户态与内核态 (User Mode & Kernel Mode)](concepts/UserModeVsKernelMode.md) — 系统调用与陷入；判断代码是否进内核的方法（含 strace 验证）
- [并发与并行 (Concurrency & Parallelism)](concepts/ConcurrencyAndParallelism.md) — 时间片轮转、并发 vs 并行、CPU/IO 密集型选型
- [进程通信 (IPC)](concepts/IPC.md) — 经内核通信的必然性；六大机制总览与演进逻辑
- [管道 (Pipe)](concepts/Pipe.md) — 内核内存缓冲区；匿名管道限父子进程，FIFO 无亲缘限制
- [消息队列 (Message Queue)](concepts/MessageQueue.md) — 内存中的消息链表；有格式但拷贝开销大
- [共享内存 (Shared Memory)](concepts/SharedMemory.md) — 页表映射同一物理区域；最快的 IPC，需同步
- [信号量与 PV 操作 (Semaphore)](concepts/Semaphore.md) — 资源计数变量；P 申请 V 释放，实现互斥与同步
- [信号 (Signal)](concepts/Signal.md) — 唯一异步 IPC 机制；硬件/软件来源，≠ 信号量
- [套接字 (Socket)](concepts/Socket.md) — 封装 TCP/IP 的 API；唯一能跨网络的进程通信
- [模型上下文协议 (Model Context Protocol)](concepts/ModelContextProtocol.md) — 连接 LLM 与外部工具的开放协议；Client-Server 架构与三种传输方式
- [MCP 工具 (Tools)](concepts/MCPTool.md) — 可执行能力模块，类似函数调用，会改变系统状态
- [MCP 资源 (Resources)](concepts/MCPResource.md) — 只读数据资源，URI 定位，只提供上下文不执行操作
- [MCP 提示词 (Prompts)](concepts/MCPPrompt.md) — 标准化任务模板，规范模型行为与任务流程
- [IO 模型 (BIO/NIO/AIO)](concepts/IOModels.md) — 阻塞×同步/异步二维矩阵；Reactor/Proactor；io_uring
- [IO 多路复用 (I/O Multiplexing)](concepts/IOMultiplexing.md) — 等待队列、select/poll/epoll、C10K
- [事件循环 (Event Loop)](concepts/EventLoop.md) — 单线程消费就绪事件队列；Node libuv vs Java AIO 线程池
- [Future 与 Promise](concepts/FutureAndPromise.md) — 阻塞句柄 vs 回调协议；CompletableFuture
- [进程地址空间布局 (Address Space Layout)](concepts/AddressSpaceLayout.md) — 用户空间/内核空间两半划分、页表权限位、跨空间数据通路
- [Java 虚拟机 (JVM)](concepts/JVM.md) — 进程内的字节码运行时；启动流程、三大子系统、与硬件虚拟机的区别
- [JVM 运行时数据区 (Java 内存结构)](concepts/JvmRuntimeMemory.md) — 堆/方法区共享、虚拟机栈/PC 私有；永久代→元空间
- [Java 内存模型 (JMM)](concepts/JavaMemoryModel.md) — 主内存/工作内存抽象、happens-before 八条、重排序与内存屏障
- [协程与 Goroutine (Coroutine & Goroutine)](concepts/Goroutine.md) — GMP 模型、动态栈、netpoller；把调度与等待搬回用户态
- [线程同步原语 (Thread Synchronization Primitives)](concepts/ThreadSynchronization.md) — pthread 五件套：mutex/cond/rwlock/spinlock/信号量；虚假唤醒与自旋锁取舍

## Syntheses
- [线程的本质：三层剥洋葱](syntheses/thread-essence.md) — 线程这个抽象在硬件层和内核层的真实面目
