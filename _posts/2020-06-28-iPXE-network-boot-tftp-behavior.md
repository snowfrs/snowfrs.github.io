---
title: iPXE 网络启动中关于tftp的解释
tags: iPXE tftp
---
<!--more-->

iPXE 网络启动中经常发现tftp error code 8

```txt
Apr  3 17:14:29 localhost kernel: txgbe 0000:02:00.0 enp2s0f0: NIC Link is Down

Apr  3 17:14:29 localhost kernel: txgbe 0000:02:00.0 enp2s0f0: NIC Link is Up 10 Gbps, Flow Control: RX/TX

Apr  3 17:14:30 localhost dhcpd: Solicit message from fe80::b2d5:9dff:fe5a:1f47 port 546, transaction ID 0x47252A00

Apr  3 17:14:30 localhost dhcpd: Sending Advertise to fe80::b2d5:9dff:fe5a:1f47 port 546

Apr  3 17:14:30 localhost dhcpd: Request message from fe80::b2d5:9dff:fe5a:1f47 port 546, transaction ID 0x47252B00

Apr  3 17:14:30 localhost dhcpd: Sending Reply to fe80::b2d5:9dff:fe5a:1f47 port 546

Apr  3 17:14:30 localhost in.tftpd[2599]: Error code 8: User aborted the transfer

Apr  3 17:14:30 localhost in.tftpd[2600]: Client 3ffe:501:ffff:100::40 finished ipxe.efi

Apr  3 17:14:30 localhost dhcpd: Release message from fe80::b2d5:9dff:fe5a:1f47 port 546, transaction ID 0x47252C00

Apr  3 17:14:30 localhost dhcpd: Client 00:01:00:01:22:51:d0:d0:30:09:f9:20:2c:f0 releases address 3ffe:501:ffff:100::40

Apr  3 17:14:30 localhost dhcpd: Sending Reply to fe80::b2d5:9dff:fe5a:1f47 port 546
```

[RFC2347](https://tools.ietf.org/html/rfc2347) 中关于tftp过程有详细解释.

客户端可以带多个tftp option对 `[opt1,value1]`...`[optN,vauleN]` 且option的顺序不重要.

服务端收到并能正确识别option时, 回应OACK(Option Acknowledgment)包. 服务端OACK包只包含客户端请求的option字段, 不能包含未请求的option字段。

也就是说, 只能由客户端发起option初始化协商.

如果option被服务端识别但是invalid, 服务端的OACK中会携带一个正确的值. 客户端必须使用服务端OACK中的值, 并且不能使用未被服务端识别的option字段.


下面的例子中

客户端发起read request(opc=1), 请求filename是ipxe.efi, mode是octet. 同时携带两个option: tsize=0, blksize=1228

transfer size 客户端只知道filename并不知file size

服务端OACK(opc=6)时携带option: tsize=966176,blksize=1228

客户端发起第二次read request, 只携带一个option: blksize=1228

![tftp-flow](/assets/img/blog/ipxe/Tftp-flow.jpg)

tsize = 966176 bytes

blksize = 1228 bytes

需要 966176/1228 = 787 (向上取整).

![tftp-flow-2](/assets/img/blog/ipxe/Tftp-flow-2.jpg)