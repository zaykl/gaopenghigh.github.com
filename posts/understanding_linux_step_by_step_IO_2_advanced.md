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

通过`flock`结构中的`l_type`字段定义了2种锁：共享读锁（`F_RDLCK`）和
独占写锁（`F_WRLCK`）。从名字可以看出，等一个区域被某个进程加上了独占写锁时，
其他进程不能获取它的读锁或者写锁。

当两个进程相互等待对方持有并锁定的资源时，就会发生死锁。
文件锁与文件以及进程相关，文件close时，锁也被清空，fork出的子进程也不继承父进
程的锁。

## IO多路转接

“IO多路转接”是指这样一种技术：先构造好一张关于文件描述符的列表，然后调用一个函
数，直到这些描述符中的一个准备好IO时，函数才返回，并且告诉进程哪些描述符准备好
了。这类函数主要有`select`，`poll`。

### `select`函数

    #include <sys/select.h>
    int select(int maxfdp1,
               fd_set *restrict readfds,
               fd_set *restrict writefds,
               fd_set *restrict exceptfds,
               struct timeval *restrict tvptr);
    /* 返回值：准备好的描述符数，超时返回0，出错返回-1 */

简单地说，该函数告诉内核每个状态（读、写、异常）下我们关系的描述符有哪些，超时
时间是多少。然后内核返回给进程准备好的描述符数之和。

`fd_set`结构是“描述符集”，它相当于一个数组，为每一个可能的描述符保持了一位，最
大的位数可以设置。

`maxfdp1`表示“最大描述符加1”，即在三个`fd_set`中有效的最大描述符加上1的值，这样
内核只需要关心三个`fd_set`中的小于`maxfdp1`的那些描述符了。

### `poll`函数

    #include <poll.h>
    int poll(struct pollfd fdarray[], nfds_t nfds, int timeout);
    /* 返回值：准备好的描述符数，超时返回0，出错返回-1 */

与select不同的是，poll不为每个状态设置一个描述符集，而是统一用一个`pollfd`类型
的数组，里面放上我们关心的描述符以及所关心的状态。

    struct pollfd {
        int fd;        /* <0 to ignore */
        short events;  /* events of interest on fd */
        short revents; /* events that occurred on fd */
    };

常用的events和revents标志有：

    POLLIN    不阻塞地可读除高优先级以外的数据
    POLLPRT   不阻塞地可读高优先级数据
    POLLOUT   不阻塞地可写普通数据
    POLLERR   已出错
    POLLHUP   已挂断


通过这些值，进程告诉内核我们关心的是什么，然后内核通过设置revents来告诉进程实际
发生的是什么。

### `epoll`函数

使用select和poll不可比避免的一个操作就是遍历文件描述符，看看哪些准备好了。当进
程关注的文件描述符较多时，比如一个web服务器，遍历的开销就会变得非常大，以至于成
为性能的瓶颈。另外，select和poll支持的最大文件描述符数也有限制，即`FD_SETSIZE`
，这个数一般是1024。为了解决这些问题，于是有了epoll。和poll一样，epoll也是监控
大量的文件描述符，看他们是否准备好IO。

#### epoll的API

主要有三个API来创建和管理epoll实例：

* `epoll_create` 创建一个epoll实例，返回与这个实例关联的一个文件描述符。
* `epoll_ctl` 对于我们感兴趣的文件描述符，可以用`epoll_ctl`来注册。
    注册到一个epoll实例上哪些文件描述符组成一个集合，一般叫做“epoll set”。
* `epoll_wait` 等待IO事件，当前没有事件发生时，就阻塞调用它的进程。

#### “边缘触发”和“水平触发”

epoll支持“水平出发（Level-triggered）”或者“边缘触发（Edge-triggered）”。如果学学过“电子线路”之类的课，就很容易理解这两种触发方式。

我们可以把文件描述符没有准备好的状态设想为“0（低电平）”，已经准备好的状态设想为
“1（高电平）”。那么当文件从没准备好变成准备好了时，状态就从“0”变到“1”，如果有个
示波器，那图像就是一个直角矩形。这时，就造成一个“边缘触发”。而“水平触发”，就是
只要状态为“1”，则总处于触发状态。如果某次检测时有一个文件描述符准备好了，则它从
“0”变为“1”，此时无论是“边缘触发”还是“水平触发”都能检测到改描述符准备好了。但如
果一个文件描述符的状态一直是“1”，那么对于“水平触发”，它就能一直被检测到。

