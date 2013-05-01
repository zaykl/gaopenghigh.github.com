---
title: 5_year_linux_tips_4_software
---

<head><link rel='stylesheet' href='/style/github2.css'/></head>

五年Linux使用技巧(4)--软件
=========================

作者：[gaopenghigh](http://gaopenghigh.github.com)
，转载请注明出处。
[（原文地址）](http://gaopenghigh.github.io/posts/5_year_linux_tips_4_software.html)

本文主要介绍了Linux软件方面的一些技巧。
从最开始接触Linux到现在已经有5年了，和所有人一样，少不了折腾。折腾后偶尔我会把方法记录下来，现在简单总结一下。
所以的命令功能通过man都能找到具体用法，我只把自己觉得常用的列举出来。

### Nautilus的技巧

* 打开一个位置：`Ctrl + L`
* 打开父目录：`Ctrl + Up`

### evince

ubuntu的默认PDF阅读器evince中，`j`和`k`可以上下滚动

### 把图片缩小为原来的20%大小：

    convert -resize 20%x20% IMGNAME NEWIMGNAME

### mplayer字符播放：

    mplayer -vo aa xxx.avi 用无颜色的字符播放；
    mplayer -vo caca xxx.avi 用有颜色的ASCII字符播放；
    mplayer -vo matrixiew xxx.avi 用类似黑客帝国里面的终端播放！

### 命令行下的截屏可以用软件fbgrab，延迟10s截屏

    fbgrab -s 10 screen.png

### virtualbox中克隆vdi文件

    VBoxManage clonevdi source.vdi target.vdi

### 好用的快捷操作软件：`synapse`

### 自定义的终端自动补全

比如我要对ssh, ping, myscript这三个命令自动补全参数，其中参数名都写在了/tmp/my_word_list文件中，我们可以在 `.bashrc`中做如下设置：

    function _my_cmpl() {
        local my_cmpl_words cur
        COMPREPLY=()
        cur="${COMP_WORDS[COMP_CWORD]}"
        my_cmpl_words=`cat /tmp/my_word_list`
        COMPREPLY=( $( compgen -W "$my_cmpl_words" -- "$cur" ) )
    }
    complete -F _my_cmpl ssh ping myscript

### ssh保存会话

vi /home/用户名/.ssh/config (没有就新建一个),加入以下内容:

    jHost *
    jControlMaster auto
    jControlPath /tmp/%r@%h:%p

保存退出. 只要登录一次服务器，再在新的终端中登录同一个服务器时，就不用再输密码了。


### ssh翻墙

ssh翻the墙，如果你在墙外有台服务器，并且可以不用密码ssh到上面，则可以使用ssh做端口转发，实现翻the墙。加上Chrome上的switchy或者Firefox上的autoProxy插件就可以自由上网了。

把本地的7001端口作为转发端口：

    ssh -qTfnN -D 7001 root@YOUR_SERVER

JH, 2013-03-09

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

