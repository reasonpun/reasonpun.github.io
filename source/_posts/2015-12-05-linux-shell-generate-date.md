---
layout: post
title: Linux shell 获得以前日期
date: 2015-12-05 18:02:16
excerpt: Linux shell 获取各种格式的日期
---


###  在linux shell里，我想获得以前的日期

1、比如，去年的上个月的昨天的日期：（今天是2009年2月2日，也就是2008年1月1日）

```
reasonpun@reasonpun:~$ logRecordDate="`date -d "-1 year -1 month -1 day" "+%Y_%m_%d"`"
reasonpun@reasonpun:~$ echo $logRecordDate
2008_01_01
reasonpun@reasonpun:~$
```

<!--more-->

2、上个月的今天：

```
reasonpun@reasonpun:~$ logRecordDate="`date -d "-1 month" "+%Y_%m_%d"`"
reasonpun@reasonpun:~$ echo $logRecordDate
2009_01_02
```

3、去年的今天：

```
reasonpun@reasonpun:~$ logRecordDate="`date -d "-1 year" "+%Y_%m_%d"`"
reasonpun@reasonpun:~$ echo $logRecordDate
2008_02_02
```

4、上个月的昨天：

```
reasonpun@reasonpun:~$ logRecordDate="`date -d "-1 month -1 day" "+%Y_%m_%d"`"
reasonpun@reasonpun:~$ echo $logRecordDate
2009_01_01
```

5、根据时间戳转换成日期

```
[mofun_mining@i-qe32ajmq ~]$ date -d @1434847028 "+%Y-%m-%d"
2015-06-21
```

其他的类推～～呵呵，还是希望大家给测测其他日期会不会出错呵呵。多谢～～～
鼓捣之环境：ubuntu8.04

update @ 2016-12-19 15:48

### PHP获取后一天日期的写法

```

$date = "04-15-2013";
$date1 = str_replace('-', '/', $date);
$tomorrow = date('m-d-Y', strtotime($date1 . "+1 days"));

echo $tomorrow;

```

### 另附上windows下获得前一天的日期：

```
@echo off
set td=%date:~2,2%%date:~5,2%%date:~8,2%
set dy=%date:~0,4%
set dm=%date:~5,2%
set dd=%date:~8,2%
set da=%date:~8,2%
if %dm%%dd%==0101 goto L01
if %dm%%dd%==0201 goto L02
if %dm%%dd%==0301 goto L07
if %dm%%dd%==0401 goto L02
if %dm%%dd%==0501 goto L04
if %dm%%dd%==0601 goto L02
if %dm%%dd%==0701 goto L04
if %dm%%dd%==0801 goto L02
if %dm%%dd%==0901 goto L02
if %dm%%dd%==1001 goto L05
if %dm%%dd%==1101 goto L03
if %dm%%dd%==1201 goto L06
if %dd%==02 goto L10
if %dd%==03 goto L10
if %dd%==04 goto L10
if %dd%==05 goto L10
if %dd%==06 goto L10
if %dd%==07 goto L10
if %dd%==08 goto L10
if %dd%==09 goto L10
if %dd%==10 goto L11
set /A dd=dd-1
set dt=%dy%-%dm%-%dd%
goto END
:L10
set /A dd=%dd:~1,1%-1
set dt=%dy%-%dm%-0%dd%
goto END
:L11
set dt=%dy%-%dm%-09
goto END
:L02
set /A dm=%dm:~1,1%-1
set dt=%dy%-0%dm%-31
goto END
:L04
set /A dm=dm-1
set dt=%dy%-0%dm%-30
goto END
:L05
set dt=%dy%-09-30
goto END
:L03
set dt=%dy%-10-31
goto END
:L06
set dt=%dy%-11-30
goto END
:L01
set /A dy=dy-1
set dt=%dy%-12-31
goto END
:L07
set /A "dd=dy%%4"
if not %dd%==0 goto L08
set /A "dd=dy%%100"
if not %dd%==0 goto L09
set /A "dd=dy%%400"
if %dd%==0 goto L09
:L08
set dt=%dy%-02-28
goto END
:L09
set dt=%dy%-02-29
goto END
:END
set dateTime=20%dt:~2,2%%dt:~5,2%%dt:~8,2%
```