epoll的默认触发方式就是“水平触发”。

#### /proc 接口

epoll支持的最大文件描述符数并非没有限制。以下出自epoll的man手册。

The following interfaces can be used to limit the amount of kernel memory
consumed by epoll:

    /proc/sys/fs/epoll/max_user_watches (since Linux 2.6.28)

This specifies a limit on the total number of file descriptors that a user can
register across all epoll instances on the system.  The limit is per real user
ID.  Each registered file descriptor costs roughly 90 bytes on a 32-bit kernel,
and roughly 160 bytes on a 64-bit kernel.  Currently, the default value for
`max_user_watches` is 1/25 (4%) of the available low memory, divided by the
registration cost in bytes.

#### `epoll_create`

`epoll_create`函数创建一个epoll文件描述符：

    #include <sys/epoll.h>
    int epoll_create(int size);
    /* 成功返回文件描述符，出错返回-1，并通过设置errno指定错误类型 */

`epoll_create()`创建一个epoll实例。
从2.6.8版本的内核开始，参数`size`只要不是0，它事实上是被忽略的。
该函数返回与这个新的epoll实例关联的文件描述符，它之后所有对epoll实例的调，都要
用到这个文件描述符，当它不再需要时，应该通过`close`来关闭它。
当关联到一个epoll实例的所有文件描述符都被关闭时，内核就会销毁epoll实例，并释放
相关资源。

#### `epoll_ctl`

`epoll_ctl`是epoll文件描述符的控制接口。

    #include <sys/epoll.h>
    int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
    /* 成功时返回0，出错返回-1并设置errno */

它控制由epfd关联的epoll实例，op代表想要的操作，这些操作会作用在目标fd上。
上。op可能的值是：

    EPOLL_CTL_ADD    注册操作
    EPOLL_CTL_MOD    修改操作
    EPOLL_CTL_DEL    删除操作

`event`表示我们在`fd`上关心的IO类型

    typedef union epoll_data {
        void        *ptr;
        int          fd;
        uint32_t     u32;
        uint64_t     u64;
    } epoll_data_t;

    struct epoll_event {
        uint32_t     events;      /* Epoll events */
        epoll_data_t data;        /* User data variable */
    };

其中，`events`可以是下面这些值中的一个：

    EPOLLIN      可读
    EPOLLOUT     可写
    EPOLLRDHUP   Stream socket对关闭连接，或者关闭连接的写端
    EPOLLPRI     紧急数据可读
    EPOLLERR     出错
    EPOLLHUP     挂断
    EPOLLET      设置边缘触发
    EPOLLONESHOT 设置one-shot模式

#### `epoll_wait`

`epoll_wait`等待在一个epoll文件描述符上的IO事件。

    #include <sys/epoll.h>
    int epoll_wait(int epfd, struct epoll_event *events,
                   int maxevents, int timeout);
    /* 成功时返回可以IO的文件描述符数量，出错返回-1并设置errno */

* `epfd`表示我们关心的epoll实例。
* `events`是`struct epoll_event`类型的指针，它指向的数组存放准备好了的事件。
* `maxevents`只我们想要接受的最大事件的数目。可以设置为`struct epoll_event`数组
    `events`的长度。
* `timeout`表示超时时间，单位是ms。0表示不阻塞，-1表示永远阻塞。

#### epoll的使用

下面我们通过man手册中给出的标准示例看看epoll的用法。

