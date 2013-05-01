---
title: 5_year_linux_tips_5_bash
---

<head><link rel='stylesheet' href='/style/github2.css'/></head>

五年Linux使用技巧(5)--bash
=========================

作者：[gaopenghigh](http://gaopenghigh.github.com)
，转载请注明出处。
[（原文地址）](http://gaopenghigh.github.io/posts/5_year_linux_tips_5_bash.html)

本文主要介绍了Linux中bash使用的一些技巧。
从最开始接触Linux到现在已经有5年了，和所有人一样，少不了折腾。折腾后偶尔我会把方法记录下来，现在简单总结一下。
所以的命令功能通过man都能找到具体用法，我只把自己觉得常用的列举出来。


### 第一个参数作为函数名调用函数

    func_eval() {
        TYPE=`type $1 | head -1 | awk '{print $NF}'`
        if [ $? -gt 0 ]; then
            echo "@ops:ERROR call function: $1 ... failed"
        elif [ "$TYPE" == "function" ]; then
            eval $*
        else
            echo "@ops:ERROR invalid function: $1 ..."
            exit 1
        fi 
    }

### 逐行处理文件的最易懂且高效的方法：

    while read $LINE
    do
         echo "$LINE" >> $OUTFILE
         :
    done

### 得到一个变量代表的字符串的长度:

    echo ${#VAR}

### `${PAGER:-more}

shell命令`${PAGER:-more}`
的意思是：如果shell变量PAGER已经定义，且其值非空，则使用其值，否则使用字符串more。


JH, 2013-03-10

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

