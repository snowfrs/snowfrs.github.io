---
title: 双系统进入grub救援模式解决办法
tags:
  - grub
  - dual-boot
  - windows
  - archlinux
---
  目前使用ThinkPad
  T480作为主力机，安装了win10和archlinux双系统。Win10更新频繁，每次大更新之后grub总是进入救援模式，记录一下，方便以后。
<!--more-->
 开机进入grub救援模式后，ls找到磁盘分区相关信息. 我的EFI分区是单独存在的。
 ```
 set prefix=(hd1,gpt5)/boot/grub
 set root=(hd1,gpt5)
 insmod normal
 normal
```
  进入系统后，重新安装grub
```
grub-install /dev/nvme0n1 --target=x86_64-efi --efi-directory=/boot/efi --boot-loader=archlinux
grub-mkconfig -o /boot/grub/grub.cfg
```
关键步骤
```
cp /boot/efi/EFI/archlinux/grubx64.efi /boot/efi/EFI/Boot/bootx64.efi
```
 