listener 是一个飞阻塞的socket，通过listen系统调用在这个socket上进行请求的接收。
`do_use_fd()`函数使用已经准备好IO的文件描述符，read或者write，直到遇到`EAGAIN`
，然后记录它自己的状态，这样在下一次调用`do_use_fd()`时它能够从之前停止的的地方
继续read或write。

    #define MAX_EVENTS 10
    struct epoll_event ev, events[MAX_EVENTS];
    int listen_sock, conn_sock, nfds, epollfd;

    /* Set up listening socket, 'listen_sock' (socket(),
       bind(), listen()) */

    epollfd = epoll_create(10);    /* 这里的10事实上被忽略的 */
    if (epollfd == -1) {
        perror("epoll_create");
        exit(EXIT_FAILURE);
    }

    ev.events = EPOLLIN;          /* 对于listen_sock，我们关系它是否可读 */
    ev.data.fd = listen_sock;
    /* 在epollfd关联的epoll实例上注册listen_sock */
    if (epoll_ctl(epollfd, EPOLL_CTL_ADD, listen_sock, &ev) == -1) {
        perror("epoll_ctl: listen_sock");
        exit(EXIT_FAILURE);
    }

    for (;;) {
        nfds = epoll_wait(epollfd, events, MAX_EVENTS, -1);
        if (nfds == -1) {
            perror("epoll_pwait");
            exit(EXIT_FAILURE);
        }

        for (n = 0; n < nfds; ++n) {
            if (events[n].data.fd == listen_sock) {
                /* 检测到请求，建立新的socket来处理请求 */
                conn_sock = accept(listen_sock,
                                (struct sockaddr *) &local, &addrlen);
                if (conn_sock == -1) {
                    perror("accept");
                    exit(EXIT_FAILURE);
                }
                setnonblocking(conn_sock);
                ev.events = EPOLLIN | EPOLLET;
                ev.data.fd = conn_sock;
                /* 把新的socket注册到epollfd代表的epoll实例上 */
                if (epoll_ctl(epollfd, EPOLL_CTL_ADD, conn_sock,
                            &ev) == -1) {
                    perror("epoll_ctl: conn_sock");
                    exit(EXIT_FAILURE);
                }
            } else {
                do_use_fd(events[n].data.fd);
            }
        }
    }


## readv和writev函数

readv和writev函数用于在一次函数调用中读、写多个非连续缓冲区。有时也将这两个函数
成为**散布读（scatter read）**和**聚集写（gather write）**。如果使用read或者
write，完成同样的功能需要多次的系统调用。现在用readv和writev主要调用一次就OK。

    #include <sys/uio.h>
    ssize_t readv(int filedes, const struct iovec *iov, int iovcnt);
    ssize_t writev(int filedes, const struct iovec *iov, int iovcnt);
    /* 成功时返回已读、写的字节数，出错返回-1 */

这两个文件的第二个参数是指向iovec结构数组的指针，该结构表示一个缓冲区：

    struct iovec {
        void *iov_base;    /* starting address of bufffer */
        size_t iov_len;    /* size of buffer */
    };


## 存储映射IO

### 创建映射存储区

**存储映射IO（Memory-mapped IO）**使一个磁盘文件与存储空间中的一个缓冲区映射。
操作缓冲区就相当于操作磁盘上的文件。`mmap`函数实现这个功能。

    #include <sys/mman.h>
    void *mmap(void *addr, size_t len, int prot, int flag, int filedes,
               off_t off);
    /* 若成功则返回映射区的起始地址，若出错则返回MAP_FAILED */

* `addr` 参数表示映射区的起始地址。通常设为0，表示由系统选择该映射区的起始地址
* `len` 表示映射的字节数
* `off` 表示要映射字节在文件中的起始偏移量
* `prot` 表示对映射存储区的保护要求，其值可以下面这些值的任意组合：
* `fileds` 表示要被映射的文件的描述符
        PROT_HEAD     映射区可读
        PROT_WRITE    映射区可写
        PROT_EXEC     映射区可执行
        PROT_NONE     映射区不可访问
* `flag` 参数影响映射区的多种属性：
        MAP_FIXED     返回值必须等于addr，但一般把addr设为0
        MAP_SHARED    对存储区的操作相当于对文件write
        MAP_PRIVATE   对存储区的操作不会产生对文件的write
    其中，`MAP_SHARED`和`MAP_PRIVATE`两个参数必须选一个。

与映射存储区相关的有两个信号：

* `SIGSEGV` 试图访问对它不可用的存储区，或企图写数据到只读的存储区时产生该信号
* `SIGBUS` 访问映射区的某个部分，而在访问时这一部分实际上已经不存在时产生该信号

fork时子进程继承父进程的映射存储区。

### `mprotect`，`fsync`和`munmap`函数

使用`mprotect`函数可以修改映射存储区的权限：

    #include <sys/mman.h>
    int mprotect(void *addr, size_t len, int prot);

使用`fsync`函数可以冲洗映射存储区的数据到被映射的文件中：

    #include <sys/mman.h>
    int msync(void *addr, size_t len, int flags);

当进程终结或者调用了`munmap`函数时，映射存储区就被解除映射：

    #include <sys/mman.h>
    int munmap(caddr_t addr, size_t len);

需要注意的是，`munmap`函数不会自动把映射区的内容写到磁盘文件上。

### mmap和read/write的区别

（未完待续）
