---
title: "线程同步与互斥(pthread 原语详解)"
type: source
tags: [线程, 同步, 互斥, pthread, Linux]
date: 2026-09-28
source_file: raw/articles/pthread-sync-primitives.md
---

## Summary

CSDN 文章,pthread 同步原语全家桶:互斥锁、条件变量、读写锁、自旋锁、信号量五类,每类给出特点、操作流程、API 与 demo。开篇先立同步与互斥的概念区分(同步=严格先后次序、互斥=公共资源同一时刻只一个使用),结尾用信号量同时演示"用于同步"(初值 0 交替推进)与"用于互斥"(初值 1 夹临界区)。是 [[ThreadSynchronization]] 概念页的直接来源。

## Key Claims

- **互斥锁三特性**:原子性(锁定是原子操作)、唯一性(解锁前他人无法锁定)、非繁忙等待(竞争者挂起不占 CPU)。
- **条件变量用于等待而非上锁**,必须与互斥锁配合:wait 时释放锁并阻塞;signal 唤醒至少一个等待者,broadcast 唤醒全部。
- **虚假唤醒(spurious wakeup)**:无 signal 也可能从 wait 返回,必须用 `while(条件不满足)` 而非 `if` 复检——Linux 多处理器/信号、Windows、JVM 上都存在。
- **读写锁三种状态**:读锁可多线程共存,写锁独占(写时禁止读);适合读远多于写的场景。
- **自旋锁**:与互斥锁功能相同,但阻塞时忙等待不让出 CPU;用户态少用、内核态常用;适用条件:持锁时间短于约 2 次上下文切换。
- **信号量**=非负整数计数器,PV 原语操作;初值 1 做互斥、初值按资源数做同步。

## Key Quotes

> "互斥锁只有两种状态,即上锁(lock)和解锁(unlock)" —— 互斥锁的极简定义

> "即使没有线程调用 condition_signal,原先调用 condition_wait 的函数也可能会返回" —— 虚假唤醒的定义

> "自旋锁……互斥量阻塞后休眠让出cpu,而自旋锁阻塞后不会让出cpu,会一直忙等待" —— 自旋锁与互斥锁的唯一差别

## Connections

- [[ThreadSynchronization]] — 本文的提炼页:pthread 五类同步原语
- [[Semaphore]] — 本文第六节的 OS 视角对应页(PV 语义、互斥两步走/同步三步走)
- [[Thread]] — 同步互斥存在的根因:线程共享数据区/堆区
- [[ContextSwitch]] — 自旋锁"持锁时间 < 2 次上下文切换"的取舍依据

## Contradictions

- 无。与 [[Semaphore]] 完全互补:彼页讲 PV 语义与信号量理论,本页讲 pthread 全套原语的工程形态(含 demo 与 API)。
