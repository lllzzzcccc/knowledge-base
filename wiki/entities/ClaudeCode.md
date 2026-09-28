---
title: "Claude Code"
type: entity
tags: [AI, 工具, Agent, 并发]
sources: [claude-concurrency-io-session]
last_updated: 2026-09-26
---

# Claude Code

[[Anthropic]] 推出的 AI 编程 CLI(agent 工具)。在 [[claude-concurrency-io-session]] 中,用户与其子 agent 并行机制的源码分析结论如下。

## 并发模型(源码分析结论)

**并行子 agent 不用多线程**:整个 `src/` 无 `worker_threads` 原语(零命中)。

- **in-process 路径 = 单线程 async/await 协程并发**:一条 assistant 消息发出多个 `Agent` 调用时,`StreamingToolExecutor` 对工具调用发起 Promise 但不 await(`Promise.race` 等待);Agent 工具声明 `isConcurrencySafe`,多个子 agent 作为 AsyncGenerator 在同一事件循环交错推进([[EventLoop]] 模型)。
- **workflow 同为单线程**:workflow 任务是 `LocalWorkflowTask`,子 agent 复用普通 subagent 的 `runAgent()` 路径,`parallel` 编排原语是对 `runAgent()` 的 async 封装。
- **swarm/tmux/iTerm2 队友 = 多进程**:`spawnMultiAgent` 拼命令后 `send-keys` 启动全新 OS 进程([[Process]]),每个进程各自一个线程。
- **Bash 工具**:spawn 子进程执行命令,OS 可调度到不同 CPU 核。

## 为什么不需要多线程

子 agent 耗时大头是**等 LLM API 的网络 I/O**(流式返回),不是 CPU 密集计算。单线程事件循环上的 async I/O 足以让大量 agent 交错推进;真正需要 CPU 并行的地方由子进程解决。多线程(worker_threads)在此没有收益,反增共享内存同步复杂度。

## 关联

- [[Anthropic]] — Claude Code 的开发者;其协议生态见 [[ModelContextProtocol]](Claude Desktop 是典型 MCP Host)
- [[EventLoop]] — in-process agent 并发的运行时模型
- [[Thread]] / [[Process]] — 单线程协程并发 vs 多进程隔离的取舍实例

## 来源

- [[claude-concurrency-io-session]]
