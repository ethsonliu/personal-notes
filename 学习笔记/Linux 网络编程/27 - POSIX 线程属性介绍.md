```c
// 线程属性结构如下：  
typedef struct  
{  
    int                   etachstate;      // 线程的分离状态  
    int                   schedpolicy;     // 线程调度策略  
    structsched_param     schedparam;      // 线程的调度参数  
    int                   inheritsched;    // 线程的继承性  
    int                   scope;           // 线程的作用域  
    size_t                guardsize;       // 线程栈末尾的警戒缓冲区大小  
    int                   stackaddr_set;   // 线程的栈设置  
    void*                 stackaddr;       // 线程栈的位置  
    size_t                stacksize;       // 线程栈的大小  
}pthread_attr_t;  
```

参考：<http://man7.org/linux/man-pages/man3/pthread_attr_init.3.html>

**线程的作用域（scope）**

作用域属性描述特定线程将与哪些线程竞争资源。线程可以在两种竞争域内竞争资源：

进程域（process scope）：与同一进程内的其他线程。

系统域（system scope）：与系统中的所有线程。一个具有系统域的线程将与整个系统中所有具有系统域的线程按照优先级竞争处理器资源，进行调度。

Solaris 系统，实际上，从 Solaris 9 发行版开始，系统就不再区分这两个范围。

**线程的绑定状态（binding state）**

轻进程（LWP：Light Weight Process）关于线程的绑定，牵涉到另外一个概念：轻进程（LWP：Light Weight Process）：轻进程可以理解为内核线程，它位于用户层和系统层之间。系统对线程资源的分配、对线程的控制是通过轻进程来实现的，一个轻进程可以控制一个或多个线程。

非绑定状态：默认状况下，启动多少轻进程、哪些轻进程来控制哪些线程是由系统来控制的，这种状况即称为非绑定的。

绑定状态：顾名思义，即某个线程固定的"绑"在一个轻进程之上。被绑定的线程具有较高的响应速度，这是因为 CPU 时间片的调度是面向轻进程的，绑定的线程可以
保证在需要的时候它总有一个轻进程可用。通过设置被绑定的轻进程的优先级和调度级可以使得绑定的线程满足诸如实时反应之类的要求。

**线程的分离状态（detached state）**

线程的分离状态决定一个线程以什么样的方式来终止自己。

非分离状态：线程的默认属性是非分离状态，这种情况下，原有的线程等待创建的线程结束。只有当 pthread_join() 函数返回时，创建的线程才算终止，才能释放自己占用的系统资源。

分离状态：分离线程没有被其他的线程所等待，自己运行结束了，线程也就终止了，马上释放系统资源。应该根据自己的需要，选择适当的分离状态。

线程分离状态的函数：`pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate)`。第二个参数可选为 PTHREAD_CREATE_DETACHED（分离线程）和 PTHREAD _CREATE_JOINABLE（非分离线程）。
这里要注意的一点是，如果设置一个线程为分离线程，而这个线程运行又非常快，它很可能在 pthread_create 函数返回之前就终止了，它终止以后就可能将线程号
和系统资源移交给其他的线程使用，这样调用 pthread_create 的线程就得到了错误的线程号。要避免这种情况可以采取一定的同步措施，最简单的方法之一是可以
在被创建的线程里调用 pthread_cond_timewait 函数，让这个线程等待一会儿，留出足够的时间让函数 pthread_create 返回。设置一段等待时间，是在多线程
编程里常用的方法。但是注意不要使用诸如 wait() 之类的函数，它们是使整个进程睡眠，并不能解决线程同步的问题。

**线程的优先级（priority）**

新线程的优先级为默认为 0。

新线程不继承父线程调度优先级(PTHREAD_EXPLICIT_SCHED)

仅当调度策略为实时（即 SCHED_RR或SCHED_FIFO）时才有效，并可以在运行时通过 pthread_setschedparam() 函数来改变，缺省为 0。

**线程的栈地址（stack address）**

POSIX.1 定义了两个常量 _POSIX_THREAD_ATTR_STACKADDR 和 _POSIX_THREAD_ATTR_STACKSIZE 检测系统是否支持栈属性。

也可以给 sysconf 函数传递 _SC_THREAD_ATTR_STACKADDR 或 _SC_THREAD_ATTR_STACKSIZE 来进行检测。

当进程栈地址空间不够用时，指定新建线程使用由 malloc 分配的空间作为自己的栈空间。通过 pthread_attr_setstackaddr 和 pthread_attr_getstackaddr 两
个函数分别设置和获取线程的栈地址。传给 pthread_attr_setstackaddr 函数的地址是缓冲区的低地址（不一定是栈的开始地址，栈可能从高地址往低地址增长）。

**线程的栈大小（stack size）**

当系统中有很多线程时，可能需要减小每个线程栈的默认大小，防止进程的地址空间不够用。当线程调用的函数会分配很大的局部变量或者函数调用层次很深时，可能需
要增大线程栈的默认大小。

函数 pthread_attr_getstacksize 和 pthread_attr_setstacksize 提供设置。

**线程的栈保护区大小（stack guard size）**

在线程栈顶留出一段空间，防止栈溢出。当栈指针进入这段保护区时，系统会发出错误，通常是发送信号给线程。该属性默认值是 PAGESIZE 大小，该属性被设置时，系
统会自动将该属性大小补齐为页大小的整数倍。

当改变栈地址属性时，栈保护区大小通常清零。

**线程的调度策略（schedpolicy）**

POSIX 标准指定了三种调度策略：先入先出策略 (SCHED_FIFO)、循环策略 (SCHED_RR) 和自定义策略 (SCHED_OTHER)。SCHED_FIFO 是基于队列的调度程序，对于每
个优先级都会使用不同的队列。SCHED_RR 与 FIFO 相似，不同的是前者的每个线程都有一个执行时间配额。SCHED_FIFO 和 SCHED_RR 是对 POSIX Realtime 的扩展。SCHED_OTHER 是缺省的调度策略。

新线程默认使用 SCHED_OTHER 调度策略。线程一旦开始运行，直到被抢占或者直到线程阻塞或停止为止。

SCHED_FIFO 如果调用进程具有有效的用户 ID 0，则争用范围为系统 (PTHREAD_SCOPE_SYSTEM) 的先入先出线程属于实时 (RT) 调度类。如果这些线程未被
优先级更高的线程抢占，则会继续处理该线程，直到该线程放弃或阻塞为止。对于具有进程争用范围 (PTHREAD_SCOPE_PROCESS)) 的线程或其调用进
程没有有效用户 ID 0 的线程，请使用 SCHED_FIFO，SCHED_FIFO 基于 TS 调度类。

SCHED_RR 如果调用进程具有有效的用户 ID 0，则争用范围为系统 (PTHREAD_SCOPE_SYSTEM)) 的循环线程属于实时 (RT) 调度类。如果这些线程未被优先级更高的线程抢占，并且这些线程没有放弃或阻塞，则在系统确定的时间段内将一直执行这些线程。对于具有进程争用范围 (PTHREAD_SCOPE_PROCESS) 的线程，请使用 SCHED_RR(基于 TS 调度类)。此外，这些线程的调用进程没有有效的用户 ID 0。

**线程并行级别（concurrency）**

应用程序使用 pthread_setconcurrency() 通知系统其所需的并发级别。
