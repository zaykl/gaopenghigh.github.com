---
title: understanding_linux_step_by_step_IO_1
---

<link rel='stylesheet' href='/style/github2.css'/>

一步步理解Linux IO（1）
=====================

作者：[gaopenghigh](http://gaopenghigh.github.com)
，转载请注明出处。
[（原文地址）](http://gaopenghigh.github.io/posts/understanding_linux_step_by_step_IO_1.html)

------------------------------------------------

# 1. Linux文件IO的常用函数

Linux中做文件IO最常用到的5个函数是：`open`, `close`, `read`, `write`和`lseek`，不是ISO C的组成部分，这5个函数是不带缓冲的IO，也即每个read和write都调用了内核的一个系统调用。另外，dup函数和sync函数也经常用到，下面我们大致看一下这7个函数。

## `open`函数

    #include <fcntl.h>
    int open(const char *pathname, int oflag, ... /* mode_t mode */);
    /* 成功返回文件描述符， 失败返回-1 */

open函数打开或创建一个文件。pathname表示文件的路径，oflag表示打开的方式。通过控制oflag，可以规定打开文件的读写和追加模式、是否创建不存在的文件、是否截短为0、是否阻塞、读操作是否等待写操作完成、写操作是否等待物理IO操作完成等等。

## `close`函数

    #include <unistd.h>
    int close(int filedes);
    /* 成功返回0， 失败返回-1 */

关闭一个文件时，还会释放该文件上的所有记录锁。另外，当一个进程终止时，它会自动关闭所有它打开的文件。

## `lseek`函数

每个打开的文件都有一个“当前文件偏移量（current file offset）”，read和write操作都从这个当前文件偏移量开始。

    #include <unistd.h>
    off_t lseek(int filedes, off_t offset, int whence);
    /* 成功返回新的文件偏移量，出错返回-1 */

参数whence表示到底是从文件开始处、文件末尾还是文件当前偏移位置来计算offset。

## `read`函数

read函数从打开的文件中读取数据，数据存放到buf指向的空间中，最大读取的字节数为nbytes。

    #include <unistd.h>
    ssize_t read(int filedes, void *buf, size_t nbytes);
    /* 成功则返回读取到的字节数，若已到文件的结尾返回0，出错返回-1 */

## `write`函数

write函数向打开的文件写入数据，数据由buf指向的空间提供，最大写入的字节数为nbytes。

    #include <unistd.h>
    ssize_t write(int filedes, void *buf, size_t nbytes);
    /* 成功则返回写入的字节数，出错返回-1 */

## `dup`和`dup2`函数

这两个函数用来复制一个现存的文件描述符。

    #include <unistd.h>
    int dup(int filedes);
    int dup2(int fileds, int filedes2);
    /* 成功则返回新的文件描述符，出错返回-1 */

`dup`函数从当前可用的文件描述符中找一个最小的返回

`dup2`用filedes2指定新文件描述符的值。

## sync, fsync和fdatasync函数

这三个函数都是刷缓存的：

    #include <unistd.h>
    int fsync(int filedes);     /* 刷filedes指代的文件的缓存到磁盘 */
    int fdatasync(int filedes); /* 与fsync类似，但只刷数据，不刷属性 */
    /* 成功返回0，失败返回-1 */
    void sync(void);            /* 刷所有脏的缓存 */

----------------------

# 2. Linux中的标准IO库

## 2.1 缓冲

标准IO库提供缓冲功能，包括：

* 全缓冲。即填满IO缓冲区后才进行IO操作。
* 行缓冲。遇到换行符时执行IO操作。
* 无缓冲。

一般情况下，标准出错无缓冲。如果涉及终端设备，一般是行缓冲，否则是全缓冲。

可用`setbuf`和`setvbuf`函数设置缓冲类型已经缓冲区大小，使用fflush函数冲洗缓冲区。

## 2.2 打开流

使用`fopen`, `freopen`, `fdopen`三个函数打开一个流，这三个函数都返回FILE类型的指针。

    #include <stdio.h>
    FILE *fopen(const char *restrict pathname, const char *restrict type);
    FILE *freopen(const char *restrict pathname, const char *restrict type, 
                  FILE *restrict fp);
    FILE *dopen(int filedes, const char *type);
    /* 成功返回FILE类型指针，出错返回NULL */
    
其中，

* `fopen`打开一个指定的文件。
* `freopen`在一个指定的流上打开一个文件，比如在标准输出流上打开某文件。
* `dopen`打开指定的文件描述符代表的文件。常用于读取管道或者其他特殊类型的文件，因为这些文件不能直接用fopen打开。

`type`参数指定操作类型，入读写，追加等等。

## 2.3 关闭流

`fclose`函数关闭一个流：

    #include <stdio.h>
    int flose(FILE *fp);
    /* 成功返回0，出错返回EOF */

## 2.4 读和写流

### 每次一个字符的IO

    #include <stdio.h>
    /* 输入 */
    int getc(FILE *fp);
    int fgetc(FILE *fp);
    int getchar(void);
    /* 上面三个函数的返回值为int，因为EOF常实现为-1，返回int就能与之比较 */
    
    /* 判断出错或者结束 */
    int ferror(FILE *fp);
    int feof(FILE *fp);
    void clearerr(FILE *fp); /* 清除error或者eof标志 */
    
    /* 把字符压送回流 */
    int ungetc(intc FILE *fp);
    
    /* 输出 */
    int putc(int c, FILE *fp);
    int fputc(int c, FILE *fp);
    int putchar(int c);

### 每次一行的IO

    #include <stdio.h>
    /* 输入 */
    char *fgets(char *restrict buf, int n, FILE *restrict fp);
    char *gets(char *buf);
    /* gets由于没有指定缓冲区，所以有可能造成缓冲区溢出，要小心 */
    
    /* 输出 */
    int fputs(char *restrict buf, FILE *restrict fp);
    int puts(const char *buf);

### 二进制IO



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