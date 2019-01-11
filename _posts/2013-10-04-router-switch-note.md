---
title: 路由交换学习笔记
tags:
  - Network
date: 2013-10-04 21:13:42
permalink: router-switch-notes
---

找到了以前的笔记，有点乱。

<!--more-->

什么是网络？

网络是通过网络中继设备提供终端间的通信，最终实现资源共享。

网络的组成部分：  getmac 只查看物理地址

网线：

线序：568B：白橙、橙、白绿、蓝、白蓝、绿、白棕、棕        

568A：将1、3和2、6线序对调即可

RJ-45接头的8个接脚的识别方法，铜接点朝自己，头朝右，从上往下数，分别是1、2、3、4、5、6、7、8.

1.       输出数据(+)

2.       输出数据(-)

3.       输入数据(+)

4.       保留为电话使用

5.       保留为电话使用、

6.       输入数据(-)

7.       保留为电话使用

8.       保留为电话使用

直通线：两端都是568A或者568B标准的双绞线

交叉线：一端是568A标准，另一端是568B标准的双绞线

做RJ-45水晶头时最好按照标准线序做，不然可能出现传输问题。

双绞线按照其是否外加金属网丝套的屏蔽层而区分为屏蔽双绞线(STP)和非屏蔽双绞线(UTP, Unshielded Twisted Pair)。             

三类、四类、五类双绞线：

三类：ANSI和EIA/TIA568 标准中指定的电缆，符合IEEE802.3 10Base-T的标准。传输频率为16MHz，用于语音传输及最高传输速率为10Mbps的数据传输。主要用于10Base-T.

四类：传输频率为20MHz,用于语音传输和最高传输速率为16Mbps的数据传输，主要用于基于令牌的局域网和10Base-T/100Base-T。

五类：符合IEEE802.3 100Base-T的标准，传输频率为100MHz，用于语音传输和最高传输速率为100Mbps的数据传输，主要用于100Base-T和10Base-T网络，这是最常用的以太网电缆。

串行链路：DCT  DTE  时钟频率(clock rate)  串行链路默认封装HDLC 

组网标准：以太网组网标准。  凡是以CSMA/CD标准防止产生冲突的网络统称为以太网。

CSMA/CD：

第一个计算机网络：ARPANET(The Advanced Research Projects Agency Network).

数据传输方式：单播、组播、广播

单播：检查链路通讯，严重对端是否存在

组播：对特定范围对象同步数据

广播：寻址

区分ISO  OSI  IOS

ISO：国际标准化组织  International Organization for Standardization

OSI：开放式通信系统互联参考模型  Open System Interconnection Reference Model

IOS：互联网操作系统  Internet work Operating System

OSI七层参考模型：

1.       物理层                                           比特流     电信号

2.       数据链路层                                  MAC地址

3.       网络层                                           IP地址

4.       传输层                                           TCP  UDP

5.       会话层                                           分类

6.       表示层                                           加密

7.       应用层

TCP/IP模型：

网络接口层：                                     对应OSI 物理层+数据链路层

网络层：                                               对应OSI网络层

主机到主机层：                                 对应OSI传输层

应用层：                                               对应OSI会话层+表示层+应用层

常用知名端口：(C:\Windows\System32\drivers\etc\services)

FTP: 20 21

HTTP: 80

DNS: 53

TELNET: 23

POP3: 110

SMTP: 25/TCP

SNMP: 161/UDP

关于设备登录方式：

1.       本地登录  CONSOLE口

2.       异地登录  Telnet

3.       WEB图形化登录  Cisco SDM

IP地址：点分十进制

A类：1 – 127                                       默认子网掩码：255.0.0.0

B类：128 – 191                                  默认子网掩码：255.255.0.0

C类：192 – 223                                  默认子网掩码：255.255.255.0

D类：224 – 239                                  组播地址，无子网掩码

E类：240 – 255                                  科研地址

255.255.255.255 全网广播地址

子网掩码规则[2进制]：1. 从左到右必须为连续的1

​                                                2. 右边的数必须小于左边的数

0       128  192  224  240  248  252  254  255 这9个数是 合格 的子网掩码数

计算IP地址是否在同一网段：

计算机只能进行三种运算：与、或、非

1.       将IP地址和子网掩码转换成2进制

2.       将IP地址和子网掩码分别按位进行与运算

3.       对比结果，结果相同则认为是同一网段。

子网掩码中，全1部分为网络位，全0部分为主机位。

将IP地址和子网掩码按位与运算得出的结果即为网络号；

将子网掩码中主机位对应到IP地址中，将主机位部分全部置为1，即为广播地址。

主机数：2^N           (N为主机位数)

有效主机数：2^N-2

CISCO基础：

将命令转化为字符： CTRL+V  再打问号

设备配置密码   enable password #######  或者 enable secret ########

CISCO密码还原：

1.       手工重启设备。在读取进程时，按下CTRL+BREAK键，进入设备监控模式 rommon 1>

或者 CTRL+C键

2.       监控模式下输入 confreg 0x2142

设备开机执行0x2142配置文件  0x2142原始配置文件  0x2102保存过的配置文件

Running-config           当前配置文件         易失性内存

Startup-config            保存配置文件         非易失性内存

3.       监控模式下重启设备  rommon 2> boot

