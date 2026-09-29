# Wiki Log

Append-only chronological record of all operations.

Format: `## [YYYY-MM-DD] <operation> | <title>`

Parse recent entries: `grep "^## \[" wiki/log.md | tail -10`

---

## [2026-09-25] ingest | 进程 vs 线程：从原理到区别，一次讲清楚

知乎专栏文章（raw/articles/process-vs-thread.md）。新增概念页：Process、Thread、ContextSwitch、ConcurrencyAndParallelism、Semaphore（与第二篇文章共建）。核心结论：进程切换开销是线程切换的 10~100 倍；线程共享进程五类资源；CPU 密集型宜多进程、IO 密集型宜多线程。

## [2026-09-25] ingest | 一文搞懂六大进程通信机制原理(全网最详细)

知乎专栏文章（raw/articles/six-ipc-mechanisms.md）。新增概念页：IPC、Pipe、MessageQueue、SharedMemory、Semaphore、Signal、Socket。核心结论：进程通信必须经内核；六大机制按"管道→消息队列→共享内存"效率演进；信号是唯一的异步机制，与信号量完全不同。

## [2026-09-25] query | 用户态与内核态（判断代码是否陷入内核的方法）

由会话问答提炼新增概念页 concepts/UserModeVsKernelMode.md：核心判据（是否需要内核特权操作）、缓冲层与隐藏系统调用两个反直觉陷阱、strace 查证法、vDSO 例外；并回链 Process、ContextSwitch 页。

## [2026-09-26] ingest | MCP 到底是什么？一文带你彻底搞懂(附:从零构建MCP实例)

腾讯云开发者社区文章（raw/articles/mcp-explained.md）。知识库首个 AI 领域来源：新增概念页 ModelContextProtocol、MCPTool、MCPResource、MCPPrompt，实体页 Anthropic。核心结论：MCP 是 Anthropic 推出的"AI USB-C"开放协议，Client-Server 架构，三大原语 Tools/Resources/Prompts 分工，通信统一 JSON-RPC（STDIO 默认 / SSE 已弃用 / Streamable HTTP 推荐）。跨领域呼应：STDIO 复用 Pipe 机制，网络传输建立在 Socket 之上。

## [2026-09-28] ingest | Claude 会话：多 Agent 并发模型与 Java/Node IO 模型深谈

Claude Code 会话 a2a769e1-cf8e-4290-8cc1-efcd0f073f82（raw/sessions/claude-concurrency-io-session.md，2026-09-26 ~ 09-28，8 轮问答）。新增概念页 IOModels、IOMultiplexing、EventLoop、FutureAndPromise，实体页 ClaudeCode；Thread 页补充"阻塞 ≠ 挂起"辨析。核心结论：Claude Code 并行子 agent 全程单线程 async/await 协程并发（swarm 走多进程），workflow 同为 LocalWorkflowTask；IO 主线为 BIO→NIO→AIO 二维矩阵 + epoll 就绪链表 + 事件循环。知识库新增"应用层并发与 IO 模型"领域。

## [2026-09-28] query | 线程的本质是什么？

由会话问答提炼新增综合页 syntheses/thread-essence.md：三层拆解——抽象层（执行与资源分离）、硬件层（线程=一份寄存器现场快照，CPU 不知线程存在）、内核层（Linux 无线程结构，task_struct 共享 mm_struct 即线程；协程是同路线延伸）。

## [2026-09-28] query | 进程地址空间的用户/内核划分 & 协程在该边界上的优化

连续问答固化：新增概念页 AddressSpaceLayout（两半空间的权限位、数据通路、与 IPC/IOMultiplexing 的关系）与 Goroutine（GMP 模型、动态栈、阻塞三条出路、与线程的边界 crossings 对比表）；UserModeVsKernelMode、Process、IOMultiplexing、EventLoop 四页回链。核心结论：协程未消除用户/内核边界，但把调度决策与等待挂起搬回用户态，过边界的线程数从 N 降到 ~核数。

