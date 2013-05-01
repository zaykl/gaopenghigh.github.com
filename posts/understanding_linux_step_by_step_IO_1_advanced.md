---
title: understanding_linux_step_by_step_IO_1_advanced
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

一步步理解Linux IO（1）--高级IO
=========================================

作者：[gaopenghigh](http://gaopenghigh.github.com)
，转载请注明出处。
[（原文地址）](http://gaopenghigh.github.io/posts/understanding_linux_step_by_step_IO_1_advanced.html)

------------------------------------------------

# 高级IO

## 记录锁（record locking）

记录锁应该叫“字节范围锁”，因为它就是把文件中的某个部分加上锁。
使用`fcntl`函数操作记录锁：

    #include <fcntl.h>
    int fcntl(int filedes, int cmd, ... /* struct flock *flockptr */ );
    /* 成功的返回依赖cmd，出错返回-1 */

对于记录锁，`cmd`可以是`F_GETLK`，`F_SETLK`或者`F_SETLKW`。

第三个参数`flockptr`是一个指向`flock`结构的指针，该结构如下：

    struct flock {
        short l_type;    /* F_RDLCK, F_WRLCK or F_UNLCK */
        off_t l_start;   /* offset in bytes, relative to l_whence */
        short l_whence;  /* SEEK_SET, SEEK_CUR or SEEK_END */
        off_t l_leng;    /* length in bytes, 0 means lock to EOF */
        pid_t l_pid;     /* returned with F_GETLK */
    };

通过`flock`结构中的`l_type`字段定义了2种锁：共享读锁（`F_RDLCK`）和独占写锁（`F_WRLCK`）。从名字可以看出，等一个区域被某个进程加上了独占写锁时，其他进程不能获取它的读锁或者写锁。

当两个进程相互等待对方持有并锁定的资源时，就会发生死锁。文件锁与文件以及进程相关，文件close时，锁也被清空，fork出的子进程也不继承父进程的锁。

## IO多路转接

“IO多路转接”是指这样一种技术：先构造好一张关于文件描述符的列表，然后调用一个函数，直到这些描述符中的一个准备好IO时，函数才返回，并且告诉进程哪些描述符准备好了。这类函数主要有`select`，`poll`。

### `select`函数

    #include <sys/select.h>
    int select(int maxfdp1,
               fd_set *restrict readfds,
               fd_set *restrict writefds,
               fd_set *restrict exceptfds,
               struct timeval *restrict tvptr);
    /* 返回值：准备好的描述符数，超时返回0，出错返回-1 */

简单地说，该函数告诉内核每个状态（读、写、异常）下我们关系的描述符有哪些，超时时间是多少。然后内核返回给进程准备好的描述符数之和。

`fd_set`结构是“描述符集”，它相当于一个数组，为每一个可能的描述符保持了一位，最大的位数可以设置。

`maxfdp1`表示“最大描述符加1”，即在三个fd_set中有效的最大描述符加上1的值，这样内核只需要关心三个`fd_set`中的小于`maxfdp1`的那些描述符了。

### `poll`函数

    #include <poll.h>
    int poll(struct pollfd fdarray[], nfds_t nfds, int timeout);
    /* 返回值：准备好的描述符数，超时返回0，出错返回-1 */

与select不同的是，poll不为每个状态设置一个描述符集，而是统一用一个`pollfd`类型的数组，里面放上我们关心的描述符以及所关心的状态。

    struct pollfd {
        int fd;        /* <0 to ignore */
        short events;  /* events of interest on fd */
        short revents; /* events that occurred on fd */
    };

 