4.       将之前配置复制到当前配置         copy startup-config running-config

删除密码        no enable secret

5.       启动设备时参数更改     config_register  0x2102

设定之后重启设备读取0x2102 配置文件

重启设备： reload    终止错误命令：CTRL+SHIFT+6

DHCP建立过程：服务端 S     客户端 C

1.       C向S发送一个discovery数据包寻找Server端  数据包(源目IP，源目MAC，协议)

2.       S收到C发送的数据包后，立即回送offer数据包  (源IP：Server IP 目IP：255.255.255.255)

3.       C收到S回送的offer数据包后，会进行一次检测，检测本地网卡是否存在IP地址，若无IP地址，C发送request请求数据包(源IP 0.0.0.0 目标IP  Server IP)

4.       S收到C发送的请求数据包，回送reply数据包(源IP：Server IP 目标IP：Clinet IP)

5.       C收到reply数据包后，进行检测，若本地网卡无IP地址，则将reply数据包中的IP地址加入自己网卡中，惊且激活该网卡。

DHCP中继：

DHCP服务器分发不同网段的地址，通过中继设备进行定向转发——路由器。

路由器默认情况下不支持转发广播数据包，将广播转换成单播

Router(config-if)#ip help-address ####  将广播数据包转换成单播给####

配置：

​         #ip dhcp pool name

​         #network 192.168.1.0   /24

​         #default-router  ?                       网关

​         #dns-server     ?             

​         #lease  1                              租期

​         全局下排除不想分配的地址：

​         #ip dhcp excluded_address     ?

远程登录telnet  (vty)

Line vty 0 4   开启远程登录线程，允许5个人登录

Password   cisco   为远程登录设置密码

Privilege level #   默认情况所有远程登录的用户权限等级为1，只能停留在用户模式；等级2 – 14 可进入特权模式；等级为15 可进入全局配置模式。

Router(config-line)#login local                本地数据库认证

本地创建认证数据库，全局模式：  username ### privilege # password ###

Show users  显示当前在线用户

Clear line vty #  清除线程vty #

CDP  直连邻居发现协议：

Show cdp neighbors   CDP每60s更新一次，180s刷新

Show cdp neighbors detail  邻居详细信息

全局#no cdp run  关闭发送邻居CDP数据包

地址解析协议 ARP

将IP地址和MAC地址绑定，映射到自己的ARP表中

ARP首次建立过程：PCA:  1.1.1.1     AAAA        PCB:    1.1.1.2      BBBB

PCA需要访问PCB，此时查找本地ARP映射表，发现表中不存在PCB的映射信息，因此发送ARP请求数据包，解析PCB的MAC

1.       PCA发送ARP请求数据包(源IP：1.1.1.1 目标IP: 1.1.1.2  源MAC：AAAA 目的MAC：FFFF)
  show mac-address-table           查看交换机MAC地址

2.       当交换机收到PCA发送的广播数据包后，进行广播转发，并将PCA的MAC地址和对应端口加入自己的MAC地址表中。

3.       当PCB收到PCA发送的请求数据包后，进行回复reply数据包。
  源IP：1.1.1.2          目标IP：1.1.1.1    源MAC:  BBBB        目标MAC：AAAA

4.       当交换机收到PCB发送的reply数据包后，定向转发给PCA。并将PCB的MAC地址和端口加入自己的MAC地址表中。

5.       当PCA收到PCB回送的reply后，将reply中的源IP源MAC进行绑定，并映射到本地ARP表中

什么是路由？   当一个接口收到数据时，根据数据包的目标IP进行定向转发的过程叫路由.

路由选择原则：

1.       最长子网掩码匹配         [1/0]  管理距离/开销值

2.       管理距离                                       管理距离衡量该协议的可靠度  越小越优

3.       开销值                                           源到目的地所需花费          越小越优

4.       负载均衡

路由协议：   IGP 内部网关路由协议 EGP 外部网关路由协议

​                            DV 距离矢量路由协议 LS链路状态路由协议

更新接口：1. 能接收和发送更新数据包的接口

​                     2. 能根据自己IP地址产生子网路由条目的接口   必须同时满足

链路工作状态：单工双工 半双工

双工：链路两端接口可同时进行数据收发

单工：链路端口只能接收/发送数据包

半双工：链路端口在一定时间段内只能接收  收完后可发送

Duplex auto/half/full  链路两端工作状态必须相同，否则链路协议状态为DOWN

Status表示接口打了no shutdown命令，protocol up 表示协议端口参数配置正确

\#show  ip  interface f0/0               三层信息

\#show  interface  f0/0                   二层信息

\#show  controllers f0/0                 物理特性

简单排错：

1.    查看接口IP地址配置情况              #show ip interface brief

2.    查看不正常端口的详细信息        #show interface f0/0

3.    查看接口配置命令                             #show running interface f0/0  接口下是否挂载ACL

4.    查看访问控制列表配置情况                    #show ip access-list

5.    查看协议配置命令                                       #show running-config | begin router ospf/rip/eigrp

6.    查看单条路由条目                                       #show ip route | include 192.168.81.0

7.    #show ip route                     #show running-config

产生默认路由的两种方式：

\#ip default-network 192.168.81.0 或者         #ip route 0.0.0.0     0.0.0.0    192.168.91.146