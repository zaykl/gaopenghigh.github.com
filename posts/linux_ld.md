---
title: linux_ld
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
</head>

Linux共享库学习笔记 
=================

`ldd`命令察看动态链接程序依赖了哪些库： 

    $ ldd /bin/ln  
    linux-gate.so.1 =>  (0x00990000)  
    libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x00c6b000)  
    /lib/ld-linux.so.2 (0x0085d000)  


其中linux-gate.so.1是Linux Virtual Dynamic Shared Object，介绍如下: 
（详见[这里](http://www.ibm.com/developerworks/cn/linux/l-lpic1-v3-102-3/)） 

> 在早期的 x86 处理器中，用户程序与管理服务之间的通信通过软中断实现。 随着处理器速度的提高，这已成为一个严重的瓶颈。 自 Pentium® II 处理器开始，Intel® 引入了 Fast System Call 装置来提高系统调用速度， 即采用 SYSENTER 和 SYSEXIT 指令，而不是中断。 
您所看到的 linux-vdso.so.1 是个虚拟库，或者说是 Virtual Dynamic Shared Object，它只存在于程序的地址空间当中。 在旧版本系统中该库为 linux-gate.so.1。 该虚拟库为用户程序以处理器可支持的最快的方式 （对于特定处理器，采用中断方式；对于大多数最新的处理器，采用快速系统调用方式） 访问系统函数提供了必要的逻辑 。


动态库配置文件/etc/ld.so.conf,可以用`ldconfig`命令来处理/etc/ld.so.conf. 

> "ldconfig 命令在 /etc/ld.so.cache 中为最近使用过的共享库生成必须的链接和 cache , 动态加载器利用来自 ld.so.cache 的缓存文件来定位需要动态加载及链接的文件。 如果改变了 ld.so.conf（或在 ld.so.conf.d 中增加新文件）， 必须运行 ldconfig 命令（以 root 用户身份）来重构 ld.so.cache 文件。 "

`/sbin/ldconfig -p`命令展示/etc/ld.so.cache中的内容 

想要加载制定的库,可以设定环境变量 `LD_LIBRARY_PATH`, 以冒号分隔, 该环境变量主要用于指定查找共享库时除了默认路径之外的其他路径。(该路径在默认路径之前查找), 例子: 
 
    export LD_LIBRARY_PATH=/usr/lib/oldstuff:/opt/IBM/AgentController/lib

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