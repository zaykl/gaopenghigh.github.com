---
title: mysql_duble_replicate
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

MySQL双向同步热备设置以及同步错误的处理
==================================

# 环境 
 
A: 192.168.0.1

B: 192.168.0.2

其中A上已经有数据库在服务，需要在B上搭建一个备库，并且和A实现双向同步。

# 设置

## 授权复制用户 
即分别在A，B上增加一个用户让彼此访问 
 
A:  

    grant replication slave,file on *.* to 'backup'@'192.168.0.2' identified by '123456';
  
B:

    grant replication slave,file on *.* to 'backup'@'192.168.0.1' identified by '123456';  

检测通过backup这个用户能够访问彼此的MySQL。

## 配置文件设置
B:

    [mysqld]  
    # 省略...  
    # replication settings  
    # A 192.168.0.1  
    # B 192.168.0.2  
    server-id=2  
    log-bin=/home/mysql/log/mysql-bin  
    # 因为是双向，自动增加的id会有冲突，把步长改为2 初始设为2  
    auto_increment_increment=2  
    auto_increment_offset=2  
    # master设置  
    master-host=192.168.0.1  
    master-user=backup  
    master-pass=123456  
    master-port=3306  
    master-connect-retry=30  
    # 设置需要复制的库  
    replicate-do-db=your_db1  
    replicate-do-db=your_db2
    # 省略...  

A: 

    [mysqld]  
    # 省略...  
    # replication settings  
    # A 192.168.0.1  
    # B 192.168.0.2  
    server-id=1  
    log-bin=/home/mysql/log/mysql-bin  
    # 因为是双向，自动增加的id会有冲突，把步长改为2 初始设为1  
    auto_increment_increment=2  
    auto_increment_offset=1  
    # master设置  
    master-host=192.168.0.2  
    master-user=backup  
    master-pass=123456  
    master-port=3306  
    master-connect-retry=30  
    # 设置需要复制的库  
    replicate-do-db=your_db1  
    replicate-do-db=your_db2  
    # 省略...  


## 同步现有数据 
停止A数据库的操作，把A中的数据用mysqldump或者直接拷贝文件的方法复制到B，确保B正常。 

## 设置并检查同步 
启动A和B上的的MySQL

检查master status:

A: 

    mysql> show master status;  
    +------------------+----------+--------------+------------------+  
    | File | Position | Binlog_Do_DB | Binlog_Ignore_DB |  
    +------------------+----------+--------------+------------------+  
    | mysql-bin.000004 |  8937501 |  |  |  
    +------------------+----------+--------------+------------------+  
    1 row in set (0.00 sec) 

B: 

    mysql> show master status;  
    +------------------+----------+--------------+------------------+  
    | File | Position | Binlog_Do_DB | Binlog_Ignore_DB |  
    +------------------+----------+--------------+------------------+  
    | mysql-bin.000019 |  597 |  |  |  
    +------------------+----------+--------------+------------------+  
    1 row in set (0.00 sec)  


检查slave status:

A: 

    mysql> show slave status\G  
    *************************** 1. row ***************************  
     Slave_IO_State: Waiting for master to send event  
    Master_Host: 192.168.0.2  
    Master_User: backup  
    Master_Port: 3306  
      Connect_Retry: 60  
    Master_Log_File: mysql-bin.000019  
    Read_Master_Log_Pos: 597  
     Relay_Log_File: mysqld-relay-bin.000005  
      Relay_Log_Pos: 429  
      Relay_Master_Log_File: mysql-bin.000019  
       Slave_IO_Running: Yes  
      Slave_SQL_Running: Yes  
    Replicate_Do_DB: your_db1,your_db2  
    Replicate_Ignore_DB:  
     Replicate_Do_Table:  
     Replicate_Ignore_Table:  
    Replicate_Wild_Do_Table:  
    Replicate_Wild_Ignore_Table:  
     Last_Errno: 0  
     Last_Error:  
       Skip_Counter: 0  
    Exec_Master_Log_Pos: 597  
    Relay_Log_Space: 429  
    Until_Condition: None  
     Until_Log_File:  
      Until_Log_Pos: 0  
     Master_SSL_Allowed: No  
     Master_SSL_CA_File:  
     Master_SSL_CA_Path:  
    Master_SSL_Cert:  
      Master_SSL_Cipher:  
     Master_SSL_Key:  
      Seconds_Behind_Master: 0  
    1 row in set (0.00 sec)  

B: 

    mysql> show slave status\G  
    *************************** 1. row ***************************  
     Slave_IO_State: Waiting for master to send event  
    Master_Host: 192.168.0.1  
    Master_User: backup  
    Master_Port: 3306  
      Connect_Retry: 30  
    Master_Log_File: mysql-bin.000004  
    Read_Master_Log_Pos: 9914005  
     Relay_Log_File: mysqld-relay-bin.000008  
      Relay_Log_Pos: 9914142  
      Relay_Master_Log_File: mysql-bin.000004  
       Slave_IO_Running: Yes  
      Slave_SQL_Running: Yes  
    Replicate_Do_DB: your_db1,your_db2  
    Replicate_Ignore_DB:  
     Replicate_Do_Table:  
     Replicate_Ignore_Table:  
    Replicate_Wild_Do_Table:  
    Replicate_Wild_Ignore_Table:  
     Last_Errno: 0  
     Last_Error:  
       Skip_Counter: 0  
    Exec_Master_Log_Pos: 9914005  
    Relay_Log_Space: 9914142  
    Until_Condition: None  
     Until_Log_File:  
      Until_Log_Pos: 0  
     Master_SSL_Allowed: No  
     Master_SSL_CA_File:  
     Master_SSL_CA_Path:  
    Master_SSL_Cert:  
      Master_SSL_Cipher:  
     Master_SSL_Key:  
      Seconds_Behind_Master: 0  
    1 row in set (0.00 sec)  

需要特别注意的是这两行： 

    Slave_IO_Running: Yes  
    lave_SQL_Running: Yes  

必须都是Yes，否则同步不成功。 

分别在A和B上做操作，看同步是否生效； 

# 复制不成功的解决方法 

一开始我设置从A复制到B时出现了错误 

    Slave_IO_Running: No  
    lave_SQL_Running: Yes  

mysqld.log里面： 

    121217 17:43:39 [ERROR] Error reading packet from server: error reading log entry ( server_errno=1236)  
    121217 17:43:39 [ERROR] Got fatal error 1236: 'error reading log entry' from master when reading data from binary log  

这时我用CHANGE MASTER命令重新指定master： 

停止A的MySQL操作，查看master状态:

A: 

    mysql> show master status;  
    +------------------+----------+--------------+------------------+  
    | File | Position | Binlog_Do_DB | Binlog_Ignore_DB |  
    +------------------+----------+--------------+------------------+  
    | mysql-bin.000003 |98|  |  |  
    +------------------+----------+--------------+------------------+  
    1 row in set (0.00 sec)  

记住上面的File和Position数值 

在B上用CHANGE MASTER命令重新指定master:

    mysql> slave stop;  
    mysql> change master to master_host='192.168.0.1',master_user='backup',master_password='123456', master_log_file='mysql-bin.000003',master_log_pos=98;  
    mysql> slave start;  

此时检查B上的slave status, 发现已经OK： 

    Slave_IO_Running: Yes  
    lave_SQL_Running: Yes  


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
