---
title: 5_year_linux_tips_3_system_2
---
<head>
<link rel='stylesheet' href='/style/github2.css'/>
</head>

五年Linux使用技巧(3)--系统（下）
===============================

本文主要介绍了Linux系统方面的一些技巧。
从最开始接触Linux到现在已经有5年了，和所有人一样，少不了折腾。折腾后偶尔我会把方法记录下来，现在简单总结一下。
所以的命令功能通过man都能找到具体用法，我只把自己觉得常用的列举出来。

### bash中的`$`相关参数

* `$0` - 表示当前文件名
* `$*` - 以空格分离所有参数，形成一个字符串
* `$@` - 以空格分离所有参数，形成一个字符串组合。与`$*`的不同表现在被""引用时，`"$*"`是一个字符串，而`"$@"`则包含多个字符串
* `$#` - 传递给进程的参数数目
* `$?` - 上一条命令的执行结果，没有错误时为0
* `$$` - 本条命令的PID

### bash技巧，由变量的内容来组合为另一个变量的变量名

EXAMPLE：

    A_B_C_D="something"
    t1="B"
    t2="_D"
    eval echo \$A_${t1}_C${t2};

### ThinkPad X220指点杆设置

Ubuntu(12.04, 12.10)下，ThinkPad X220指点杆设置，分别设置灵敏度和速度

    echo -n 225 > /sys/devices/platform/i8042/serio1/serio2/sensitivity
    echo -n 115 > /sys/devices/platform/i8042/serio1/serio2/speed

### 备份主引导扇区（bootsector）

如果启动文件随坏，可以通过恢复主引导扇区来视图修复：

备份

    dd if=/dev/hda of=bootsector.img bs=512 count=1

恢复

    dd if=bootsector.img of=/dev/hda

上面两步只是恢复了主引导扇区，很可能还需要把/boot里面的内容全都恢复才能正常启动，所以也可以备份一下/boot下面的文件。

### bash命令行输入技巧

使用`Ctrl+R`来搜索以前用过的命令
使用`Ctrl+W`删除当前单次
使用`Ctrl+U`删除当前行

### xargs

xargs很强大，用 `-l{}` 可以指定参数的位置：

    cat hosts | xargs -I{} ssh root@{} hostname

### 写安全的bash脚本

* `set -e`，当有错误发生时，脚本会退出
* `set -u`，当bash发现有没有初始化的变量时就退出

更多可参考：[《写出健壮的Bash脚本》](http://article.yeeyan.org/view/58906/257928)

### tar打包指定列表中列出的文件

    cat yourlist.lst
    /etc/fstab
    /home/admin/bin/somefile.sh
    /home/mysql/somefile
    ...
    
    tar cvzf xxx.tar.gz -T yourlist.lst

### 指定一个DNS服务器查询域名记录

    dig @8.8.8.8 www.google.com

### sort命令最需要注意的参数是-k和-s：

    -s, --stable
                  stabilize sort by disabling last-resort comparison

stable表示最终的顺序依赖于原来的顺序。

    $ cat a.txt 
    a
    A
    B
    b
    $ sort -f a.txt
    a
    A
    b
    B
    $ sort -f -s a.txt
    a
    A
    B
    b

例子中，-f表示不区分大小写，-s表示顺序依赖于原来文件的顺序

    -k, --key=POS1[,POS2]
                  start a key at POS1 (origin 1), end it at POS2 (default end of line).

所以只以第二列来排序应该写：

    sort -k1,1

更多关于sort的技巧，可以参考
[《Sort Files Like A Master With The Linux Sort Command (Bash)》](http://www.skorks.com/2010/05/sort-files-like-a-master-with-the-linux-sort-command-bash/)


----

参考资料：

* http://www.quora.com/Linux/What-are-some-time-saving-tips-that-every-Linux-user-should-know
* http://article.yeeyan.org/view/58906/257928
* http://www.skorks.com/2010/05/sort-files-like-a-master-with-the-linux-sort-command-bash/


JH, 2013-03-08

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

