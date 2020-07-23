---
title: 计算机启动过程介绍
tags: []
date: 2012-10-02 11:48:30
permalink: computer-boot
---

本文介绍从按下电源按钮到系统弹出登录界面, 计算机发生了什么.

<!--more-->
```
1. BIOS

2. MBR

3. Boot Loader

4. Kernel

5. /etc/inittab

6. /etc/rc.sysinit

7. /etc/rcX.d

8. /etc/rc.local

9. /bin/login
```
![pxe-boot-flow](/assets/img/blog/pxe-boot-flow.png)

当我们按下电源按钮时, 电源就开始向主板和其他设备供电, 此时电压不太稳定, 主板控制芯片组会向CPU发出并保持一个RESET信号, 让CPU内部自动恢复到初始状态. 当主板控制芯片组检测到电源稳定供电时, 它就撤去RESET信号, CPU就马上开始执行指令, 此指令地址一般为系统BIOS地址范围, 将控制权交给BIOS. 

 系统BIOS的启动代码首先要做的事情就是POST(Power-on Self Test 加电自检), POST的主要任务是检测系统的一些关键设备是否存在和能否正常工作, 比如显卡, 内存等设备. 由于POST是最早进行的检测过程, 此时显卡还没有初始化, 如果系统BIOS在进行POST的过程中发现了一些致命错误, 比如没有找到内存或者内存有问题(640k常规内存), 那么系统BIOS就会直接通过蜂鸣器发出警告, 根据蜂鸣器发出声音的长短和次数也可以判别错误类型. 如果POST通过, 系统BIOS会查找显卡BIOS, 找到后调用显卡BIOS的初始化代码,此时显示屏已经有显示信息啦. 显卡检测成功后系统BIOS会执行其他设备的检测, 完全检测通过后系统BIOS重新执行自己的代码, 获取系统控制权. 此后系统BIOS会检测系统的标准硬件, 比如硬盘, 软驱, 串行接口和并行接口等, 再之后就是检测即插即用设备,如果有则为即插即用设备分配中断, DMA通道和I/O端口等资源. 到了这里, 所有的设备都检测完毕. 如果和上次启动相比, BIOS发现系统硬件出现变动, 则BIOS还会更新ESCD(Extended System Configuration Data 扩展系统配置数据). ESCD是系统BIOS用来与操作系统交换硬件配置信息的数据, 这些信息被存放在CMOS中. 系统BIOS执行的最后一个工作是根据用户指定的启动顺序, 从硬盘, 光驱或者USB启动系统.

 BIOS非常重要, 它包含了CPU的相关信息, 设备启动顺序信息, 硬盘信息, 内存信息, PnP特性等.

如果系统BIOS执行的最后一步是从硬盘启动系统. 则系统会查找MBR(Master Boot Record 主引导记录, 硬盘的0磁道0磁头1扇区被称为MBR), MBR的大小是512 Byte, 512 Byte= 446 Byte + 64 Byte + 2 Byte. 前446 Byte是调用操作系统的机器码, 中间64 Byte是磁盘分区信息表, 最后2 Byte是主引导记录签名(0x55和0xAA). 系统找到硬盘MBR后, 就会将其复制到0x7c00地址所在的物理内存, 其实复制到物理内存的内容就是Boot Loader. 具体到计算机, 就是lilo或者grub. 我们常见的为grub, 用户从grub选择要启动的操作系统后, 控制权就会转移到操作系统.

根据grub设定的内核映像所在路径, 系统读取内核映像, 解压缩并将解压后的内核放到内存中, 调用start_kernel()函数来启动一系列的初始化函数并初始化各种设备, 完成linux核心环境的建立. 此时, 基于linux的程序可以正常运行啦. 

内核加载后, 第一个运行的程序是/sbin/init, 该程序会读取/etc/inittab文件, 根据此文件进行初始化工作. 该程序的作用是确定linux的运行等级. linux的运行等级为0-6, 都有不同的含义. init的pid为1, 它是所有其他进程的父进程. linux启动的第一个进程都是init.

确定运行等级后, linux执行的第一个用户层文件是/etc/rc.sysinit脚本程序, 它的工作非常多, 包括设定PATH, 设定网络配置(/etc/sysconfig/network), 启动swap分区, 设定/proc等. 具体信息可以到/etc路径中查看rc.sysinit文件.

启动内核模块. 依据/etc/modules.conf和/etc/modules.d目录下的文件来启动内核模块.

执行不同运行级别的脚本程序. /etc/rcX.d    (此处X是指linux的运行等级0-6)

执行用户定义的脚本程序. /etc/rc.local

执行/bin/login脚本, 等待用户输入.
