---
title: 5_year_linux_tips_2_system_1
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

五年Linux使用技巧(2)--系统（上）
===============================

作者：[gaopenghigh](http://gaopenghigh.github.com)
，转载请注明出处。
[（原文地址）](http://gaopenghigh.github.io/posts/5_year_linux_tips_2_system_1.html)

本文主要介绍了Linux系统方面的一些技巧。
从最开始接触Linux到现在已经有5年了，和所有人一样，少不了折腾。折腾后偶尔我会把方法记录下来，现在简单总结一下。
所以的命令功能通过man都能找到具体用法，我只把自己觉得常用的列举出来。

### /etc/fstab文件出错

此时，系统不能正常启动，此时可以启动进入single user模式，而改模式下根目录"/"是只读的，可以用如下的命令把"/"重新挂载为“读写”：

    [root@linux]# mount -n -o remount,rw /
    -n : mount but do not change /etc/mtab
    -o : options

### partprobe--不用重启使用新的分区表

`partprobe` : reinitializes the kernel in memory of the partition table.更改分区设置后，系统提示需要重启以更改kernel中的分区表，利用partprobe即可免除重启。


### ubuntu系统在GDM和KDM之间切换

如果你同时安装了GNOME和KDE，有时候需要在gdm和kdm之间切换：

    sudo dpkg-reconfigure gdm

### 增加swap空间

* 创建一个新的分区或新的文件
* 用mkswap工具写入特殊标记
* 在/etc/fstab中加入新的记录
* 激活swap分区，命令swapon -a 或者 swapon [SWAPFILE]
* 用swapon -s 命令检查swap分区的状态

例子：

    dd if=/dev/zero of=/swapfile bs=1M count=100
    mkswap /swapfile
    vi /etc/fstab ...
    swapon -a

### 不用格式化把ext2转换为ext3

    tune2fs -j /dev/sdax

### 一个网卡绑定多个IP

例子：
系统是CentOS, 网卡是eth0，如果只要额外设置1个IP，则创建文件
`/etc/sysconfig/network-scripts/ifcfg-eth0:0`
，在该文件中设置IP信息。
如果需要设置一个IP段，则创建文件`ifcfg-ethX-rangeX`：

ifcfg-eth0-range0:

    DEVICE=eth0-range0
    BOOTPROTO=static
    HWADDR=08:00:27:24:C2:72
    ONBOOT=yes
    IPADDR_START=192.168.56.20
    IPADDR_END=192.168.56.30
    NETMASK=255.255.255.0

然后重启网络

### 更改hostname搜寻顺序

修改 /etc/nsswitch.conf
（修改这个文件可以实现更多功能，比如查询group, passwd, networks等的查询顺序，具体可以查看man手册）

### lsof

lsof命令可以列出所有打开的文件。这个命令最常用的功能是找到“丢失”的空间。

比如我们用df命令看到/home分区只剩下1G了，但用du命令得到的结果是应该还有5G才对，这种情况往往是由于一些文件被删除，但这些被删除的文件的文件句柄还没有被释放导致的。用命令

    lsof | grep -i deleted

查看有哪些文件被删除了但文件句柄还没释放，kill或者重启响应的进程就能找回“丢失”的空间。

lsof有时候还能恢复被误删除的文件，具体方法请google.

###  目录长度

目录的长度从来不会是0，因为它总是包含.和..两项。符号连接的长度指其路径名包含的字符数，由于路径名中至少有一个字符，所以长度也不为0.

### 创建一个名为“-f”的文件夹

要创建一个名为“-f”的文件夹，使用命令`mkdir -f`必然失败，而用`mkdir -- -f`则可以创建成功.

### 用"cd -"在最近使用的两个目录间切换

### su 和 su - 的区别

执行su时新shell将继承当前的shell环境，su -模拟实际的root登陆会话

### 快速清除history

    export HISTSIZE=0

JH, 2013-03-07

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

