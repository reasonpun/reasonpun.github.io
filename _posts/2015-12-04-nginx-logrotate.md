---
layout: post
title: Nginx Logrotate
---

<!--  -->
<!--  -->
<!--  _ __    __     __      ____    ___     ___   _____   __  __    ___     -->
<!-- /\`'__\/'__`\ /'__`\   /',__\  / __`\ /' _ `\/\ '__`\/\ \/\ \ /' _ `\   -->
<!-- \ \ \//\  __//\ \L\.\_/\__, `\/\ \L\ \/\ \/\ \ \ \L\ \ \ \_\ \/\ \/\ \  -->
<!--  \ \_\\ \____\ \__/.\_\/\____/\ \____/\ \_\ \_\ \ ,__/\ \____/\ \_\ \_\ -->
<!--   \/_/ \/____/\/__/\/_/\/___/  \/___/  \/_/\/_/\ \ \/  \/___/  \/_/\/_/ -->
<!--                                                 \ \_\                   -->
<!--                                                  \/_/                   -->
<!--  -->


### Centos Logrotate ###

####  关于 ####

_logrotate_ is  designed to ease administration of systems that generatelarge numbers of log files.  It allows automatic rotation, compression,removal, and mailing of log files.  Each log file may be handled daily,weekly, monthly, or when it grows too large. ([Logrotate man page](http://linuxcommand.org/man_pages/logrotate8.html))

#### 配置文件 ####

##### 内容示例

```
[reason@online_http:/etc/logrotate.d]$ cat nginx.logrotate.d
#
# author: reason@mofunsky.com
# date:   2015-03-31 10:15
# cron:
#       59 23 * * * /usr/sbin/logrotate /etc/logrotate.conf > /dev/null
#
# location:
#       /etc/logrotate.d/nginx.logrotate.d
#
/local/nginx*.log {
    daily
    rotate 10
    missingok
    compress
    notifempty
    sharedscripts
    postrotate
        /bin/kill -USR1 $(cat /var/nginx.pid 2>/dev/null) 2>/dev/null || :
    endscript
}
```

##### 存放位置

```
[reason@online_http:/etc/logrotate.d]$ pwd
/etc/logrotate.d
[reason@online_http:/etc/logrotate.d]$
```

##### 执行方式和时间
```
# add by reason @ 2015-04-05 20:41
# logrotate nginx log daily
59      23      *       *       *       root /usr/sbin/logrotate /etc/logrotate.conf >/dev/null 2>&1
```

##### 执行效果
```
# 会按照日期，在每天的23：59分将改天日志转储压缩为*.gz文件
[reason@online_http:/local/nginx_access_log]$ ls -lsh
total 5.1G
427M -rwxrwxrwx 1 nginx nginx 427M Oct 11 12:01 nginx_me_access.log
259M -rwxrwxrwx 1 nginx nginx 259M Oct  1 23:59 nginx_me_access.log-20151001.gz
233M -rwxrwxrwx 1 nginx nginx 233M Oct  2 23:59 nginx_me_access.log-20151002.gz
228M -rwxrwxrwx 1 nginx nginx 228M Oct  3 23:59 nginx_me_access.log-20151003.gz
225M -rwxrwxrwx 1 nginx nginx 225M Oct  4 23:59 nginx_me_access.log-20151004.gz
232M -rwxrwxrwx 1 nginx nginx 232M Oct  5 23:59 nginx_me_access.log-20151005.gz
```

##### 另外一种执行方式
logrotate 是linux系统的缺省安装命令，初始化状态下即存在缺省的配置信息，其中包括执行文件和时间

```
[reason@online_http:/local/nginx_access_log]$ cd /etc/logrotate.d/
[reason@online_http:/etc/logrotate.d]$ ls
nginx.logrotate.d
[reason@online_http:/etc/logrotate.d]$ pwd
/etc/logrotate.d
[reason@online_http:/etc/logrotate.d]$
```

比如在/etc/logrotate.d目录下会缺省放置一批需要转储的日志服务对应的配置文件，比如httpd，exim等，logrotate会按照配置文件信息按照既定时间对其产生的日志数据进行转储操作。

```
[reason@online_http:/etc/logrotate.d]$ pwd
/etc/logrotate.d
[reason@online_http:/etc/logrotate.d]$ cat yum
/var/log/yum.log {
    missingok
    notifempty
    size 30k
    yearly
    create 0600 root root
}
[reason@online_http:/etc/logrotate.d]$
```
具体参数涵义可参考[Logrotate man page](http://linuxcommand.org/man_pages/logrotate8.html)

缺省状态下，logrotate的自动执行分别被放入了如下几个文件中

  *  cron.daily
  *  cron.weekly
  *  cron.monthly

而对于不同的linux发行版本以上脚本可能被放置在/etc/crontab中
或者

```
[root@i-bntub2bp logrotate.d]# cat /etc/anacrontab
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1       5       cron.daily              nice run-parts /etc/cron.daily
7       25      cron.weekly             nice run-parts /etc/cron.weekly
@monthly 45     cron.monthly            nice run-parts /etc/cron.monthly
[root@i-bntub2bp logrotate.d]#
```
针对

```
[root@i-bntub2bp logrotate.d]# cat /etc/redhat-release
CentOS release 6.6 (Final)
[root@i-bntub2bp logrotate.d]#
```
而言，可以直接在/etc/[anacrontab](https://www.centos.org/docs/2/rhl-cg-en-7.2/anacron.html)中找到如上信息。

大体解释下anacrontab的部分参数涵义

```
START_HOURS_RANGE=3-22  # 执行时间为3点到22点之间
RANDOM_DELAY=45 # 时间处于可执行区间内随机延迟45分钟之内任意时间
1       5       cron.daily              nice run-parts /etc/cron.daily
# 执行时间为3点到22点之间执行/etc/cron.daily脚本 (after reboot and after the machine has been up for 5 minutes^^), 如果没有重启服务，则会在3：05之后执行。
```

#### 参考文献
  * [http://huoding.com/2013/04/21/246](http://huoding.com/2013/04/21/246)
  * [http://serverfault.com/questions/135906/when-does-cron-daily-run](http://serverfault.com/questions/135906/when-does-cron-daily-run)
  * [http://www.cyberciti.biz/faq/linux-when-does-cron-daily-weekly-monthly-run/](http://www.cyberciti.biz/faq/linux-when-does-cron-daily-weekly-monthly-run/)
  * [http://linuxcommand.org/man_pages/logrotate8.html](http://linuxcommand.org/man_pages/logrotate8.html)
  * [https://www.centos.org/docs/2/rhl-cg-en-7.2/anacron.html](https://www.centos.org/docs/2/rhl-cg-en-7.2/anacron.html)
