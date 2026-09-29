---
title: "进程控制块 (PCB / Process Control Block)"
type: concept
tags: [操作系统, 进程, 内核]
sources: [process-vs-thread]
last_updated: 2026-09-28
---

# 进程控制块 (PCB / Process Control Block)

**PCB 是内核管理进程的"档案",驻留在内核内存中,不在进程的地址空间里。** 它由内核创建、读写和销毁,用户代码无法直接访问。

## 位置:档案柜 vs 房间

```
进程地址空间(进程"能看到的内存")   ← 代码/数据/堆/栈,见 [[Process]]
        │
        │  里面没有 PCB
        ▼
PCB 住在内核的直接映射区:内核虚拟地址 ↔ 物理内存一一对应的那段
Linux 实现:task_struct 结构体(几 KB)
         由内核 slab 分配器从 task_struct 专用缓存(kmem_cache)中分配
```

类比:**进程地址空间是你的房间,PCB 是物业的住户档案**——档案不在房间里,在内核的档案柜里。二者通过 `mm_struct` 指针关联:PCB 指着进程的页表,进程通过系统调用让内核碰自己的 PCB。

## 核心内容

| 字段 | 内容 | 对应知识库 |
|---|---|---|
| **进程状态** | 就绪 / 运行 / 阻塞 | [[Process]]、[[ConcurrencyAndParallelism]] |
| **上下文现场** | 寄存器值、PC、栈指针——切换时保存/恢复的对象 | [[ContextSwitch]] |
| **内存管理信息** | `mm_struct` 指针 → 进程的页表 | [[AddressSpaceLayout]];[[thread-essence]](线程 = 共享 mm 的 task_struct) |
| **调度信息** | 优先级、时间片、所在队列 | — |
| **资源清单** | 打开文件表(fd)、信号处理方式 | [[Thread]](信号处理即线程五类共享资源之一)、[[Signal]] |
| **身份** | PID、PPID、用户/组 | — |

## 生命周期

1. **fork()** → 内核复制父进程的 task_struct 创建新的(写时复制 mm,见 [[Process]])
2. **上下文切换** → 内核把当前寄存器现场**写入旧进程的 PCB**,从**新进程的 PCB 恢复**——[[ContextSwitch]] 那张"全景照"的实体
3. **退出** → 内核销毁它;僵尸进程期间 PCB 保留,等父进程通过 `wait` 收尸后才释放

## 用户怎么"看到"它

不能直接访问,但内核暴露了只读窗口:

- `cat /proc/<pid>/status`、`/proc/<pid>/task/` — procfs 把 task_struct 字段渲染成文件
- `ps`、`top`、`htop` — 遍历这些档案列出来

## Linux 实现细节

- **task_struct ≈ 线程**:Linux 中进程和线程共用 task_struct,区别仅在是否共享 `mm_struct`——这是 [[thread-essence]] 内核层结论的结构基础
- **配套的内核栈**:task_struct 的 `stack` 字段指向本进程的内核栈(8~16KB,系统调用期间使用,与用户栈分居两半空间,见 [[AddressSpaceLayout]])
- **current 宏**:内核靠它找到"当前正在运行的 task_struct"(x86 传统上借内核栈位置反推,新内核用 per-CPU 变量)——中断/系统调用一进来,内核第一件事就是回答"我是谁"

## 一句话总结

> **进程内存结构图画的是"进程拥有的资源"(地址空间);PCB 画的是"内核拥有的记录"(管理信息)。一个在用户空间的地基上,一个在内核空间的档案柜里。**

## 关联概念

- [[Process]] — PCB 所描述的进程本体
- [[ContextSwitch]] — PCB 读写最频繁的时刻
- [[thread-essence]] — task_struct 同时是线程的真相
- [[Thread]] / [[AddressSpaceLayout]] / [[Signal]]

## 来源

- [[process-vs-thread]](PCB 管理进程状态、上下文保存/恢复的出处)
