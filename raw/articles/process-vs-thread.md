# 进程 vs 线程：从原理到区别，一次讲清楚

来源: https://zhuanlan.zhihu.com/p/2012582824830600068

面试官：“聊聊进程和线程的区别吧”，大多数人都能把八股文背出来： “一个是资源分配单位，一个是调度执行单位”。

这问题背后，藏的是操作系统资源调度的核心逻辑，远没表面那么简单。

如果把操作系统比作一座大型工厂，进程就是工厂里独立的生产车间，而线程则是车间里各司其职的工人。带着这个比喻往下看，很多抽象概念你会瞬间清晰。

Part1 进程的本质

1.1、进程的 “激活” 过程

当你双击打开一个软件、敲下指令运行一段程序时，操作系统不会直接让程序跑起来，它会先给这个程序划一块 “专属地盘”—— 也就是独立的地址空间。这块地盘被分成了三个核心区域：

代码区：存放程序的执行指令，相当于车间里的生产手册，所有操作都得按手册来；
数据区：存储全局变量、静态变量等公共数据，好比车间里的公共物料架；
堆栈区：栈区负责函数调用的临时数据，堆区用于动态分配内存，就像工人手边的临时工具台和可申请的备用物料箱。

有了这块独立地盘，程序才算真正拥有了 “运行资格”，此时它就从静态的代码变成了动态的进程。我们可以给进程一个通俗定义：进程是程序在专属地址空间内的动态执行实例，它带着 OS 分配的独立资源，是系统资源分配的基本单位。

结合工厂类比，进程有三个核心特征很好理解：

动态性与静态性的区别：程序是写在硬盘里的 “生产手册”，是静态的；而进程是手册被拿到车间、工人按手册开工的过程，是动态的，程序运行则进程生，程序停止则进程灭；
资源分配的独立单位：每个车间（进程）都有自己的物料、工具和生产区域，OS 会给每个车间独立分配资源（CPU 资源除外），车间之间的物料互不流通，对应进程的地址空间相互隔离，一个进程崩溃不会影响其他进程；
调度的基本单元（非 CPU 层面）：OS 会统筹各个车间的开工顺序，但不会直接管车间里工人的分工，这也为后续线程的出现埋下了伏笔。
1.2、进程上下文切换

进程切换是操作系统的核心能力（如从 “浏览器” 切到 “音乐软件”），而实现 “无缝切换” 的关键，就是进程上下文—— 它是进程运行状态的完整快照，包含四类必须保存的信息：

上下文组成	技术定义	工厂类比（车间切换）
CPU 寄存器值	累加器（存计算结果）、程序计数器（PC，存下条指令地址）、栈指针（存栈顶位置）等	记录车间设备参数（如机床转速、卡尺当前读数）
进程状态	由 PCB 管理，含 “就绪 / 运行 / 阻塞” 等状态（如进程因等待网络请求进入 “阻塞” 态）	记录车间生产进度（如 “待料中”“加工中”“已完工”）
地址空间信息	页表 / 段表（映射虚拟内存到物理内存）、内存权限（如代码区只读、数据区可读写）	记录车间布局（如 “物料区在东、设备区在西”）+ 区域规则（如 “物料区禁止吸烟”）
内核栈与用户栈内容	内核栈存系统调用参数（如open函数的文件路径），用户栈存函数局部变量 / 调用参数	记录车间管理员的调度笔记（内核栈）、工人的操作记录（用户栈）
进程上下文的作用：为什么切换进程开销大？

以 “浏览器切到音乐软件” 为例，操作系统会执行两步操作：

保存浏览器进程上下文：将浏览器的寄存器值、页表、栈数据全部写入 PCB—— 相当于给浏览器车间拍 “全景照”，连设备参数、物料位置都不放过；
加载音乐进程上下文：从音乐进程的 PCB 中读取快照，恢复寄存器值、重建内存映射 —— 相当于按 “全景照” 还原音乐车间，继续之前的播放进度。

正因为进程上下文包含 “地址空间信息” 这类重量级数据，进程切换的开销通常是线程切换的 10~100 倍—— 这也是线程存在的核心原因。




Part2 线程的由来与特性

进程虽能实现 “独立运行”，但切换开销太大。为解决 “轻量化执行” 需求，线程（Thread）作为 “进程内的执行分支” 应运而生，类比车间里的 “生产线”—— 共享车间资源，仅保留专属执行工具。

2.1、线程的本质

线程不单独拥有资源，而是 “复用” 所属进程的大部分资源，仅保留三类 “执行必需的私有资源”（确保独立运行）：

运行栈：存函数局部变量、调用参数（如void func(int a)中的a），每个线程有独立栈空间，避免数据干扰；
程序计数器（PC）：记录当前线程下一条指令地址，确保切换后能 “续上” 执行；
部分寄存器：如通用寄存器（存临时计算结果）、栈指针（指向栈顶），属于线程私有，不与其他线程共享。

这三类私有资源统称线程上下文—— 对比进程上下文，它不含 “地址空间信息”，因此切换时只需保存 / 加载少量数据，开销极低。




