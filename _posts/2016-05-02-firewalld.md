---
title: firewalld 介绍
tags: firewalld
---

<!--more-->
## firewalld介绍
firewalld随RHEL7一同推出,作为iptalbes的前端控制器。firewalld是iptables的一个封装。

RHEL8开始firewalld后端由iptalbes更换为nftables。

- firewalld使用区域和服务而不是链式规则
- 动态管理规则集,允许更新规则而不破坏现有会话

做完操作后记得执行
`sudo firewall-cmd --reload`

检查防火墙状态
```
sudo firewall-cmd --state
```
输出应该是`running`或`not running`

查看firewalld守护进程状态
```
sudo systemctl status firewalld
```

## 常用firewall-cmd命令

```
firewall-cmd --get-zones
firewall-cmd --get-default-zone
firewall-cmd --set-default-zone
firewall-cmd --get-active-zones
firewall-cmd --list-all
firewall-cmd --list-rich-rules
```
## rich rule
`man 5 firewalld.richlanguage`
General rule structure
```
rule
	[source]
	[destination]
	service|port|protocol|icmp-block|icmp-type|masquerade|forward-port|soure-port
	[log]
	[audit]
	[accept|reject|drop|mark]
```
举例

接受所有192.168.1.0/24端口范围7900-7905的TCP流量
```
firewall-cmd --permanent --zone=public --ad-rich-rule="rule family=ipv4 source
address=192.168.1.0/24 port port=7900-7905 protocol=tcp accept"
```
伪造NAT
```
firewall-cmd --permanent --zone=<ZONE> --add-masquerade
firewall-cmd --permanent --zone=<ZONE> --add-rich-rule='rule family=ipv4 source
address=192.168.0.0/24 masquerade'
```
记录日志

将来自192.168.1.0/24的ssh流量记录到日志，每分钟最多写50条日志记录，log level
为info或更高级别的才写入日志
```
firewall-cmd --permanent --zone=<ZONE> --add-rich-rule='rule family=ipv4 source
address=192.168.1.0/24 service name =ssh log prefix=ssh level=info limit value=50/m accept'
```
## 高级用法
目标：将来自 114.114.0.0/16 的流量ban掉

初始操作
```
firewall-cmd --permanent --new-ipset=networkblock --type=hash:net --option=maxelem=10000000
--option=family=inet --option=hashsize=4096
firewall-cmd --permanent --zone=drop --add-source=ipset:networkblock
firewall-cmd --reload
```
ban掉指定流量
```
firewall-cmd --permanent --ipset=networkblock --add-entry=114.114.0.0/16
firewall-cmd --reload
```
