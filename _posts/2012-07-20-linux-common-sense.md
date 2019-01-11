---
title: Linux 基础学习
tags: []
date: 2012-07-20 22:29:40
permalink: linux-common-sense
---

**Everything is a file**

<!--more-->

/dev/sda1	第一块设备第一个分区

/dev/sda5	第一块设备扩展分区的第一个逻辑分区

硬盘 每个扇区 512 Byte 第一扇区MBR最重要

MBR Master Boot Record 硬盘主引导记录

硬盘分区表 DPT Disk Partition Table  占64字节

Magic Number  2字节 固定为 55AA

MBR  512 = 446+64+2



type 用于查看命令是内部命令还是外部命令

tee  -a 追加保存

tr 'a-z' 'A-Z' 大小写转换

cut -d 指定分隔符 -f 指定字段 -c 用字符串分隔

cut -d: -f3 /etc/passwd

没有-d选项时 默认TAB分隔符

sort 排序 默认按字母排序

-n 按数字排序 -k 字段 -t 分隔符

sort -n -k3 -t:	/etc/passwd

-r 反向排序 -u 重复行只显示一次

uniq 连续相同的行只显示一行

sort -u 重复行只显示一次，不管是否联系

uniq -c 重复的次数 -d 只显示重复的行



ps -aux	显示所有进程

pstree	显示进程树

ps -ef 	显示所有进程

pgrep -u root	只显示root产生的进程

pidof	显示某一个进程的进程号

kill -l	显示所有64种信号
常用：
1	重新加载

15	完全退出

9	强制杀死

18	继续

19	stop

killall -9 httpd	强制杀死所有httpd进程

进程优先级	数字越小	，优先级越大

只有root可以调节优先级，普通用户只能将优先级数字调大

nice -n 5 command	开启一个进程，给予优先级5

renice 5 pid			重新赋予优先级

bg %1	作业号1，继续放到后台运行

fg %1	作业号1放到前台与性

kill -9 %1	强制杀死作业号1



echo $?	显示上次命令执行的返回值
0 代表成功执行 127 代表此命令找不到 其他表示没有成功执行

test -f -d -e -r -w -x -n -z	测试

-f 文件 -d 目录 -e 文件或目录

-r 可读 -w 可写 -x 可执行

-n 是否非空值 -z 是否空值

