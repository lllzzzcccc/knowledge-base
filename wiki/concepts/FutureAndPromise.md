---
title: "Future 与 Promise"
type: concept
tags: [并发, 异步, Java, JavaScript]
sources: [claude-concurrency-io-session]
last_updated: 2026-09-26
---

# Future 与 Promise

两者抽象概念相同——都代表"一个未来才会产生的值",但**行为语义差很远**,核心差别在"阻塞 vs 非阻塞"([[claude-concurrency-io-session]]):

> **Future ≈"未来值的阻塞句柄";Promise ≈"未来值的回调协议"。** 前者问"现在好了没?没好我原地等着";后者说"好了叫我"。

## 关键差别

| | `Future.get()` | `Promise.then()` |
|---|---|---|
| 语义 | **阻塞**:线程挂起等结果 | **非阻塞**:注册回调,立刻返回 |
| 组合/链式 | 没有 | `.then().then().catch()` |
| 错误处理 | 抛 `ExecutionException` 手动拆 | 自动沿链传播到 `.catch()` |
| 取消 | `cancel()` | 标准没有(靠 `AbortController`) |
| 多任务编排 | 手动 ExecutorService | `Promise.all/race/allSettled` |

## 真正对应 Promise 的:CompletableFuture

```java
CompletableFuture.supplyAsync(() -> fetchUser())
    .thenApply(user -> user.name)              // ≈ .then(map)
    .thenCompose(name -> fetchProfile(name))   // ≈ .then(返回 Promise 的 fn)
    .exceptionally(e -> "fallback");           // ≈ .catch()
```

`thenApply` / `thenCompose` / `exceptionally` 的组合、回调、异常传播能力,原生 `Future` 完全没有。Guava 的 `ListenableFuture` 就是为给 `Future` 补回调能力而出现的——反证原生 Future 之原始。

## 为什么语言间形态不同

- **JS/Node 单线程事件循环**([[EventLoop]]):绝对不能用阻塞的 `get()`——一阻塞整个事件循环冻结。只能用 Promise 这种"注册回调"模型。
- **Java 多线程**:`get()` 阻塞的是一个线程,线程多,阻塞就阻塞,别的线程照跑——所以早期阻塞 Future 也能凑合。

## AIO 语境下的选择

AIO 的 `read()` 支持两种写法:`Future` 版本仍需阻塞 `.get()` 拿结果,反而失去异步意义;`CompletionHandler` 回调才是真正非阻塞、语义上最接近 `Promise.then()` 的写法(见 [[IOModels]])。

## 关联概念

- [[IOModels]] — AIO 的 Future/CompletionHandler 两种接口
- [[EventLoop]] — Promise 存在的理由:单线程不能阻塞
- [[Thread]] — 阻塞一个线程在多线程模型里是可接受的代价

## 来源

- [[claude-concurrency-io-session]]
