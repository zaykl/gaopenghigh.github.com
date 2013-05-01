---
title: inotifyrsync
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

inotifyrsync -- 用inotify和rsync实现多机实时同步
=============================================

对于热备的机器，需要实时把文件目录的修改从master同步到slave上。用crontab加rsync的方式能够实现几分种延迟内的同步，但用对于文件数目很大的情况，每次rsync都会有很多计算，比较耗费CPU资源。下面的脚本用inotify和rsync实现实时同步。为了保险起见，最好还是设置一个crontab来定时跑整个目录的rsync，不过定时间隔可以稍微长一点，以免占用太多系统资源。

[github上的链接](https://github.com/gaopenghigh/inotifyrsync)

直接上代码：

**README**

    Use rsync and inotify to sync files and directories from one host to other hosts
    
    Include 2 files:
    sync.sh
    do_sync.py
    
    RUN:
    0. Make sure source host can ssh into destination hosts without password
    1. Do some setting in file do_sync.py
    2. run sync.sh

**sync.sh**

    #!/bin/bash

    LOG=`cat $(dirname $0)/do_sync.py | grep "LOG_FILE = " | head -n 1 | awk '{print $3}' | tr -d \'`
    BASE_DIR=`cat $(dirname $0)/do_sync.py | grep "BASE_DIR = " | head -n 1 | awk '{print $3}' | tr -d \'`
    tester=`/bin/ps aux | grep -v grep | grep do_sync.py`
    if [[ ! -z $tester ]];then
        echo "sync.sh already running"
    else
        echo "/usr/bin/inotifywait -mrq --format '%w%f' -e modify,delete,create,move $BASE_DIR | $(dirname $0)/do_sync.py > $LOG &"
        /usr/bin/inotifywait -mrq --format '%w%f' -e modify,delete,create,move $BASE_DIR | $(dirname $0)/do_sync.py > $LOG &
    fi


**do_sync.py**

    #!/usr/bin/python
    # -*- coding: utf-8 -*-
    # Author: gaopenghigh@gmail.com
    
    import sys
    import os
    import commands
    import threading
    import time
    
    ### CONFIGS, YOU CAN CHANGE SETTINGS HERE ###
    THREAD_NUM = 2                # how many threads do the rsync job
    SLEEP_TIME = 3                # if nothing to sync, just sleep, second
    MERGE_NUM = 25                # try to sync dirs instead of files in the same dir
    BASE_DIR = '/home/admin/run'  # root dir you want to sync, anything out of this dir will not sync
    LOG_FILE = '/home/admin/out/sync.log'  # log file, set to '/dev/null' if no log file needed
    DESTS = [                     # Destination Hosts
            'hz-wsbuyblog-web1',
            'hz-wsbuyblog-web2',
            'hz-ws-buyblog-web3'
            ]
    DESTS = ['hz-wsbuyblog-web1',]
    ### END OF CONFIGS ###
    
    
    class WorkThread(threading.Thread):
        ''' working thread in thread pool'''
        def __init__(self, work_set, **kwargs):
            threading.Thread.__init__(self, kwargs=kwargs)
            self.work_set = work_set
            self.setDaemon(True)
            self.start()
    
        def run(self):
            '''always trying to get rsync job, if nothing to sync, sleep SLEEP_TIME seconds'''
            while True:
                try:
                    # if too many files need to sync, try rsync dir of these files
                    set_size = len(self.work_set)
                    while set_size > MERGE_NUM:
                        last_size = set_size
                        merge(self.work_set)
                        set_size = len(self.work_set)
                        # Can not merge any more
                        if last_size == set_size:
                            break
                    f = self.work_set.pop()
                    if not f.startswith(BASE_DIR):
                        continue
                    if os.path.isdir(f):
                        dest_file = os.path.dirname(f)
                    else:
                        dest_file = f
                    for d in DESTS:
                        cmd = '/usr/bin/rsync -az --delete %s %s:%s' % (f, d, dest_file)
                        res = commands.getstatusoutput(cmd)
                        if res[0] != 0:
                            flag = 'FAIL'
                        else:
                            flag = 'OK'
                        print '[%s] [WAITING:%s] [%s] %s ---> %s:%s' % (time.strftime('%Y-%m-%d_%H:%M:%S'), len(self.work_set), flag, f, d, dest_file)
                except KeyError:
                    time.sleep(SLEEP_TIME)
                except:
                    print sys.exc_info()
                    raise
    
    
    class ThreadPool:
        # thread pool
        def __init__(self, work_set, thread_num = 10):
            self.work_set = work_set
            self.threads = []
            self.thread_num = thread_num
            self._create_thread_pool()
    
        def _create_thread_pool(self):
            self.threads = []
            for i in range(self.thread_num):
                t = WorkThread(self.work_set)
                self.threads.append(t)
    
    
    def merge(file_set):
        '''
        if more thanfiles in the same dir, use dir instead of these files to make file_set smaller
        '''
        print '### MERGE %s' % len(file_set),
        tmp_set = file_set.copy()
        dir_dic = {}
        for f in tmp_set:
            dir = '/'.join(f.split('/')[:-1])
            if dir in dir_dic:
                dir_dic[dir] += [f,]
            else:
                dir_dic[dir] = [f,]
        for d, fs in dir_dic.items():
            if len(fs) > 1:
                for f in fs:
                    file_set.remove(f)
                file_set.add(d)
        print '---> %s' % len(file_set)
    
    
    def review(file_set):
        '''delete unnecessary files'''
        all_files = file_set.copy()
        for f in all_files:
            # added dir
            if os.path.isdir(f):
                # do not need rsync files in the dir
                tmp_set = file_set.copy()
                for xf in tmp_set:
                    if xf.startswith(f) and xf != f:
                        file_set.remove(xf)
        all_files = file_set.copy()
        for f in all_files:
            # deleted file or dir
            if not os.path.isfile(f) and not os.path.isdir(f):
                file_set.remove(f)
                father = '/'.join(f.split('/')[:-1])
                while not os.path.isdir(father):
                    father = '/'.join(father.split('/')[:-1])
                file_set.add(father)
                tmp_set = file_set.copy()
                for xf in tmp_set:
                    if xf.startswith(father) and xf != father:
                        file_set.remove(xf)
    
    
    def main():
        file_set = set([])
        tp = ThreadPool(file_set, THREAD_NUM)
        if not tp:
            print "create thread pool failed"
            return False
        while True:
            line = sys.stdin.readline()
            ignore = False
            if not line:
                break
            line = line.strip()
            file_set_tmp = file_set.copy()
            for exist_file in file_set_tmp:
                if line.startswith(exist_file) and line != exist_file:
                    ignore = True
                    break
            if ignore:
                continue
            file_set.add(line)
            review(file_set)
    
    
    if __name__ == '__main__':
        main()

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
