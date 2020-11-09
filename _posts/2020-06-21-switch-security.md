---
title: 交换机安全配置
tags: switch security networking
---
<!--more-->

# 端口安全
```txt
   Switch>enable	
   Switch#configure terminal	
   Switch(config)#interface fastethernet 0/1	
   Switch(config-if)#switchport mode access	
   Switch(config-if)#switchport access vlan 10
   Switch(config-if)#switchport port-security	
   Switch(config-if)#switchport port-security maximum 1	
   #Set limit for hosts that can be associated with interface. Default value is 1. Skip this command to use default value.
   Switch(config-if)#switchport port-security violation shutdown	
   #Set security violation mode. Default mode is shutdown. Skip this command to use default mode.
   Switch(config-if)#switchport port-security mac-address sticky	
   #Enable sticky feature.
```
violation shutdown触发之后端口进入err-disable状态 可在全局设置err-disable自动恢复 
```txt
   #errdisable recovery cause psecure-violation
   #errdisable recovery cause mac-locking
   #errdisable recovery interval 30 	#(30s)
```
也可手动 shutdown / no shutdown 开启端口 

## 全局配置port-security 
```txt
switchport port-security
```
## 端口配置port-security 
```txt
   switchport port-security
   switchport port-securtiy maximum 3
   switchport port-security mac-address sticky 
   switchport port-security violation shutdown
```
# DHCP安全
开启dhcp snooping 只在接入层交换机开启 
```txt
   ip dhcp snooping
   ip dhcp snooping vlan 10
   ip dhcp snooping verify mac-address
```

```txt
   #untrusted port (end suer port)
   int fastethernet 1/0/1
   ip dhcp snooping limit rate 20
   #trusted port (trunk interface)
   int fastethernet 1/0/10
   ip dhcp snooping trust
```
原理：

When option 82 is enabled on the switch, then this sequence of events occurs when a DHCP client sends a DHCP request:

1. The switch receives the request and inserts the option 82 information in the packet header.
2. The switch forwards or relays the request to the DHCP server.
3. The server uses the DHCP option 82 information to formulate its reply and sends a response back to the switch. It does not alter the option 82 information.
4. The switch strips the option 82 information from the response packet.
5. The switch forwards the response packet to the client.

Suboption Components of Option 82

Option 82 as implemented on a switching device comprises the suboptions circuit ID, remote ID, and vendor ID. These suboptions are fields in the packet header:

circuit ID—Identifies the circuit (interface or VLAN) on the switching device on which the request was received. The circuit ID contains the interface name and VLAN name, with the two elements separated by a colon—for example, ge-0/0/10:vlan1, where ge-0/0/10 is the interface name and vlan1 is the VLAN name. If the request packet is received on a Layer 3 interface, the circuit ID is just the interface name—for example, ge-0/0/10.

Use the prefix option to add an optional prefix to the circuit ID. If you enable the prefix option, the hostname for the switching device is used as the prefix; for example, device1:ge-0/0/10:vlan1, where device1 is the hostname.


You can also specify that the interface description be used rather than the interface name or that the VLAN ID be used rather than the VLAN name.

remote ID—Identifies the remote host. See remote-id for details.

vendor ID—Identifies the vendor of the host. If you specify the vendor-id option but do not enter a value, the default value Juniper is used. To specify a value, you type a character string.


STP 生成树协议 防止二层交换机出现环路

在接入端口设置 
# mac-based VLAN
全局下 
```txt
    vlan 2
    vlan association mac xxxx.xxxx.xxxx
```
查看
```txt
   show vlan association mac
```