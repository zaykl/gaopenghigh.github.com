---
title: get_name_script
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

一个小脚本从小说中过滤出经常出现的人名(花名获取利器)！
===================================

小说中，人名后经常跟着一个动词或者介词，根据这一点可以找出常见的人名。下面用一个小shell脚本来玩一下^.^

脚本如下:

    #!/bin/sh
    # name:filter_name.sh
    # JH Gao <gaopenghigh@gmail.com>
    # function:从小说中过滤出经常出现的人名
    # 主要步骤如下：
    # 编码转换
    # 把动词替换为换行，于是每行的前几个字很可能就是人名,再把空行去除
    # 取得每行的前3个字
    # 过滤掉一些一般不是名字但又经常出现的字
    # 过滤掉一个字的行
    # 排序，统计，取前100个

    iconv -f GB18030 -t utf-8 $1 \
    | sed 's/[、，“”听笑说道想答。！：？]/\n/g' | sed 's/[[:space:]]*//g' | sed '/^$/d' \
    | cut -nb 1-9 \
    | grep -v -e '^$' -e [:\<\>父今哈咱\"还似转整间没他她它在地低众到却急这就怎最嗷但那是什么都拿曰吃二其每另否两么不了你啊只着突我吧各此又虽便即第嘿忽的忙] -e '其实' -e 'http' -e '……' -e '原来' -e '自己' -e '心想'  -e '终于' -e '当然' -e '微笑' -e '淡淡' -e '们' -e '然后' -e '所以' -e '可以' \
    | sed '/^.\{1\}$/d' \
    | sort | uniq -c | sort -k 1 -n -r | head -n 100

执行结果如下:

    $ ./filter_name.sh 天龙八部.txt
        596 段誉
        564 慕容复
        532 木婉清
        528 王语嫣
        461 段正淳
        358 鸠摩智
        351 游坦之
        323 南海鳄
        297 阿紫
        293 虚竹
        265 阿朱
        257 保定帝
        249 萧峰
        232 丁春秋
        211 乌老大
        203 马夫人
        174 王夫人
        160 段延庆
        159 段公子
        156 巴天石
        143 朱丹臣
        140 钟万仇
        139 段誉心
        137 乔峰
        136 耶律洪
        123 风波恶
        119 寻思
        113 云中鹤
        108 邓百川
        106 苏星河
        105 钟夫人
        103 王姑娘
        101 少林寺
         91 公冶乾
         90 左子穆
         90 全冠清
         89 段誉见
         89 李秋水
         89 徐长老
         87 童姥
         86 慕容公
         84 白世镜
         84 段誉一
         83 非也
         83 赵钱孙
         83 薛神医
         81 黄眉僧
         81 萧峰心
         78 星宿派
         78 崔百泉
         77 司空玄
         73 青袍客
         73 大哥
         72 很好
         72 大伙儿
         71 秦红棉
         71 姊夫
         71 妙极
         70 乔帮主
         69 镇南王
         68 大声
         67 阮星竹
         67 薛慕华
         67 萧远山
         67 段誉大
         67 星宿老
         66 钟灵
         66 司马林
         62 阿碧
         61 慕容博
         60 虚竹心
         55 段郎
         54 霎时之
         54 心中一
         53 萧峰一
         53 包三先
         53 刀白凤
         52 陈长老
         52 诸保昆
         51 玉虚散
         51 江湖上
         51 姑娘
         50 摘星子
         50 康广陵
         50 姚伯当
         49 飞库论
         49 飞库制
         49 颤声
         49 」阿朱
         49 电脑访
         49 手机访
         48 木姑娘
         47 褚万里
         47 虚竹一
         47 少林派
         46 高升泰
         45 萧峰见
         45 大理段
         44 华赫艮
         43 站起身

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