2.2、核心技术点：线程共享的进程资源

“线程共享进程资源” 是核心特性，但具体共享哪些、如何验证？下面分五类拆解

（1）共享代码区：所有线程可执行同一函数

代码区存编译后的机器指令（如thread_func函数），同一进程内的线程可直接调用任意函数 —— 类比所有生产线共用一本 “设计图”，无需单独复印。

代码：

#include <iostream>
#include <thread>
using namespace std;
// 代码区的函数，t1、t2均可调用
void thread_func(int thread_id) {
    cout << "线程" << thread_id << "执行代码区函数" << endl;
}
int main() {
    thread t1(thread_func, 1); // 线程1调用thread_func
    thread t2(thread_func, 2); // 线程2调用同一函数
    t1.join();
    t2.join();
    return 0;
}

输出：

线程1执行代码区函数
线程2执行代码区函数
（2）共享数据区：全局变量、静态变量多线程可见

数据区存全局变量（如int global_var）和静态变量（如static int static_var），所有线程访问的是 “同一内存地址”—— 类比车间的 “公共物料架”，一个生产线用了，其他线看到的数量会减少。

代码：

#include <iostream>
#include <thread>
#include <mutex>
using namespace std;
mutex mtx; // 互斥锁：避免cout输出混乱
int global_var = 10; // 数据区全局变量（线程共享）
void modify_global(int thread_id) {
    lock_guard<mutex> lock(mtx);
    global_var += thread_id; // 线程1加1，线程2加2
    cout << "线程" << thread_id << "修改后，global_var=" << global_var << endl;
}
int main() {
    thread t1(modify_global, 1);
    thread t2(modify_global, 2);
    t1.join();
    t2.join();
    cout << "主线程读取global_var=" << global_var << endl; // 主线程也能访问
    return 0;
}

输出：

线程1修改后，global_var=11
线程2修改后，global_var=13
主线程读取global_var=13
（3）共享堆区：动态分配内存多线程可访问

堆区是通过new/malloc动态分配的内存（如int* p = new int(10)），只要线程持有指针，就能读写该内存 —— 类比车间的 “备用物料库”，所有生产线知道库位（指针）就能取用。

代码：

#include <iostream>
#include <thread>
#include <mutex>
using namespace std;
mutex mtx;
int* heap_var = new int(10); // 堆区内存（线程共享）
void modify_heap(int thread_id) {
    lock_guard<mutex> lock(mtx);
    *heap_var += thread_id * 5; // 线程1加5，线程2加10
    cout << "线程" << thread_id << "修改后，heap_var=" << *heap_var << endl;
}
int main() {
    thread t1(modify_heap, 1);
    thread t2(modify_heap, 2);
    t1.join();
    t2.join();
    delete heap_var; // 堆区需手动释放（进程退出会自动回收）
    return 0;
}

输出：

线程1修改后，heap_var=15
线程2修改后，heap_var=25
（4）共享文件描述符：打开的文件 / 网络连接多线程可用

进程打开的文件（如FILE* f = fopen("log.txt", "w")）、网络连接（如socket）会被分配 “文件描述符”（整数标识），该描述符属于进程，所有线程可通过它读写 —— 类比车间的 “公共打印机”，所有生产线都能使用。

场景示例：

#include <iostream>
#include <thread>
#include <cstdio>
#include <mutex>
using namespace std;
mutex mtx;
FILE* log_file = fopen("app.log", "w"); // 进程打开的文件（共享）
void write_log(int thread_id) {
    lock_guard<mutex> lock(mtx);
    fprintf(log_file, "线程%d：写入日志\n", thread_id); // 多线程共享文件描述符
}
int main() {
    thread t1(write_log, 1);
    thread t2(write_log, 2);
    t1.join();
    t2.join();
    fclose(log_file);
    return 0;
}

查看*app.log内容：

线程1：写入日志
线程2：写入日志
（5）共享信号处理方式：进程注册的信号回调多线程生效

进程通过signal函数注册的信号处理逻辑（如处理Ctrl+C的SIGINT信号），对所有线程生效 —— 类比车间的 “紧急停机规则”，所有生产线听到警报后都会按同一规则停机。

场景示例：

#include <iostream>
#include <thread>
#include <signal.h>
#include <unistd.h>
using namespace std;
// 进程注册的信号处理函数（所有线程生效）
void sig_handler(int sig) {
    cout << "收到信号" << sig << "，所有线程停止运行" << endl;
    exit(0);
}
void thread_task() {
    while (true) {
        sleep(1);
        cout << "线程运行中..." << endl;
    }
}
int main() {
    signal(SIGINT, sig_handler); // 注册SIGINT信号（Ctrl+C触发）
    thread t1(thread_task);
    thread t2(thread_task);
    t1.join();
    t2.join();
    return 0;
}

操作与输出：

运行程序，线程持续输出 “线程运行中...”；
按Ctrl+C触发SIGINT信号，所有线程停止，输出 “收到信号 2，所有线程停止运行”。




Part3 关系和区别

3.1、本质区别（核心定位）

