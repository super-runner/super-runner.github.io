---
layout: post
title:  让openwrt自动重启网络接口
date: 2019-05-05
categories: linux
tags:  os
author: Jason Chi
---
* content
{:toc}




#### 创建自动执行shell脚本。 注意openwrt下要用sh而非bash。
```
root@lede:~# touch /usr/sbin/autoRestartNetwork
root@lede:~# chmod a+x /usr/sbin/autoRestartNetwork
```
#### 脚本内容
```
#!/bin/sh

PING_HOST=8.8.8.8
LOG_FILE=/usr/sbin/autoRestartNetwork.log
x=`ping -c1 $PING_HOST 2>&1 | grep "64 bytes from"`
if [ ! "$x" = "" ]; then
    echo "`date`: Network Good :)"
else
    echo "`date`: Restarting Network" >> $LOG_FILE
    /etc/init.d/network restart
    sleep 20
    x=`ping -c1 $PING_HOST 2>&1 | grep "64 bytes from"`
    if [ ! "$x" = "" ]; then
        echo "`date`: Network Fixed :)" >> $LOG_FILE
    else
        echo "`date`: Failed to restart network. Rebooting System :(" >> $LOG_FILE
        reboot
    fi
fi
```

#### 创建log文件
```
root@lede:~# touch /usr/sbin/autoRestartNetwork.log
```

#### 添加计划任务。这里设置每3分钟执行一次脚本。
```
root@lede:~# crontab -e
*/3 * * * * /usr/sbin/autoRestartNetwork
```
