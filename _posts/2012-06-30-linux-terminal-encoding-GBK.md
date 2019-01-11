---
title: 终端中文显示乱码
tags: terminal
---

Ubuntu 12.04 LTS 安装的的时候选的英文, 安装好之后解决了txt文本显示乱码问题. 但是在终端下 telnet 某BBS论坛, 终端下显示中文显示乱码.
<!--more-->

问题描述如下:

字符集编码：

1.BBS的输出始终是GBK;

2.终端采用哪种编码输出;

Linux 下可输入命令 local 以查看本机字符集编码。默认编码为UTF-8，因此无法正常显示GBK的内容。在这种情况下, 可以使用luit来转化编码：

```bash
$ luit -encoding GBK telnet Hostname
```

在Linux下，推荐使用Term程序上BBS, 例如qterm、pcmanx.
