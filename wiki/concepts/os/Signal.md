---
title: "信号 (Signal)"
type: concept
tags: [操作系统, IPC, Linux]
sources: [six-ipc-mechanisms, process-vs-thread]
last_updated: 2026-09-25
---

# 信号 (Signal)

**[[IPC]] 中唯一的异步通信机制**——可在任何时候发送给某个进程，通知其某个异步事件发生，迫使进程执行信号处理程序；处理完毕后，被中断进程恢复执行（[[six-ipc-mechanisms]]）。

⚠️ **信号和信号量（[[Semaphore]]）是完全不同的两个概念**，名字相似极易混淆。

## 来源

- **硬件来源**：键盘组合键，如 Ctrl+C 产生 SIGINT（终止进程）
- **软件来源**：kill 系列命令，如 `kill -9 1111` 产生 SIGKILL 让 PID 1111 立即结束

用户、内核和进程都能生成和发送信号。

## 进程级与线程级

[[Process]] 通过 `signal()` 注册的信号处理函数，对进程内所有 [[Thread]] 生效——如同车间的"紧急停机规则"，所有生产线听同一警报（[[process-vs-thread]]）。例如 C++ 示例中注册 SIGINT 处理函数后，按 Ctrl+C 所有线程停止运行。

## 对比其他 IPC 机制

[[Pipe]]、[[MessageQueue]]、[[SharedMemory]] 用于批量数据传输；信号不传输业务数据，只传达异步事件通知。

## 关联概念

- [[Semaphore]]：最易混淆的概念，务必区分
- [[Process]] / [[Thread]]：信号的接收与处理主体
- [[IPC]]：六大机制之一
