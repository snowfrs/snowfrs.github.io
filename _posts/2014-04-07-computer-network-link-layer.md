---
layout: blog
title:  《计算机网络》笔记：数据链路层
tags: IP TCP 网络 路由 检错码 纠错码 滑动窗口
---

## 设计要点

* 为网络层提供服务：无确认的无连接服务、有确认的无连接服务、有确认的面向连接服务
* 处理传输错误
* 流控制：基于反馈的流控制、基于速率的流控制

<!--more-->

## 错误检测与纠正

### 纠错码

* 包含冗余信息，使接收方知道肯定包含哪些信息（前向纠错）
* 海明距离：两码字中不同位数
* 编码方案的海明距离：合法码字列表中最小的海明距离
* 纠错码；为检测d个错误，需要海明距离为d+1的编码方案；为纠正d个错误，需要海明距离为2d+1的编码方案
* 海明码：纠正单个错误、突发性错误（每次传送一列，分散错误）

### 检错码

* 包含冗余信息，使接收方知道发生了错误并请求重传。
* 多项式编码（polynomial code），也称CRC（cyclic redundancy check，循环冗余校验码）：在帧尾加入校验和使追加后的帧可以被生成多项式G(x)除尽。

## 基本数据链路协议

* 无限制的单工协议
* 单工的停-等协议：等待确认
* 有噪声的单工协议：PAR（positive acknowledgement with retransmission，支持重传的肯定确认协议）又称ARQ（automatic repeat request，自动重复请求协议），确认超时后重传

## 滑动窗口协议

* 稍待确认
* 发送窗口、接收窗口

### 1位滑动窗口协议

* 收到确认后发送下一帧

### 使用回退n帧技术的协议

* 回避往返延时
* 管道化技术：管道容量=带宽*往返延迟
* 回退n帧：接收方只接受下一帧
* 选择性重传：接收方缓存错误帧后的所有帧
* 否定的确认帧（NAK，negative acknowledgement）：避免发送方等待确认超时

## 协议验证

### 有限状态机模型

* 可达性分析、协议机、初始状态
* 模型内容
+ S：进程与信道可能的状态集合
+ M：能在信道上进行交换的帧的集合
+ I：进程初始状态的集合
+ T：状态之间转换的集合

### Peri网模型

* 模型内容
+ 库所（place）：状态
+ 变迁（transition）
+ 弧（arc）
+ 标记（token）：系统当前的状态
* 转换用垂直或水平线表示，标记用粗黑点表示，库所用圆圈表示，
* 激活的转换：转换的输入库所中有至少一个输入标记。激活的转换随时可以激发。
* 文法：转换用箭头表示，两边为输入输出库所，每一转换对应一条文法。eg. BD->AC

## 数据链路层协议示例

### HDLC

* 历史
+ IBMSDLC（synchronous data link control，同步数据链路控制）协议
* ADCCP（advanced data communication control procedure，高级数据通信控制规程）：ANSI修改
* HDLC（high-level data link control，高级数据链路控制）：ISO修改
* CCITT采纳并修改HDLC作为LAP（link access procedure，链路访问规程），及LAPB
* 面向位的协议的帧结构：分界标志序列+地址+控制+数据+校验和+分界标志序列
* 三种帧：信息帧、管理帧、无序号的帧

### Internet 中的数据链路层

* internet连接过程：PC中使用TCP/IP的进程->调制解调器->拨号电话线（使用PPP的TCP/IP连接）->调制解调器->路由器->路由选择进程...
* PPP（point-to-point protocol，点到点的协议）功能
+ 成帧的方法：无歧义分割、错误检测
+ 链路控制协议：LCP（link control protocol）
+ 协商网络层选项的方法：对每一支持的网络层给出NCP（network control protocol，网络控制协议）
* PPP帧格式与HDLC非常相似


转载自 <a href="https://harttle.land">Harttle.Land</a>
