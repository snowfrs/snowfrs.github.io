---
title: 使用 btrfs 文件系統安裝 archlinux
tags: btrfs archlinux
---
距离上次 [安装archlinux][install-arch] 已经过去很久。最近想尝试下btrfs，于是有了这篇。

<!--more-->

## 备份当前系统数据
备份方式太多了。我只备份了 dotfiles, 当前系统安装的包列表，以及一些重要数据。

## 准备分区
我的磁盘分区是这样的

- /dev/nvme0n1p1	EFI分区
- /dev/nvme0n1p4	btrfs分区，用来安装新的系统

EFI分区已经有数据，不再格式化。

格式化
```
mkfs.btrfs -m single -L arch /dev/nvme0n1p4
```
挂载
```
mount -o compress=lzo /dev/nvme0n1p4 /mnt
```
创建subvolume
```
cd /mnt
btrfs subvolume create @
btrfs subvolume create @home
```
我这里创建了两个subvolume @ 和 @home，你可以根据自己意愿创建更多。

挂载subvolume
```
cd /
umount /mnt
mount -o compress=lzo,subvol=@ /dev/nvme0n1p4 /mnt
cd /mnt
mkdir -p {home,/boot/efi}
mount -o compress=lzo,subvol=@home /dev/nvme0n1p4 home
```
此时不要忘记将efi分区挂载上
```
mount /dev/nvme0n1p1 boot/efi
```

## 安装 base system
首先修改 /etc/pacman.d/mirrorlist 将TUNA源放到最前面
```
pacstrap -i /mnt base base-devel vim snapper
```

## 配置基本项 
```
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime #替换Region/City为你所在区域
hwclock --systohc
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
pacman -S networkmanager
systemctl enable NetworkManager.service
hostnamectl set-hostname YOUR-HOSTNAME
```
## 配置initramfs参数
```
vim /etc/mkinitcpio.conf
添加 btrfs 到 MODULES=(...)行
找到 HOOKS=(...)行，更换fsck为btrfs
最终你看到的/etc/mkinitcpio.conf文件格式为

...
MODULES=(btrfs)
HOOKS=(base udev autodetect modconf block filesystems keyboard btrfs)
...

```

## 生成initramfs
```
mkinitcpio -p linux
```
## 用户，密码，引导项，DM, WM
```
passwd root
useradd -m -u UID -s /bin/bash USERNAME
passwd USERNAME

pacman -S grub os-prober efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch --recheck
grub-mkconfig -o /boot/grub/grub.cfg

pacman -S i3-wm i3status i3blocks i3lock rxvt-unicode dmenu py3status sddm
systemctl enable sddm.service
pacman -S wpa_supplicant dialog
```

## 重启
```
exit #退出chroot
umount /mnt/boot/efi
umount /mnt/home
umount /mnt
reboot
```
## application
自己配置，我一般根据备份的软件列表安装

## AUR, archlinuxcn
```
vim /etc/pacman.conf
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

## snapshot
btrfs有snapshot功能，snapper可以自动帮你自动做snapshot
```
pacman -S snapper
snapper -c root create-config /
snapper -c home create-config /home
# 根据自己的subvolume实际情况创建snapshot策略
snapper list-configs
systemctl enable --now snapper-timeline.timer
systemctl enable --now snapper-cleanup.timer
```

## 如果你要删除subvolume
进入live环境
```
mount /dev/nvme0n1p4 /mnt
cd /mnt
ls
btrfs subvolume delete @home
# 删除subvolume之后记得修改对应的/etc/fstab
```
[install-arch]: https://snowfrs.com/2017/01/01/archlinux-for-new-user.html
