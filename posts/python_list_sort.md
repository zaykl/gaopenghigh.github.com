---
title: python_list_sort
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

python列表排序
=============

简单记一下python中List的sort方法（或者sorted内建函数）的用法。

**关键字：**
python列表排序，python字典排序，sorted

## `sorted`方法

List的元素可以是各种东西，字符串，字典，自己定义的类等。

sorted函数用法如下：

    sorted(data, cmp=None, key=None, reverse=False)

其中

- ``data``是待排序数据，可以使List或者iterator, cmp和key都是函数，这两个函数作用与data的元素上产生一个结果，sorted方法根据这个结果来排序。
- ``cmp(e1, e2)`` 是带两个参数的比较函数, 返回值: 负数: e1 < e2, 0: e1 == e2, 正数: e1 > e2. 默认为 None, 即用内建的比较函数.
- ``key`` 是带一个参数的函数, 用来为每个元素提取比较值. 默认为 None, 即直接比较每个元素.
通常, ``key`` 和 ``reverse`` 比 ``cmp`` 快很多, 因为对每个元素它们只处理一次; 而 cmp 会处理多次.

下面通过例子来说明sorted的用法：

## 对由tuple组成的List排序

    >>> students = [('john', 'A', 15), ('jane', 'B', 12), ('dave', 'B', 10),]

- 用key函数排序(lambda的用法见 注释1)

        >>> sorted(students, key=lambda student : student[2])   # sort by age
        [('dave', 'B', 10), ('jane', 'B', 12), ('john', 'A', 15)]

- 用cmp函数排序

        >>> sorted(students, cmp=lambda x,y : cmp(x[2], y[2])) # sort by age
        [('dave', 'B', 10), ('jane', 'B', 12), ('john', 'A', 15)]

- 用 operator 函数来加快速度, 上面排序等价于:(itemgetter的用法见 注释2)

        >>> from operator import itemgetter, attrgetter
        >>> sorted(students, key=itemgetter(2))

- 用 operator 函数进行多级排序

        >>> sorted(students, key=itemgetter(1,2))  # sort by grade then by age
        [('john', 'A', 15), ('dave', 'B', 10), ('jane', 'B', 12)]

## 对由字典排序

    >>> d = {'data1':3, 'data2':1, 'data3':2, 'data4':4}
    >>> sorted(d.iteritems(), key=itemgetter(1), reverse=True)
    [('data4', 4), ('data1', 3), ('data3', 2), ('data2', 1)]

----

**注释1**
参考：http://jasonwu.me/2011/10/29/introduce-to-python-lambda.html

**注释2**
参考:http://ar.newsmth.net/thread-90745710c90cf1.html

    class itemgetter(__builtin__.object)
    |  itemgetter(item, ...) --> itemgetter object
    |
    |  Return a callable object that fetches the given item(s) from its operand.
    |  After, f=itemgetter(2), the call f(r) returns r[2].
    |  After, g=itemgetter(2,5,3), the call g(r) returns (r[2], r[5], r[3])


相当于

    def itemgetter(i,*a):
        def func(obj):
            r = obj[i]
            if a:
                r = (r,) + tuple(obj[i] for i in a)
            return r
        return func

    >>> a = [1,2,3]
    >>> b=operator.itemgetter(1)
    >>> b(a)
    2
    >>> b=operator.itemgetter(1,0)
    >>> b(a)
    (2, 1)
    >>> b=itemgetter(1)
    >>> b(a)
    2
    >>> b=itemgetter(1,0)
    >>> b(a)
    (2, 1)

----

参考资料：

+ http://www.linuxso.com/linuxbiancheng/13340.html
+ http://www.douban.com/note/13460891/

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
