---
title: "线程同步原语 (Thread Synchronization Primitives)"
type: concept
tags: [线程, 同步, 互斥, pthread, Linux]
sources: [pthread-sync-primitives]
last_updated: 2026-09-28
---

# 线程同步原语 (Thread Synchronization Primitives)

用户态(pthread)的同步互斥工具箱,五类原语解决同一根因:[[Thread]] 共享数据区/堆区,必须重建秩序。理论根基(PV 语义、互斥两步走/同步三步走)见 [[Semaphore]];本页是工程形态([[pthread-sync-primitives]])。

## 五类原语速览

| 原语 | 干什么 | 等待方式 | 适用场景 |
|---|---|---|---|
| **互斥锁 mutex** | 临界区一次一个线程 | 挂起休眠(非繁忙等待) | 通用互斥 |
| **条件变量 cond** | 等"条件成立"(配合锁) | wait 释放锁并阻塞 | 生产者-消费者类时序 |
| **读写锁 rwlock** | 读共享/写独占 | 同锁 | 读多写少 |
| **自旋锁 spinlock** | 同互斥锁 | **忙等待不让 CPU** | 持锁时间 < ~2 次上下文切换 |
| **信号量 sem** | 计数器控资源;PV 操作 | 挂起 | 互斥(初值1)与同步(初值0)统一 |

## 互斥锁:三特性

1. **原子性**:锁定是原子操作,同一时刻只有一个线程能锁成功
2. **唯一性**:解锁前他人无法锁定
3. **非繁忙等待**:竞争者挂起,**不占 CPU**(对比自旋锁)

变体:`trylock`(已锁立即返回 EBUSY,非阻塞)、`timedlock`(带超时的阻塞)。

## 条件变量:用于等待,不是上锁

与互斥锁**必须配合使用**(条件检测在锁保护下进行):

- `pthread_cond_wait`:**释放锁**并阻塞,被唤醒后重新抢锁、重新评估条件
- `pthread_cond_signal` 唤醒至少一个等待者;`broadcast` 唤醒全部

**虚假唤醒(spurious wakeup)**:无 signal 也可能从 wait 返回(Linux 多处理器/信号、Windows、JVM 均存在),所以必须:

```cpp
while (条件不满足) { pthread_cond_wait(&cond, &mutex); }  // ✅ 复检
// 不是 if —— 醒来时条件可能仍不满足
```

## 读写锁:三种状态

读锁可共存(多读并发),写锁独占(写时禁读禁写)。**适合读次数远多于写次数**的数据结构。变体 `trywrlock` 非阻塞取写锁。

## 自旋锁:用 CPU 换等待

与互斥锁功能相同,唯一差别:阻塞时**忙等待、不让出 CPU**。适用判据:**持锁时间短于约 2 次上下文切换**([[ContextSwitch]]——自旋浪费的 CPU < 切换开销才划算)。用户态少用,内核态常用(中断上下文不能睡眠,只能自旋)。接口:`pthread_spin_init/lock/unlock`(把 mutex 换成 spin)。

## 信号量:一个计数器统一互斥与同步

非负整数计数器,`sem_wait`(P,减1,>0 可访问否则阻塞)、`sem_post`(V,加1)。初值 1 → 互斥守门;初值 0/资源数 → 同步接力(详见 [[Semaphore]] 的两步走/三步走)。

## 选型直觉

- 纯互斥 → mutex;临界区极短且在中断/内核态 → spinlock
- 读多写少 → rwlock
- "等条件"而非"抢资源" → cond + mutex
- 需要计数/多资源池/跨进程序列 → 信号量

## 关联概念

- [[Semaphore]] — PV 语义与信号量理论(本文的 OS 课视角)
- [[Thread]] — 共享资源是同步互斥存在的根因
- [[JavaMemoryModel]] — synchronized/volatile 是这些原语在 Java 层的规范表达(监视器锁 hb 规则)
- [[Goroutine]] — channel/mutex 是同一问题的 Go 形态

## 来源

- [[pthread-sync-primitives]]
