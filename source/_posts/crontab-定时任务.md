---
title: crontab 定时任务
date: 2017-02-13 14:43:34
tags: linux
layout: crontab
slug: linux-crontab
---

1 检查  crontab -l 是否安装
 
 检查crond服务是否启动 service crond status

 安装cron
 
`
yum install vixie-cron
`

`
yum install crontabs
`

2.开始
crontab -e
测试每分钟打印到文件中
*/1 * * * * date >> /tmp/log.txt

`
crontab -l  可以查看crontab 任务
`

查看日志

`
/var/mail
`

开机执行
`
@reboot 执行 xx.sh
`

或者

`
vim /etc/re.d/re.locate 执行  xx.sh
`

 >
*/1 * * * * date >> /tmp/log.txt
* * * * * COMMAND
分钟0–59  小时 0-23 日期 1-31 月份 1-12 星期 0-7

每晚 21:30 重启服务
30 21 * * *

每月1 ，10 22 日 4:45 重启
45 4 1,10,22 * *

每月1到10日 4:45  重启
45 4 1-10 * *

每2分钟 重启
*/2 * * * *

1-59/2 * * * *

晚上11点到 早7点 每隔一小时重启
0 23-7/1 * * *

每天 18:00 - 23:00 每30分钟
0,30 18-23 * * *
0-59/30 18-23 * * *
 >