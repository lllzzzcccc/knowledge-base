---
title: "套接字 (Socket)"
type: concept
tags: [操作系统, 网络编程, IPC]
sources: [six-ipc-mechanisms]
last_updated: 2026-09-25
---

# 套接字 (Socket)

Socket 起源于 Unix，原意"插座"，计算机通信领域译为**套接字**，是计算机之间通信的一种约定/方式（[[six-ipc-mechanisms]]）。

## 定位

前五种 [[IPC]] 机制（[[Pipe]]、[[MessageQueue]]、[[SharedMemory]]、[[Semaphore]]、[[Signal]]）都只能用于**同一台主机**上的进程通信；Socket 是唯一能**跨网络与不同主机**进程通信的机制（同主机也能用）。

## 本质

Socket 是**应用层与 TCP/IP 协议族通信的中间软件抽象层**，本质是编程接口（API）：把复杂的 TCP/IP 协议族隐藏在接口后面，用户通过一组简单 API 即可实现网络连接。从网络层面看，Socket 是网络通信端点的抽象，包含五元信息：

1. 连接使用的协议
2. 本地主机 IP 地址
3. 本地进程的协议端口
4. 远地主机的 IP 地址
5. 远地进程的协议端口

## 对比其他机制

| | 同主机通信 | 跨网络通信 | 数据形态 |
|---|---|---|---|
| [[Pipe]] / [[MessageQueue]] / [[SharedMemory]] | ✅ | ❌ | 字节流 / 消息 / 内存 |
| [[Signal]] | ✅（通知） | ❌ | 无数据 |
| Socket | ✅ | ✅ | 字节流 |

## 关联概念

- [[IPC]]：六大机制之一，唯一跨网络者
- [[Process]]：网络通信两端仍是进程；[[Thread]] 共享进程的文件描述符（socket 也是文件描述符）
