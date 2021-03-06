---
title: Linux Performance Analysis
tags: performance
---

**Brendan D. Gregg**

[http://www.brendangregg.com/](http://www.brendangregg.com/)
<!--more-->

![Linux_observability_tool](/assets/img/blog/linux_performance/Linux_observability_tools.png)

1. uptime
2. `dmesg | tail`
3. vmstat 1
4. mpstat -P ALL 1
5. pidstat 1
6. iostat -xz 1
7. free -h
8. sar -n DEV 1
9. sar -n TCP,ETCP 1
10. top (htop,atop)

**<u>Utilization 使用率 </u>**

**<u>Saturation  饱和度 </u>**

**<u>Errors      错误 </u>**

## uptime
```bash
# uptime 
13:33:46 up 31 days,  4:22,  8 users,  load average: 74.46, 81.67, 77.75
```
显示系统 当前时间 开机多久 系统负载

## dmesg | tail

## vmstat 1
```bash
# vmstat 1
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
70  0 336584 290828  15840 313216    1    2     6     8    7    6 12  5 82  0  0	
91  0 336584 291812  15840 313244    0    0     0     0 46875 10517 44 56  0  0  0	
102  0 336584 296656  15840 313244    0    0     0     0 47664 11342 42 58  0  0  0	
84  0 336584 295044  15840 313252    0    0     0     0 47048 12425 44 56  0  0  0	
100  0 336584 320960  15840 313252    0    0     0     0 45942 11984 44 57  0  0  0	
60  0 336584 321844  15840 313256    0    0     0     0 46097 13476 46 54  0  0  0	
89  0 336584 333152  15848 313256    0    0     0    28 46286 12980 46 54  0  0  0
```

虚拟内存简短展示, 参数1表示每1秒打印一次.

r: CPU等待运行的进程数. 这个指标可以判断CPU饱和度(不包含I/O等待的进程)

## mpstat -P ALL 1
```bash
# mpstat -P ALL 1
Linux 2.6.32-754.28.1.el6.x86_64 (hostname) 	05/23/2020 	_x86_64_	(24 CPU)

01:55:36 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
01:55:37 PM  all   33.75    0.00    1.00    0.04    0.00    0.00    0.00    0.00   65.20
01:55:37 PM    0   29.59    0.00    5.10    1.02    0.00    0.00    0.00    0.00   64.29
01:55:37 PM    1   70.00    0.00    4.00    0.00    0.00    0.00    0.00    0.00   26.00
01:55:37 PM    2   90.00    0.00    2.00    0.00    0.00    0.00    0.00    0.00    8.00
01:55:37 PM    3    2.02    0.00    2.02    0.00    0.00    0.00    0.00    0.00   95.96
01:55:37 PM    4  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
01:55:37 PM    5    1.01    0.00    1.01    0.00    0.00    0.00    0.00    0.00   97.98
01:55:37 PM    6   99.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00
01:55:37 PM    7    1.01    0.00    1.01    0.00    0.00    0.00    0.00    0.00   97.98
01:55:37 PM    8    3.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00   96.00
01:55:37 PM    9    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
01:55:37 PM   10    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
01:55:37 PM   11   99.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00
01:55:37 PM   12    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
01:55:37 PM   13  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
01:55:37 PM   14    0.00    0.00    0.99    0.00    0.00    0.00    0.00    0.00   99.01
01:55:37 PM   15    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
01:55:37 PM   16   98.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    2.00
01:55:37 PM   17    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
01:55:37 PM   18   16.00    0.00    6.00    0.00    0.00    0.00    0.00    0.00   78.00
01:55:37 PM   19    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
01:55:37 PM   20   99.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00
01:55:37 PM   21    0.00    0.00    0.99    0.00    0.00    0.00    0.00    0.00   99.01
01:55:37 PM   22    0.00    0.00    0.99    0.00    0.00    0.00    0.00    0.00   99.01
01:55:37 PM   23    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

```
打印各个CPU时间统计.

## pidstat 1
```bash
# pidstat 1
Linux 2.6.32-754.28.1.el6.x86_64 (hostname) 	05/23/2020 	_x86_64_	(24 CPU)

01:58:20 PM       PID    %usr %system  %guest    %CPU   CPU  Command
01:58:21 PM      3897    0.96    0.96    0.00    1.92    14  sge_execd
01:58:21 PM    850715    0.96    0.00    0.00    0.96     6  sge_shepherd
01:58:21 PM    850815   99.04    0.00    0.00   99.04    21  simv
01:58:21 PM    854828    0.96    0.00    0.00    0.96    10  sge_shepherd
01:58:21 PM    854928  100.00    0.00    0.00  100.00     7  simv
01:58:21 PM    855131    0.96    0.00    0.00    0.96    12  sge_shepherd
01:58:21 PM    855231   99.04    1.92    0.00  100.00    22  simv
01:58:21 PM    855396    0.00    1.92    0.00    1.92    13  sge_shepherd
01:58:21 PM    855497   99.04    0.96    0.00  100.00     3  simv
01:58:21 PM    855498    0.96    0.96    0.00    1.92    11  sge_shepherd
01:58:21 PM    855599   99.04    0.00    0.00   99.04     4  simv
01:58:21 PM    857515  100.00    0.96    0.00  100.00     1  simv
01:58:21 PM    857516    0.00    0.96    0.00    0.96    11  sge_shepherd
01:58:21 PM    857616   98.08    0.00    0.00   98.08    20  simv
01:58:21 PM    857718    0.96    0.96    0.00    1.92    14  sge_shepherd
01:58:21 PM    857818   98.08    0.96    0.00   99.04    17  simv
01:58:21 PM    857819    0.96    0.00    0.00    0.96    13  sge_shepherd
01:58:21 PM    857919  100.00    0.00    0.00  100.00    14  simv
01:58:21 PM    858020  100.00    0.00    0.00  100.00     8  simv
01:58:21 PM    858023    1.92    0.96    0.00    2.88    22  sge_shepherd
01:58:21 PM    858124   99.04    0.96    0.00  100.00    13  simv
01:58:21 PM    858126    0.96    0.96    0.00    1.92     0  sge_shepherd
01:58:21 PM    858226   99.04    0.96    0.00  100.00    23  simv
01:58:21 PM    858237    0.96    0.00    0.00    0.96    14  sge_shepherd
01:58:21 PM    858239    0.96    0.96    0.00    1.92    16  sge_shepherd
01:58:21 PM    858346   99.04    0.96    0.00  100.00    11  simv
01:58:21 PM    858448   99.04    0.96    0.00  100.00     6  simv
01:58:21 PM    858449    0.00    0.96    0.00    0.96     2  sge_shepherd
01:58:21 PM    858549   99.04    0.00    0.00   99.04     2  simv
01:58:21 PM    858550    0.96    0.96    0.00    1.92    14  sge_shepherd
01:58:21 PM    858650   98.08    0.96    0.00   99.04    15  simv
01:58:21 PM    858695    0.96    2.88    0.00    3.85    18  pidstat

```
%CPU 表示所有CPU的使用率；

## iostat -xz 1
```bash
# iostat -xz 1
Linux 2.6.32-754.28.1.el6.x86_64 (hostname) 	05/23/2020 	_x86_64_	(24 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          73.19    0.00    0.39    0.16    0.00   26.25

Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.12    15.44    0.59    5.50    21.99   162.40    30.26     0.05    7.84    0.67    8.61   0.38   0.23

```
展示了块设备的请求负载和性能数据.

r/s w/s 设备每秒请求读的次数和请求写的次数

await I/O等待的平均时间(ms)

avgqu-sz 设备上请求平均数。 数值大于1 可能表示设备饱和

%util 设备利用率 是一个百分比 数值大于60可能引起性能降低

## free -h
```bash
# free -h
             total       used       free     shared    buffers     cached
Mem:          186G        22G       163G       1.9M       267M       7.0G
-/+ buffers/cache:        15G       170G
Swap:           0B         0B         0B

```
buffers 用于块设备I/O缓冲的缓存

cached 用于文件系统的页缓存

有些人习惯使用 free -m

## sar -n DEV 1
```bash
# sar -n DEV 1
Linux 2.6.32-754.28.1.el6.x86_64 (hostname) 	05/23/2020 	_x86_64_	(24 CPU)

02:11:05 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
02:11:06 PM      eth0   1134.00   7265.00     73.24   1666.35      0.00      0.00      0.00

```
检测网络接口吞吐.

rxkB/s txkB/s 收发数据负载


## sar -n TCP,ETCP 1
```bash
# sar -n TCP,ETCP 1
Linux 2.6.32-754.28.1.el6.x86_64 (hostname) 	05/23/2020 	_x86_64_	(24 CPU)

02:15:17 PM  active/s passive/s    iseg/s    oseg/s
02:15:18 PM      1.00      1.00   1029.00   7264.00

```
TCP指标统计.

active/s 每秒本地发起的TCP连接数

passive/s 每秒远程发起的连接数

retrans/s 每秒TCP重传数


## top