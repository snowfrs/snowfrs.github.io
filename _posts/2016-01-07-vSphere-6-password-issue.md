---
title: 安装VMware vSphere 6.0中出现的问题
tags:
  - ESXi
  - vCenter
  - Vsphere
date: 2016-01-07 16:26:20
permalink: vSphere-6-password-issue
---

主机Windows 10 Pro, 装虚拟机VMware WorkStation 12.
在VM 12上首先装ESXi 6.0, 然后导入vCenter Host Gateway 6.0.
导入完成后, 发现vCenter使用root/vmware不能登录. &nbsp;密码输入错误5次账户还被锁定了. &nbsp;于是进入grub设置参数修改root密码.
1\. 开机出现GRUB提示时, 按上下箭头禁止自动引导;
2\. 选中正常启动项 按e进入编辑;
3\. 按上下箭头滚动到第二行显示内核引导参数, 按e编辑引导命令;
4\. 将 init=/bin/bash附加到内核引导选项, 按enter确认
5\. 按b启动引导 &nbsp; 直至进入shell
6\. passwd root重置密码
7\. reboot重启机器

后来查阅资料得知, 自vCSA 5.5U1开始, 本地密码90天强制过期.