---
title: understanding_linux_step_by_step_system_call
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

一步步理解Linux之系统调用
=========================

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

应用程序是通过**软中断**来通知内核对系统调用的进行使用的。也就是通过引发一个异
常来促使系统切换到内核态去执行异常处理程序。此时的异常处理程序实际上就是系统调
用处理程序--`system_call()`。

至于使用的是哪个系统调用，就是通过系统调用号来判断。在陷入内核空间前，用户空间
把相应的系统调用号存入`exa`寄存器，`system_call`通过`exa`寄存器得知到底是哪个系
统调用。参数的传递也是通过寄存器，如果参数较多，则寄存器里面存的是指向这些参数
的用户空间地址的指针。

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

