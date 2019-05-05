---
layout: post
title:  Linux常考知识点
date: 2019-01-05
categories: linux
tags:  os
author: Jason Chi
---
* content
{:toc}




### Linux的运行级别
0. 系统停机状态，系统默认运行级别不能设置为0，否则不能正常启动，机器关闭。
1. 单用户工作状态，root权限，用于系统维护，禁止远程登陆，就像Windows下的安全模式登录。
2. 多用户状态，没有NFS支持。
3. 完整的多用户模式，有NFS，登陆后进入控制台命令行模式。
4. 系统未使用，保留一般不用，在一些特殊情况下可以用它来做一些事情。例如在笔记本电脑的电池用尽时，可以切换到这个模式来做一些设置。
5. X11控制台，登陆后进入图形GUI模式，XWindow系统。
6. 系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动。运行init6机器就会重启。

### 不重启系统下，转换到level 5 运行：
telinit 5

### 删除一个用户同时删除启用主目录：
userdel -r

### 目录/文件夹的权限
r：浏览目录。即读取 " . " 文件的权限。若权限设置为--x，该用户可以进入目录，但无法浏览目录列表。
w：删除、移动目录内文件。 用户必须同时对一目录具有wx权限，才能对目录下文件删除或移动。
x：是否允许该用户进入目录

### export命令的作用是:
 为其它应用程序设置环境变量

### 终端文件与黑洞文件
* 终端文件： /dev/tty
* 黑洞文件： /dev/null

### 删除文件及文件夹
* rm 删除文件
* rm -r 删除文件夹及下面的文件
* rmdir 只能删除空的文件夹

###  crontab用法
使用crontab你可以在指定的时间执行一个shell脚本或者一系列Linux命令。例如系统管理员安排一个备份任务使其每天都运行

如何往 cron 中添加一个作业?

```
#crontab –e
0 5 * * * /root/bin/backup.sh
```

这将会在每天早上5点运行 /root/bin/backup.sh


以下是 crontab 文件的格式：
{minute} {hour} {day-of-month} {month} {day-of-week} {full-path-to-shell-script}
minute: 区间为 0 – 59
hour: 区间为0 – 23
day-of-month: 区间为0 – 31
month: 区间为1 – 12. 1 是1月. 12是12月.
Day-of-week: 区间为0 – 7. 周日可以是0或7.

Crontab 示例
1. 在 12:01 a.m 运行，即每天凌晨过一分钟。这是一个恰当的进行备份的时间，因为此时系统负载不大。
> 1 0 * * * /root/bin/backup.sh
2. 每个工作日(Mon – Fri) 11:59 p.m 都进行备份作业。
> 59 11 * * 1,2,3,4,5 /root/bin/backup.sh
下面例子与上面的例子效果一样：
> 59 11 * * 1-5 /root/bin/backup.sh
3. 每5分钟运行一次命令
> */5 * * * * /root/bin/check-status.sh
4. 每个月的第一天 1:10 p.m 运行
> 10 13 1 * * /root/bin/full-backup.sh
5. 每个工作日 11 p.m 运行。
> 0 23 * * 1-5 /root/bin/incremental-backup.sh

Crontab 选项:
* crontab –e : 修改 crontab 文件. 如果文件不存在会自动创建。
* crontab –l : 显示 crontab 文件。
* crontab -r : 删除 crontab 文件。
* crontab -ir : 删除 crontab 文件前提醒用户。
