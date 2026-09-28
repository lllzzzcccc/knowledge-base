---
title: "Java 虚拟机 (JVM)"
type: concept
tags: [JVM, Java, 运行时]
sources: [java-memory-and-jmm]
last_updated: 2026-09-28
---

# Java 虚拟机 (JVM)

**JVM = 一套规范(JVMS) + 一个实现(如 HotSpot),本质上是一个执行字节码的用户态程序。** "虚拟"指虚拟出一套计算机架构(字节码指令集、虚拟内存区域、垃圾回收),让 Java 字节码跨平台运行——不是 VMware/VirtualBox 那种硬件虚拟机。

## 不是什么 / 是什么

| | 硬件虚拟机(VMware/VirtualBox) | JVM |
|---|---|---|
| 虚拟的对象 | 整台电脑(硬件) | 一套计算机架构(指令集 + 运行时) |
| 运行载体 | 宿主机上的进程,里面跑客户 OS | 宿主机上的一个普通进程 |
| 客户 | 整个操作系统 | 一门语言的字节码 |
| "跨平台"的含义 | 不同硬件的机器代码 | "一次编译,到处运行" |

## 启动过程：Java 程序 = 一个普通 OS 进程

`java Hello` → OS fork/exec 创建普通进程 → java 启动器(小 C 程序)加载 `libjvm.so` → JVM 启动(Bootstrap ClassLoader → 加载 `Hello.class` → 执行 `main`)。

- **JVM 不是独立进程**，以动态库形式住在你的进程里——不是"JVM 进程 + 程序进程"两个东西
- 一个 Java 程序 = 一个 OS 进程 = 一个 JVM 实例；跑多个 Java 程序就是多个进程、多套互不知晓的 JVM
- **JVM 里的 Java 线程 ≈ OS 线程**(HotSpot 1:1 映射)——[[Thread]] 的三类私有资源、[[ContextSwitch]] 的切换开销对 Java 线程全部成立
- JNI/native 方法在同一进程内以原生机器码直接执行

## 三大子系统

1. **类加载器(ClassLoader)**:加载 `.class` → 链接(验证/准备/解析)→ 初始化；Bootstrap/Extension/Application 三层委派模型
2. **执行引擎**:解释器逐条执行字节码 + **JIT**(热点代码编译成本地机器码，代码缓存存于方法区，见 [[JvmRuntimeMemory]])
3. **垃圾回收器(GC)**:自动管理堆(分代收集：新生代/老年代);`System.gc()` 只是建议

## 两层视图：JVM 是"租户",OS 用户空间是"房子"

- OS 只认一个进程、一套页表、两半空间([[AddressSpaceLayout]]);JVM 的堆/方法区/虚拟机栈是对**自己那半用户空间**的逻辑细分，OS 不知道这些区域的存在
- JVM 用 `mmap`/`malloc` 向内核申请内存(系统调用，[[UserModeVsKernelMode]]);内核空间那半 JVM 同样无权直碰
- 层级栈：**OS 两半空间 → JVM 运行时数据区([[JvmRuntimeMemory]]) → JMM 并发规范([[JavaMemoryModel]])**——越往上越抽象，最底层都是物理内存与页表

## 关联概念

- [[JvmRuntimeMemory]] — JVM 管理的运行时数据区(房间布局)
- [[JavaMemoryModel]] — 房间之上的使用规则
- [[AddressSpaceLayout]] — JVM 所建房子的 OS 地基
- [[Process]] — JVM 实例的容器；[[Thread]] — Java 线程即 OS 线程
- [[IOModels]] — NIO 直接内存是 JVM 绕过堆向 OS 直接申请的部分

## 来源

- [[java-memory-and-jmm]] + query 工作流提炼
