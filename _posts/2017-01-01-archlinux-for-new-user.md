---
title: 给新手的 Arch Linux 桌面安装指南
tags: ArchLinux
---

适合有经验的新手(逃)

<!--more-->

## 验证boot模式
```bash
ls /sys/firmware/efi/efivars
```
如果存在目录，说明UEFI模式启动; 如果不存在，则是BIOS启动

## 测试网络
```bash
ping -c 3 www.archlinux.org
```

## 设置NTP
```bash
timedatectl set-ntp true
```

## 磁盘分区并格式化
```bash
   lsblk 查看磁盘 
   fdisk 分区
   sda1   EFI分区
   sda2    根分区
   sda3    home
   sda4    swap
   mkfs.vfat  /dev/sda1
   mkfs.ext4   /dev/sda2
   mkfs.ext4    /dev/sda3
   mkswap      /dev/sda4
```

## 挂载
```bash
mount /dev/sda2 /mnt
mount /dev/sda3 /mnt/home
mount /dev/sda1 /mnt/boot/efi
```
## 安装基本包,安装vim 
```
pacstrap /mnt vim
```
编辑/etc/pacman.d/mirrorlist 修改源为TUNA或者USTC
```
pacstrap -i /mnt base base-devel
```

## 生成fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```
编辑/mnt/etc/fstab 加入home和swap分区的UUID

## chroot
```
arch-chroot  /mnt
```

## 修改root密码,添加新用户
```
passwd root
useradd -m -s /bin/bash snowfrs
passwd snowfrs
```
使用命令`visudo`添加新用户

## 时区
```
ln -sf /usr/share/zoneinfo/Asia/Taipei   /etc/localtime
```   
硬件时钟 
```   
hwclock --systohc
```

## Locale

`vim /etc/locale.gen`

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GBK    GBK
zh_CN    GB2312
```
使用命令`locale-gen`生成信息

编辑 /etc/locale.conf

`LANG=en_US.UTF-8`

## 主机名

```
/etc/hostname	arch
/etc/hosts	127.0.0.1  arch.localdomain  arch
```

## 网络和基本组件
网络
```
pacman -S wpa_supplicant dialog
```
窗口管理器
```
pacman -S i3-wm i3status i3blocks i3lock rxvt-unicode dmenu
```
桌面显示管理器
```
pacman -S sddm
```

## 生成randisk
`mkinitcpio   -p linux`

## 引导器
```
pacman -S grub os-prober efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

## 开机启动
```
systemctl enable sddm
systemctl enable NetworkManager
```

## 退出chroot环境
```
umount -R /mnt/boot/efi
umount -R /mnt/home
umount -R /mnt
```

reboot重启 


## 其他
中文字体 
```
pacman -S wqy-zenhei wqy-microhei
```
等款字体
```
pacman -S adobe-source-code-pro-fonts
```
思源
```
pacman -S adobe-source-han-sans-otc-fonts adobe-source-han-sans-cn-fonts
adobe-source-sans-pro-fonts adobe-source-serif-pro-fonts
```
输入法
```
pacman -S fcitx-rime fcitx-configtool fcitx-im ttf-ubuntu-font-famil noto-fonts-emoji
noto-fonts-cjk
```
然后编辑 ~/.xprofile 添加如下
```
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
```
CPU和图形
```
pacman -S mesa mesa-demos xf86-video-intel
```
PDF查看
```
pacman -S zathura zathura-pdf-poppler
```

启用AUR

/etc/pacman.conf 中添加
```
[archlinuxcn]
SigLevel = Never
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
安装yaourt
```
pacman -Syu yaourt
```

如果你有yubikey

```
pacman -S libu2f-host
```

启动器		rofi

截图工具	scrot

文件管理	thunar/pcmanfm

终端查看图片 feh

系统信息输出  neofetch

如果你需要使用 $\TeX$ 文档
```
pacman -S texlive-most
yaourt -S acroread-fonts-systemwide
```

推荐使用i3管理器

目前安装的包 [详细列表](https://gist.github.com/snowfrs/a90abd855551d1b5a93e68a668f7a7db)

```
pacman -Qe
```
