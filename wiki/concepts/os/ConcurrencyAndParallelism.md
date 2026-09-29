---
title: "并发与并行 (Concurrency & Parallelism)"
type: concept
tags: [操作系统, 并发]
sources: [process-vs-thread, six-ipc-mechanisms]
last_updated: 2026-09-25
---

# 并发与并行 (Concurrency & Parallelism)

## 时间片轮转

基本事实：**单核 CPU 一个瞬间只能处理一个任务**。单核电脑能"同时"听音乐、浏览网页，靠的是 OS 给每个 [[Process]]（或 [[Thread]]）分配**时间片**（如 10ms），时间一到强制切换到下一个任务。CPU 每秒切换成千上万次，人类感觉不到顿挫（[[process-vs-thread]]）。

## 定义对比

| | 并发 (Concurrency) | 并行 (Parallelism) |
|---|---|---|
| 含义 | 单核轮流操作多任务，靠快速切换"看似同时" | 多核同时操作多任务，"真正同时" |
| 类比 | 一个厨师轮流炒两锅菜 | 两个厨师各炒一锅菜 |
| 硬件要求 | 单核即可 | 需多核/多 CPU |

## 异步性

多道批处理系统中多个进程并发执行，但因资源有限，进程"走走停停"，以不可预知的速度推进（异步性）——这正是需要 [[Semaphore]] 等同步机制保证执行顺序的原因（[[six-ipc-mechanisms]]）。

## 任务类型选型

- **CPU 密集型**（数据运算、算法执行）：建议多进程或少线程，避免 [[ContextSwitch]] 频繁切换浪费时间
- **IO 密集型**（读文件、网络请求）：建议多线程，线程等待 IO 时 CPU 切换到其他线程提升利用率

## 关联概念

- [[Process]] / [[Thread]]：时间片调度的对象
- [[ContextSwitch]]：并发得以实现的底层机制
