---
title: book_linux_unix_philosophy
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

《Linux/Unix设计思想》读书笔记
============================

《Linux/Unix设计思想》属于那种可以再零碎时间阅读的书，每一章节都介绍一个Unix/Linux的特性，自成体系，同时合起来有大致总结了Linux/Unix的设计上的特点。这些特点，平时使用Linux时肯定会注意到，但未必弄总结出来。

1. “小即是美”。也就是KISS原则。能简单的不要弄复杂和所谓高级，只满足90%的人的需求。
2. 尽快建立原型。要知道你不肯第一遍时就做得很好，而所有漂亮的程序都是修改迭代出来的。
3. 可移植性很重要，使用文本来存储数据。除非绝对必要，不必为性能二减少可移植性或者简单性。用可以直接修改的文本存储数据是最佳方案。
4. 充分使用软件的杠杆效应。能用shell的不用C，能用一些简单工具完成的就避免自己重写。
5. 避免不必要的交互，用户要对自己负责。
6. 让每个程序成为过滤器，用管道连接它们实现各种各样的功能。
7. 这些设计思想，都接受了几十年的考验，证明是很有效的。也时刻提醒着人，软件的最终目的是为人类工作的。

JH, 2013-01-27

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
