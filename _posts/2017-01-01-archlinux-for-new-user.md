---
title: 给新手的 Arch Linux 桌面安装指南
date: 2017-01-01T16:45:59.000Z
permalink: archlinux-for-new-user.html
tags: linux
---

适合有经验的新手(逃)

<!--more-->

1. 验证boot模式
   `ls /sys/firmware/efi/efivars`
   如果存在目录，说明UEFI模式启动; 如果不存在，应该时BIOS启动


2. 测试网络
   `ping -c 3 www.archlinux.org`

3. 设置NTP
   `timedatectl set-ntp true`

4. 磁盘分区并格式化
   lsblk 查看磁盘 fdisk 分区
   sda1   EFI分区
   sda2    根分区
   sda3    home
   sda4    swap
   mkfs.fat32  /dev/sda1
   mkfs.ext4   /dev/sda2
   mkfs.ext4    /dev/sda3
   mkswap      /dev/sda4

5. 挂载
   `mount /dev/sda2 /mnt`
   `mount /dev/sda3 /mnt/home`
   `mount /dev/sda1 /mnt/boot/efi`

6. 安装基本包
   先安装vim 
   `pacstrap /mnt vim`
   编辑/etc/pacman.d/mirrorlist 修改源为TUNA或者USTC

   `pacstrap  -i	/mnt	base base-devel`

7. 生成fstab
   `genfstab -U /mnt   >>    /mnt/etc/fstab`
   编辑/mnt/etc/fstab 加入home和swap分区的UUID

8. chroot
   `arch-chroot  /mnt`
   修改root密码
   passwd root
   添加新用户/修改用户密码/添加到sudo user
   useradd -m -s /bin/bash snowfrs
   passwd snowfrs
   使用命令visudo添加新用户

9. 时区
   `ln -sf /usr/share/zoneinfo/Asia/Taipei   /etc/localtime`
   硬件时钟 
   `hwclock --systohc utc`

10. Locale
  编辑  /etc/locale.gen
  `en_US.UTF-8 UTF-8`
  `zh_CN.UTF-8 UTF-8`
  `zh_CN.GBK    GBK`
  `zh_CN            GB2312`
  使用命令`locale-gen`生成信息
  编辑 /etc/locale.conf
  `LANG=en_US.UTF-8`

11. 主机名

    ```
    /etc/hostname	arch
    /etc/hosts	127.0.0.1  arch.localdomain  arch
    ```

12. 网络和基本组件
    `pacman -S network-manager-applet wpa_supplicant dialog gnome firefox xfce4-terminal tmux`

13. 生成randisk
    `mkinitcpio   -p linux`

14. 引导器

    ```
    pacman -S grub os-prober efibootmgr
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --recheck
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

15. 开机启动

    ```
    systemctl enable gdm
    systemctl enable NetworkManager
    ```

16. 退出chroot环境

    ```
    umount -R /mnt/boot/efi
    umount -R /mnt/home
    umount -R /mnt
    ```


reboot重启 

至此gnome环境已经有了。

重启登录后

中文字体： `pacman -S wqy-zenhei wqy-microhei`
等款字体：`pacman -S adobe-source-code-pro-fonts`
 思源：       `pacman -S adobe-source-han-sans-otc-fonts`
​                    `pacman -S adobe-source-han-sans-cn-fonts`
中文输入法：`pacman -S --needed fcitx-sunpinyin fcitx-goolgepinyin fcitx-rime fcitx-configtool fcitx-im`

PDF查看： `pacman -S zathura zathura-pdf-poppler`

然后编辑 ~/.xprofile 添加如下
```
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
```

删除一些无用软件：
```
pacman -Rscn epiphany totem empathy gnome-dictionary
```

启用AUR
```
# /etc/pacman.conf 中添加
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
pacman -Syu yaourt
```

如果你有yubikey
`pacman -S libu2f-host`
如果你想使用lightdm

```
pacman -S lightdm lightdm-gtk-greter
systemctl enable lightdm
```

启动器		rofi
截图工具		scrot
文件管理		thunar
压缩管理		xarchiver
终端查看图片 feh
系统信息输出  screenfetch

如果你需要使用 $\TeX$ 文档

```
# pacman -S texlive-most
# yaourt -S acroread-fonts-systemwide
```



推荐使用i3管理器

目前安装的包 [详细列表](https://gist.github.com/snowfrs/a90abd855551d1b5a93e68a668f7a7db)
`pacman -Qe`



