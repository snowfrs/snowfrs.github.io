---
title: Neighbor Discovery Protocol
tags: NDP
---
[RFC4861](https://tools.ietf.org/html/rfc4861)
<!--more-->

Neighbor Discovery Protocol (NDP, ND) 是在IPv6环境下使用的协议. NDP工作在链路层, 并负责搜集Internet通信所需要的各种信息, 包括本地连接的配置以及用于与更远系统通信的域名服务器和网关.

RFC4861 定义了五种类型的ICMPv6报文.

+ RS - Router Solicitation (Type 133) 主机通过路由器请求信息定位链路上的路由器. 路由器定期发送路由器通告(RA), 但在收到RS时会立即生成路由器通告信息.
+ RA - Router Advertisement (Type 134) 路由器定期或响应路由器请求信息来通告路由器的存在以及各种链接和Internet参数.
+ NS - Neighbor Soliciation (Type 135) 节点使用邻居请求来确定邻居的链路层地址, 或通过缓存的链路层地址来验证邻居是否仍可访问.
+ NA - Neighbor Advertisement (Type 136) 节点使用邻居通过来响应邻居请求信息.
+ Redirect (Type 137) 路由器可以通知主机使用更好的第一跳路由器.

这五种ICMPv6报文共同实现以下功能:

+ 路由器发现. 主机可以找到位于链路上的路由器
+ 前缀发现. 主机可以发现网络地址前缀
+ 参数发现. 主机可以找到链接参数(比如MTU)
+ 地址自动配置. 网络接口地址的可选无状态配置
+ 地址解析. IP地址和链路层地址之间的映射
+ 下一跳. 主机可以找到目的地的下一跳路由器
+ 邻居不可达检测 (NUD). 确定链路上邻居是否可达
+ 重复地址检测(DAD). 节点可以检查地址是否已被使用
+ 通过RA option 分配递归DNS服务器(RDNSS)和DNS搜索别表(DNSSL) (并非所有客户都支持)
+ 数据包重定向 可为某些目的地提供更好的下一条路由.