## [2026-09-28] ingest | 深入理解Java内存区域与内存模型：从运行时数据区到并发可见性的全解析

CSDN 文章（raw/articles/java-memory-and-jmm.md）。起因于用户提问"JVM 内存结构与我讲的 OS 层布局为何不同"。新增概念页 JvmRuntimeMemory（运行时数据区：堆/方法区共享 + 虚拟机栈/PC/本地方法栈私有；永久代→元空间演进）与 JavaMemoryModel（JMM：主内存/工作内存抽象 = 物理内存/CPU 缓存的规范化身，happens-before 八条）；Thread、Semaphore、AddressSpaceLayout 三页回链。核心结论：三层视角互不冲突——OS 层画真实的地址空间，JVM 结构层是 JVM 的逻辑分区，JMM 是并发规则手册而非内存布局。

## [2026-09-28] query | 什么是 JVM？Java 程序运行时是否启动虚拟机？

由会话问答提炼新增概念页 concepts/JVM.md：JVM = 规范 + 实现，是进程内执行字节码的语言运行时（非硬件虚拟机）；`java Hello` = OS 创建普通进程、启动器加载 libjvm.so、JVM 以动态库形式住在进程内（一程序一进程一实例）；Java 线程与 OS 线程 1:1。三大子系统：类加载器/执行引擎(解释+JIT)/GC。JvmRuntimeMemory、AddressSpaceLayout、JavaMemoryModel 三页回链。核心结论：JVM 是"租户"、OS 用户空间是"房子"——堆/方法区是 JVM 的逻辑分区，建在 OS 用户空间之上，层级栈为 两半空间 → 运行时数据区 → JMM。

## [2026-09-28] ingest | 同步(Synchronous)和异步(Asynchronous)的理解和区别讲解

腾讯云文章（raw/articles/sync-vs-async.md，2023-06-11）。同步/异步=消息通信机制（调用者主动等结果 vs 回调/通知），阻塞/非阻塞=线程等待时的状态——两者不可混为一谈；同步等待时线程可能仍激活、白白占资源。与 [[IOModels]]、[[claude-concurrency-io-session]] 互为印证。

## [2026-09-28] ingest | Linux IO模型:阻塞、非阻塞、多路复用、异步

CSDN 文章（raw/articles/linux-io-models.md）。两阶段决策框架：数据没准备好时（用户进程等=阻塞 / 返回 ERROR=非阻塞）× 数据准备好后（等你来取=同步 / 送上门+发信号=异步）。核心结论：不管阻塞 IO 还是非阻塞 IO 都是同步 IO。与 [[IOModels]] 的二维矩阵一致；多路复用"内核拷贝"一句略含混（拷贝实由应用 read() 完成），已在来源页标注。

## [2026-09-28] ingest | 线程同步与互斥(pthread 原语详解)

CSDN 文章（raw/articles/pthread-sync-primitives.md）。pthread 五类原语：互斥锁（原子性/唯一性/非繁忙等待）、条件变量（与锁配合、wait 释放锁、防虚假唤醒须 while 复检）、读写锁（读共享写独占，读多写少）、自旋锁（忙等待，持锁时间 < ~2 次上下文切换）、信号量（初值1互斥/初值0同步）。新增概念页 ThreadSynchronization，Semaphore 页回链。

## [2026-09-29] graph | Knowledge graph rebuilt

38 nodes, 208 edges (208 extracted, 0 inferred).

## [2026-09-29] query | PCB 在哪？进程内存结构图里为何没有它？

由会话问答提炼新增概念页 concepts/ProcessControlBlock.md：PCB 不在进程地址空间，住内核直接映射区（Linux = task_struct，slab 分配）；内容六类（状态/上下文现场/mm 指针/调度信息/资源清单/身份）；生命周期随 fork/切换/退出；用户经 /proc、ps 间接查看；配套每进程内核栈与 current 宏。Process、ContextSwitch 两页回链，index 登记。核心结论：地址空间是"房间"，PCB 是"物业档案"，经 mm_struct 指针关联。
