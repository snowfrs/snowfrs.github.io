---
title: syslinux引导bootmgr
tags:
  - syslinux
date: 2016-01-17 17:08:24
permalink: syslink-to-bootmgr
---

使用syslinux多启动时, 添加WIN7后竟然转到grub4dos 提示命令行输入.

于是:
采用syslinux官方提供的chain.c32模块
只需要更改syslinux.cfg的内容类似下面

LABEL win7_x64_install
MENU LABEL Windows 7 x64 Install
COM32 chain.c32
APPEND fs ntldr=/bootmgr

引导至bootmgr后便可以进行WIN系统安装.


​    