进程：操作系统资源分配的基本单位（给车间分地盘、物料、图纸）；
线程：处理器（CPU）任务调度和执行的基本单位（给工人分配生产线）。

3.2、包含关系（层级归属）

一个进程至少包含一个线程（一个车间至少有一条生产线才能开工）；
线程是进程的组成部分，因开销远低于进程，被称为 “轻权进程” 或 “轻量级进程”。

3.3、资源开销（成本差异）

进程：每个进程有独立地址空间（专属车间），创建、销毁、切换时需处理全套资源（地盘、物料、图纸），开销大；
线程：同一进程内的线程共享地址空间（共用车间资源），仅私有运行栈和程序计数器（专属操作台和记录本），创建、切换、销毁开销小。

3.4、影响关系（稳定性差异）

进程：地址空间相互隔离（车间独立），一个进程崩溃后，在保护模式下其他进程不受影响（一个车间塌了，其他车间正常生产）；
线程：共享进程资源（共用车间物料），一个线程崩溃可能导致整个进程被 OS 杀掉（一条生产线出故障，可能毁了整个车间的物料，导致车间停工），因此多进程比多线程更健壮。
Part4 并发、并行与任务类型

有了 “车间 - 生产线 - 工人” 模型，咱们再拆解两个高频考点：并发 vs 并行，以及 CPU/IO 密集型任务。

4.1、并发与并行：单 CPU 如何 “同时” 多任务？

一个基本事实：单核 CPU 在一个瞬间只能处理一个任务。但为什么我们用单核心电脑时，能同时听音乐、浏览网页？答案是 ——时间片轮转调度。

OS 会给每个进程（或线程）分配一个 “时间片”（比如 10ms），即 CPU 每次执行该任务的最长时间。时间一到，不管任务是否完成，OS 都会强制把 CPU 切换到下一个任务。就像工人（单核 CPU）给每条生产线（线程）分配 10 分钟操作时间，到点就换线 —— 虽然每个瞬间只操作一条线，但切换速度极快（CPU 每秒切换成千上万次），人类完全感觉不到顿挫，误以为是 “同时运行”。

这就区分了两个概念：

并发：单工人（单核 CPU）轮流操作多生产线（线程），靠快速切换实现 “看似同时”；
并行：多工人（多核 CPU）同时操作多生产线（线程），实现 “真正同时”。

比如一个厨师轮流炒两锅菜（并发），两个厨师各炒一锅（并行）—— 这也是面试中区分两者的核心要点。

4.2、任务类型：CPU 密集型 vs IO 密集型
CPU 密集型任务（如数据运算、算法执行）：建议用 “多进程” 或 “少线程”—— 避免频繁线程切换浪费时间（单核下多线程反而变慢）；
IO 密集型任务（如读文件、网络请求）：建议用 “多线程”—— 线程等待 IO 时，CPU 可切换到其他线程，提升利用率（如一个线程等网络响应时，另一个线程处理本地逻辑）。




Part5 代码实操

直观感受进程与线程的开销差异

代码实现（Linux 环境）
#include <iostream>
#include <thread>
#include <unistd.h>
#include <sys/wait.h>
#include <chrono>
using namespace std;
const int LOOP_NUM = 10000; // 任务循环次数（模拟切换频率）
// 线程任务：空循环（仅用于消耗时间片）
void thread_task() {
    for (int i = 0; i < LOOP_NUM; i++);
}
// 进程任务：空循环（与线程任务逻辑一致）
void process_task() {
    for (int i = 0; i < LOOP_NUM; i++);
    exit(0); // 子进程执行完退出
}
int main() {
    // 1. 统计线程切换耗时
    auto start_thread = chrono::high_resolution_clock::now();
    thread t1(thread_task);
    thread t2(thread_task);
    t1.join();
    t2.join();
    auto end_thread = chrono::high_resolution_clock::now();
    auto dur_thread = chrono::duration_cast<chrono::microseconds>(end_thread - start_thread).count();
    // 2. 统计进程切换耗时
    auto start_process = chrono::high_resolution_clock::now();
    pid_t pid1 = fork(); // 创建子进程1
    if (pid1 == 0) process_task();
    pid_t pid2 = fork(); // 创建子进程2
    if (pid2 == 0) process_task();
    waitpid(pid1, nullptr, 0); // 等待子进程1退出
    waitpid(pid2, nullptr, 0); // 等待子进程2退出
    auto end_process = chrono::high_resolution_clock::now();
    auto dur_process</doubaocanvas>

整理了一波超实用的技术干货文章，刚好覆盖Linux C/C++的各种成长路径～

还在纠结 C/C++ 要不要学、值不值得深耕？

· 想做 Linux C/C++ 后端？给你大厂标准进阶路线

· 想入门 音视频流媒体？零基础也能快速上手

· 想搞 桌面 / 嵌入式开发？Qt 一条龙从入门到实战

· 想冲 Linux 内核？硬核指南帮你吃透底层

· 备战面试？1000 道高频八股直接刷

· 求职没 项目？一个项目，帮你把 C/C++ 学透、面稳、拿好 offer。