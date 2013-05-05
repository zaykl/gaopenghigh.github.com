---
title: understanding_linux_step_by_step_interrupt
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

一步步理解Linux之中断和异常
===========================

作者：[gaopenghigh](http://gaopenghigh.github.com)
，转载请注明出处。
[（原文地址）](http://gaopenghigh.github.io/posts/understanding_linux_step_by_step_interrupt.html)

------------------------------------------------

# 中断和异常的概念

* **中断**：
    硬件通过中断来通知内核。中断是一种电信号，由硬件设备生成，并送入中断控制器
    的输入引脚中，中断控制器会给CPU发送一个电信号，CPU检测到这个信号，就中断当
    前的工作转而处理中断。每个中断都通过一个唯一的数字标志。这些中断值称为
    **中断请求（IRQ，Interrupt ReQuest）线**。

* **异常**：
    当CPU执行到由于编程失误而导致的错误指令（比如被0除）的时候，或者在执行期间
    出现踢输情况（如缺叶）而必须靠内核来处理的时候，处理器就产生一个异常。异常
    和中断类似，所以异常也叫“同步中断(asynchronous interrupt)”。内核对异常的处
    理大部分和对中断的处理一样。


## 中断描述符表

中断描述符表（Interrupt Descriptor Table, IDT）是一个系统表，它与每一个中断或异
常向量相联系，每一个向量在表中有相应的中断或异常处理程序的入口地址。IDT的地址存
放在`idtr`寄存器中。中断发生时，内核就从IDT中查询相应中断的处理信息。


## 异常处理

异常处理一般由三个部分组成：

1.  在内核堆栈中保存大多数寄存器的内容（汇编）。
2.  用高级的C函数处理异常。
3.  通过`ret_from_exception()`函数从异常处理程序退出。


## 中断处理

中断处理一般由四个步骤组成：

1.  在内核态堆栈中保存IRQ的值和寄存器的内容。
2.  为正在给IRQ线服务的PIC发送一个应答，这将允许PIC进一步发出中断。
3.  执行共享这个IRQ的所有设备的中断服务例程（ISR）。
4.  跳到`ret_from_intr()`的地址。

中断处理的示意图如下：

![](pictures/understanding_linux_step_by_step_interrupt_interrupt_handling.png)


### 中断处理程序

在相应一个特定中断时，内核会执行一个函数，这个函数就叫做
**中断处理程序（interrupt handler）**，或者叫做
**中断服务例程（interrupt service routine，ISR）**。

中断处理程序运行在中断上下文中，该上下文中的代码不可以阻塞。要注意，中断处理程
序执行的代码不是一个进程，中断处理程序比一个进程要“轻”。

每个中断和异常都会引起一个内核控制路径，而内核控制路径是可以任意嵌套的。也就是
说，一个中断处理程序可以被另一个中断处理程序“中断”。为了允许这样的嵌套，中断处
理程序就必须永不阻塞，换句话说，进程被中断，在中断程序运行期间，不能发生进程切
换。这是因为，一个中断产生时，内核会把当前寄存器的内容保存在内核态堆栈中，这个
内核态堆栈属于当前进程，嵌套中断时，上一个中断执行程序产生的寄存器内容同样也会
保存在该内核态堆栈，然后从嵌套的下一个中断恢复时，又从内核态堆栈中取出来放进寄
存器中。

一个内核控制路径嵌套执行的示例图如下：

![](pictures/understanding_linux_step_by_step_interrupt_nested_excution.png)



Linux中中断处理程序是无须重入的。当一条中断线上的handler正在执行时，这条中断线
在所有处理器上都会被屏蔽掉。

在/proc/interrupts中可以查看当前系统的中断统计信息。


### IRQ数据结构

每个IRQ都有自己的描述符`irq_desc_t`，描述符中有字段指向PIC对象，有字段指向ISR的
链表（因为每个IRQ线上可以注册多个中断处理程序）。所有的`irq_desc_t`合起来组成
`irq_desc`数组。示例图如下：

![](pictures/understanding_linux_step_by_step_interrupt_irq_descriptors.png)


### 上半部和下半部的概念

有时候中断处理需要做的工作很多，而中断处理程序的性质要求它必须在尽量短的时
间内处理完毕，所以中断处理的过程可以分为两部分或者两半（half）。中断处理程序属
于“上半部（top half）”--接受到一个中断，立刻开始执行，但只做有严格时限的工作。
能够被允许稍微晚一点完成的工作会放到“下半部（bottom half）中去，下半部不会马上
执行，而是等到一个合适的时机调度执行。也就是说，关键而紧急的部分，内核立即执行
，属于上半部；其余推迟的部分，内核随后执行，属于下半部。

比如说当网卡接收到数据包时，会产生一个中断，中断处理程序首要进行的工作是通知硬
件拷贝最新的网络数据包到内存，然后读取网卡更多的数据包。这样网卡缓存就不会溢出
。至于对数据包的处理和其他随后工作，则放到下半部进行。关于下半部的细节，我们后
面会讨论。


### 注册中断处理程序

驱动程序通过`request_irq()`函数注册一个中断处理程序：

    /* 定义在<linux/interrupt.h>中 */
    typedef irqreturn_t (*irq_handler_t)(int, void *);

    int request_irq(ussigned int irq,
                    irq_handler_t handler,
                    unsigned long flags,
                    const char *name,
                    void *dev);

参数解释如下：

* `irq` 要分配的中断号
* `handler` 是指向中断处理程序的指针
* `flags` 设置中断处理程序的一些属性，可能的值如下：

        IRQF_DISABLED       在本次中断处理程序本身期间，禁止所有其他中断。
        IRQF_SAMPLE_RANDOM  这个中断对内核的随机数产生源有贡献。
        IRQF_TIMER          该标志是特别为系统定时器的中断处理准备的。
        IRQF_SHARED         表明多个中断处理程序可以共享这条中断线。也就是说这
                            条中断线上可以注册多个中断处理程序，当中断发生时，
                            所有注册到这条中断线上的handler都会被调用。

* `name` 是与中断相关设备的ASCII文本表示
* `dev` 类似于一个cookie，内核每次调用中断处理程序时，都会把这个指针传递给它，
    指针的值用来表明到底是什么设备产生了这个中断，当中断线共享时，这条中断线上
    的handler们就可以通过dev来判断自己是否需要处理。


### 释放中断处理程序

通过`free_irq`函数注销相应的中断处理程序:

    void free_irq(unsigned int irq, void *dev);

参数和`request_irq`的参数类似。当一条中断线上注册了多个中断处理程序时，就需要
`dev`来说明想要注销的是哪一个handler。

---------------------------------------------------------------

# 下半部（bottom half）

有三种机制来执行下半部的工作：“软中断”，“tasklet”和“工作队列”。

软中断是一组静态定义的下半部接口，有32个，可以在所有处理器上同时执行--即使两个
类型相同也可以。

tasklet的实现基于软中断，但两个相同类型的tasklet不能同时执行。

工作队列则是先对要推后执行的工作排队，稍后在进程上下文中执行它们。

## 软中断（softirq）

### 软中断的实现

软中断实在编译期间静态分配的，由`softirq_action`结构表示：

    /* 在<linux/interrupt.h>中 */
    struct softirq_action {
        void (*action)(struct softirq_action *);
    };
    /* kernel/softirq.c中定义了一个包含有32个该结构体的数组 */
    static struct softirq_action softirq_vec[NR_SOFTIRQS];

每个被注册的软中断都占据该数组的一项，因此最多可能有32个软中断。

当内核运行一个软中断处理程序的时候，就会执行`softirq_action`结构中的`action`指
向的函数：

    my_softirq->action(my_softirq);

它把自己（整个`softirq_action`结构）的指针作为参数。

### 软中断的触发

软中断在被标记后才会执行，这标记的过程叫做**触发软中断（raising the softirq）**
。通常在中断处理程序中触发软中断。软中断的触发通过`raise_softirq()`进行。比如

    raise_softirq(NET_TX_SOFTIRQ);

触发网络子系统的软中断。

在下面这些时刻，软中断会被检查和执行：

* 从一个硬件中断代码处返回时
* 在ksoftirqd内核线程中（稍后会讲到）
* 在那些显式检查和执行带处理的软中断的代码中，比如网络子系统中


### 软中断的执行

软中断的状态通过一个位图来表示：第n位设置为1，表示第n个类型的软中断被触发，等待
处理。`local_softirq_pending()`宏返回这个位图。`set_softirq_pending()`宏则可对
位图进行设置或清零。

软中断在`do_softirq()`函数中执行，该函数遍历每一个软中断，如果处于被触发的状态
，则执行其处理程序，该函数的核心部分类似与这样：

    u32 pending;
    pending = local_softirq_pending();

    if (pending) {
        struct softirq_action *h;
        set_softirq_pending(0);           /* 把位图清零 */

        h = soft_vec;
        do {
            if (pending & 1)
                h-action(h);
            h++;
            pending >>= 1;      /* 位图向右移1位，原来第二位的现在在第一位 */
        } while (pending);
    }

需要注意的是，如果同一个软中断在它被执行的同时又被触发了，那么另外一个处理器可
以同时运行其处理程序。这意味着任何共享数据（甚至是仅在软中断处理程序内部使用的
全局变量）都需要严格的锁保护。因此，大部分的软中断处理程序，都通过采取单处理器
数据或其他的一些技巧来避免显式地加锁。


## tasklet

### tasklet的实现

tasklet基于软中断实现，事实上它使用的是`HI_SOFTIRQ`和`TASKLET_SOFTIRQ`这两个软
中断，通过`tasklet_struct`结构表示：

    /* 在<linux/interrupt.h>中 */
    struct tasklet_struct {
        struct tasklet_struct *next;       /* 链表中的下一个tasklet */
        unsigned long state;               /* tasklet的状态 */
        atomic_t count;                    /* 引用计数器 */
        void (*func)(unsigned long);       /* tasklet处理函数 */
        unsigned long data;                /* 给tasklet处理函数的参数 */
    };

其中，state的值只可以为0，`TASKLET_STATE_SCHED`（表示tasklet已被调度，正在准备
投入运行），和`TASKLET_STATE_RUN`（表示tasklet正在运行）。


### tasklet的调度

已经调度的tasklet（相当于触发了的软中断）存放在两个由`tasklet_struct`结构组成的
链表中：`tasklet_vec`和`tasklet_hi_vec`（表示高优先级的tasklet），分别通过
`tasklet_schedule()`和`tasklet_hi_schedule()`进行调度。


### ksoftirqd

在软中断处理程序中有时候会再次触发软中断，这样就有可能出现大量的软中断。这些重
新触发的软中断不会马上被处理，而是通过内核唤醒的一组内核线程来处理的。

每个处理器都有一组辅助处理软中断（包括了tasklet）的内核线程，名字叫做
`ksoftirqd/n`，其中n代表CPU的编号。这些内核线程以最低的优先级运行（nice值19），
这样就能避免它们和其它重要的任务抢夺资源。这些内核线程会执行类似与下面的循环：

    for (;;) {
        if (!softirq_pending(cpu))
            schedule();
        set_current_state(TASK_RUNNING);
        while (softirq_pending(cpu)) {
            do_softirq();
            if (need_resched())
                shcedule();
        }
        set_current_state(TASK_INTERRUPTIBLE);
    }


### preempt_count字段

在每个进程描述符的`thread_info`结构中有一个32位的字段叫`preempt_count`，它用来
跟踪内核抢占和内核控制路径的嵌套。利用`preempt_count`的不同区域表示不同的计数器
和一个标志。

    位        描述
    0~7       抢占计数器(max value = 255)
    8~15      软中断计数器(max value = 255)
    16~27     硬中断计数器(max value = 4096)
    28        PREEMPT_ACTIVE 标志

* “抢占计数器”记录显式禁用本地CPU内核抢占的次数，只有当这个计数器为0时才允许内
  核抢占。
* “软中断计数器”表示软中断被禁用的程度，同样，值为0时表示软中断可以被触发。
* “硬中断计数器”表示本地CPU上中断处理程序的嵌套数。`irq_enter()`宏递增它的值，
  `irq_exit()`宏递减它的值。


## 工作队列

**工作队列（work queue）**是另外一种将工作推后执行的形式，它可以把工作推后，交
由一个内核线程去执行。所以这些工作会在进程上下文中执行，并且运行重新调度和睡眠
。

### 工作的表示

一个工作用`work_struct`结构体表示：

    /* 定义在<linux/workqueue.h>中 */
    typedef void (*work_func_t)(struct work_struct *work);
    struct work_struct {
        atomic_long_t data;      /* 执行这个工作时的参数 */
        struct list_head entry;  /* 工作组成的链表 */
        work_func_t func;        /* 执行这个工作时调用的函数 */
    };

这些`work_struct`构成一个链表，工作执行完毕时，该工作就会从链表中移除。

### 工作者线程的表示

可以把一些工作放到一个队列里面，然后创建一个专门的内核线程来执行队列里的任务，
这些内核线程叫做**工作者线程（worker thread）**。但是大多数情况下不需要自己创建
worker thread，因为内核已经创建了一个默认的，叫做`events/n`，这里的n表示CPU的编
号。

“worker thread”使用`workqueue_struct`结构表示：

    struct workqueue_struct {
        struct cpu_workqueue_struct cpu_wq[NR_CPUS];
        struct list_head list;
        const char *name;
        int singlethread;
        int freezeable;
        int rt;
    };

一个“worker thread”表示一种类型的工作者线程，默认情况下只有event这一种类型的工
作者线程。然后每一个CPU上又有一个该类型的工作者线程，这就表现为`cpu_wq`数组，该
数组的每一项是`struct cpu_workqueue_struct`结构：

    struct cpu_workqueue_struct {
        spinlock_t lock;               /* 通过自旋锁保护该结构 */
        struct list_head worklist;     /* 工作列表 */
        wait_queue_head_t more_work;
        struct work_struct *current_struct;
        struct workqueue_struct *wq;   /* 关联工作队列结构 */
        task_t *thread;                /* 关联线程 */
    };

该结构体中的`wq`表明自己是什么类型的worker。

---------------------------------------------------------------

# 系统调用

## 什么是系统调用

**系统调用（System Call）**就是让用户进程与内核进行交互的一组接口，它在用户进程
和硬件设备之间添加了一个中间层。

以`printf()`为例，应用程序、C库和内核之间的关系是：

    -------------------------------------------------------------------------
    printf() ----> C库中的printf() ----> C库中的write() ----> write()系统调用
    --------------------------------------------------------------------------
    | 应用程序 |                   C库                    |      内核        |
    --------------------------------------------------------------------------

每个系统调用被赋予一个独一无二的系统调用号，系统调用号一旦分配就不能再变更。否
则编译好的程序就会崩溃。

## 系统调用处理程序`system_call()`

应用程序是通过**软中断**来通知内核对系统调用的进行使用的, 事实上是第128号IRQ。
也就是通过引发一个异常来促使系统切换到内核态去执行异常处理程序。此时的异常处理
程序实际上就是系统调用处理程序--`system_call()`。

至于使用的是哪个系统调用，就是通过系统调用号来判断。在陷入内核空间前，用户空间
把相应的系统调用号存入`exa`寄存器，`system_call`通过`exa`寄存器得知到底是哪个系
统调用。参数的传递也是通过寄存器，如果参数较多，则寄存器里面存的是指向这些参数
的用户空间地址的指针。


JH, 2013-05-05

----

参考资料：

* Man pages
* [UNIX环境高级编程](http://book.douban.com/subject/1788421/)
* [Linux内核设计与实现](http://book.douban.com/subject/6097773/)

----

<div id="disqus_thread"></div>
<script type="text/javascript">
/* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
    var disqus_shortname = 'gaopenghigh'; // required: replace example with your forum shortname

    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-40539766-1', 'github.com');
  ga('send', 'pageview');

</script>

