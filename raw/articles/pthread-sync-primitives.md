# 线程同步与互斥(pthread 原语详解)

来源: https://blog.csdn.net/daaikuaichuan/article/details/82950711

## 一、同步与互斥的概念

现代操作系统基本都是多任务操作系统,即同时有大量可调度实体在运行。在多任务操作系统中,同时运行的多个任务可能:
- 都需要访问/使用同一种资源;
- 多个任务之间有依赖关系,某个任务的运行依赖于另一个任务。

【同步】:是指散布在不同任务之间的若干程序片断,它们的运行必须严格按照规定的某种先后次序来运行,这种先后次序依赖于要完成的特定的任务。最基本的场景就是:两个或两个以上的进程或线程在运行过程中协同步调,按预定的先后次序运行。比如 A 任务的运行依赖于 B 任务产生的数据。

【互斥】:是指散布在不同任务之间的若干程序片断,当某个任务运行其中一个程序片段时,其它任务就不能运行它们之中的任一程序片段,只能等到该任务运行完这个程序片段后才可以运行。最基本的场景就是:一个公共资源同一时刻只能被一个进程或线程使用,多个进程或线程不能同时使用公共资源。

## 二、互斥锁(同步)

在多任务操作系统中,多个任务可能都需要使用同一种资源。这个过程类似于,我在使用着打印机打印东西的同时(还没有打印完),别人刚好也在此刻使用打印机打印东西,如果不做任何处理的话,打印出来的东西肯定是错乱的。

线程里的这把锁——互斥锁(mutex),是一种简单的加锁的方法来控制对共享资源的访问,互斥锁只有两种状态,即上锁(lock)和解锁(unlock)。

【互斥锁的特点】:
1. **原子性**:把一个互斥量锁定为一个原子操作,这意味着操作系统(或pthread函数库)保证了如果一个线程锁定了一个互斥量,没有其他线程在同一时间可以成功锁定这个互斥量;
2. **唯一性**:如果一个线程锁定了一个互斥量,在它解除锁定之前,没有其他线程可以锁定这个互斥量;
3. **非繁忙等待**:如果一个线程已经锁定了一个互斥量,第二个线程又试图去锁定这个互斥量,则第二个线程将被挂起(不占用任何cpu资源),直到第一个线程解除对这个互斥量的锁定为止,第二个线程则被唤醒并继续执行,同时锁定这个互斥量。

【互斥锁的操作流程】:
1. 在访问共享资源后临界区域前,对互斥锁进行加锁;
2. 在访问完成后释放互斥锁上的锁;
3. 对互斥锁进行加锁后,任何其他试图再次对互斥锁加锁的线程将会被阻塞,直到锁被释放。

主要接口:

```c
#include <pthread.h>
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
int pthread_mutex_lock(pthread_mutex_t *mutex);      // 已上锁则调用者一直阻塞
int pthread_mutex_trylock(pthread_mutex_t *mutex);   // 已上锁则直接返回失败 EBUSY
int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex, const struct timespec *restrict abs_timeout); // 绑定阻塞时间
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

## 三、条件变量(同步)

与互斥锁不同,条件变量是用来等待而不是用来上锁的。条件变量用来自动阻塞一个线程,直到某特殊情况发生为止。通常条件变量和互斥锁同时使用。

条件变量使我们可以睡眠等待某种条件出现。条件变量是利用线程间共享的全局变量进行同步的一种机制,主要包括两个动作:
- 一个线程等待"条件变量的条件成立"而挂起;
- 另一个线程使"条件成立"(给出条件成立信号)。

【原理】:条件的检测是在互斥锁的保护下进行的。线程在改变条件状态之前必须首先锁住互斥量。如果一个条件为假,一个线程自动阻塞,并释放等待状态改变的互斥锁。如果另一个线程改变了条件,它发信号给关联的条件变量,唤醒一个或多个等待它的线程,重新获得互斥锁,重新评价条件。

主要接口:

```c
#include <pthread.h>
int pthread_cond_init(pthread_cond_t *cond, pthread_condattr_t *cond_attr);
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);        // 释放锁并阻塞等待条件为真
int pthread_cond_timewait(pthread_cond_t *cond, pthread_mutex *mutex, const timespec *abstime); // 超时等待
int pthread_cond_signal(pthread_cond_t *cond);      // 至少唤醒一个等待线程
int pthread_cond_broadcast(pthread_cond_t *cond);   // 唤醒所有等待线程
int pthread_cond_destroy(pthread_cond_t *cond);
```

### 虚假唤醒(spurious wakeup)

在采用条件等待时,我们使用的是:

```cpp
while(条件不满足)
{
   condition_wait(cond, mutex);
}
// 而不是:
if(条件不满足)
{
   condition_wait(cond, mutex);
}
```

这是因为可能会存在虚假唤醒的情况。也就是说,即使没有线程调用 condition_signal,原先调用 condition_wait 的函数也可能会返回。此时线程被唤醒了,但是条件并不满足,这个时候如果不对条件进行检查而往下执行,就可能会导致后续的处理出现错误。

虚假唤醒在 Linux 的多处理器系统中/在程序接收到信号时可能会发生。在 Windows 系统和 JAVA 虚拟机上也存在。在系统设计时应该可以避免虚假唤醒,但是这会影响条件变量的执行效率,而既然通过 while 循环就能避免虚假唤醒造成的错误,因此程序的逻辑就变成了 while 循环的情况。

## 四、读写锁(同步)

读写锁与互斥量类似,不过读写锁允许更改的并行性,也叫共享互斥锁。互斥量要么是锁住状态,要么就是不加锁状态,而且一次只有一个线程可以对其加锁。读写锁可以有3种状态:读模式下加锁状态、写模式加锁状态、不加锁状态。

一次只有一个线程可以占有写模式的读写锁,但是多个线程可以同时占有读模式的读写锁(允许多个线程读但只允许一个线程写)。

【读写锁的特点】:
- 如果有其它线程读数据,则允许其它线程执行读操作,但不允许写操作;
- 如果有其它线程写数据,则其它线程都不允许读、写操作。

【读写锁的规则】:
- 如果某线程申请了读锁,其它线程可以再申请读锁,但不能申请写锁;
- 如果某线程申请了写锁,其它线程不能申请读锁,也不能申请写锁。

读写锁适合于对数据结构的读次数比写次数多得多的情况。

主要接口:

```c
#include <pthread.h>
int pthread_rwlock_init(pthread_rwlock_t *rwlock, const pthread_rwlockattr_t *attr);
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);   // 申请读锁
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);   // 申请写锁
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock); // 非阻塞获取写锁
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
```

## 五、自旋锁(同步)

自旋锁与互斥量功能一样,唯一一点不同的就是互斥量阻塞后休眠让出cpu,而自旋锁阻塞后不会让出cpu,会一直忙等待,直到得到锁。

自旋锁在用户态使用的比较少,在内核使用的比较多!自旋锁的使用场景:锁的持有时间比较短,或者说小于2次上下文切换的时间。

自旋锁在用户态的函数接口和互斥量一样,把 pthread_mutex_xxx() 中 mutex 换成 spin,如:pthread_spin_init()。

## 六、信号量(同步与互斥)

信号量广泛用于进程或线程间的同步和互斥,信号量本质上是一个非负的整数计数器,它被用来控制对公共资源的访问。

编程时可根据操作信号量值的结果判断是否对公共资源具有访问的权限,当信号量值大于 0 时,则可以访问,否则将阻塞。PV 原语是对信号量的操作,一次 P 操作使信号量减1,一次 V 操作使信号量加1。

主要接口:

```c
#include <semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_wait(sem_t *sem);      // P 操作(减1)
int sem_trywait(sem_t *sem);   // 非阻塞 P 操作
int sem_post(sem_t *sem);      // V 操作(加1)
int sem_getvalue(sem_t *sem, int *sval);
int sem_destroy(sem_t *sem);
```

【信号量用于同步】:sem_g 初始0、sem_p 初始1,一个线程改字符、一个线程打印,交替推进。

【信号量用于互斥】:sem 初始1,printer 函数首尾夹 PV,两个线程打印字符串互不穿插。

参考链接(原文列出):
- https://blog.csdn.net/lianghe_work/article/details/47747413
- https://blog.csdn.net/lianghe_work/article/details/47747497
- https://blog.csdn.net/fulinus/article/details/39342371
- https://blog.csdn.net/lianghe_work/article/details/47775637
- https://www.cnblogs.com/wuchanming/p/4411908.html
- https://blog.csdn.net/qq_29350001/article/details/71709326
- https://blog.csdn.net/lianghe_work/article/details/47775741
