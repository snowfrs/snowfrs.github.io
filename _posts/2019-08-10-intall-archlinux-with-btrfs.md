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
- /dev/nvme0n1p3	btrfs分区，用来安装新的系统

EFI分区已经有数据，不再格式化。

格式化
```
mkfs.btrfs -m single -L arch /dev/nvme0n1p3
```
挂载
```
mount -o compress=lzo /dev/nvme0n1p3 /mnt
```
创建subvolume
```
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@logs
btrfs subvolume create /mnt/@tmp
btrfs subvolume create /mnt/@docker
btrfs subvolume create /mnt/@pkgs
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@build
```
subvolume @ 用来做新系统的根(/)

layout

```
ID 256 gen 405 parent 5 top level 5 path @
ID 257 gen 409 parent 5 top level 5 path @home
ID 258 gen 409 parent 5 top level 5 path @logs
ID 259 gen 404 parent 5 top level 5 path @tmp
ID 260 gen 257 parent 5 top level 5 path @docker
ID 261 gen 310 parent 5 top level 5 path @pkgs
ID 262 gen 288 parent 5 top level 5 path @snapshots
ID 263 gen 20 parent 5 top level 5 path @build
```

挂载subvolume
```
umount /mnt
mount -o noatime,nodiratime,compress=lzo,subvol=@ /dev/nvme0n1p3 /mnt
mkdir -p /mnt/{btrfs-root,boot/efi,home,var/{log,lib/{docker,build},cache/pacman},tmp,.snapshots}
mount -o noatime,nodiratime,compress=lzo,subvol=@home /dev/nvme0n1p3 /mnt/home
mount -o noatime,nodiratime,compress=lzo,subvol=@logs /dev/nvme0n1p3 /mnt/var/log
mount -o noatime,nodiratime,compress=lzo,subvol=@tmp /dev/nvme0n1p3 /mnt/tmp
mount -o noatime,nodiratime,compress=lzo,subvol=@docker /dev/nvme0n1p3 /mnt/var/lib/docker
mount -o noatime,nodiratime,compress=lzo,subvol=@pkgs /dev/nvme0n1p3 /mnt/var/cache/pacman
mount -o noatime,nodiratime,compress=lzo,subvol=@snapshots /dev/nvme0n1p3 /mnt/.snapshots
mount -o noatime,nodiratime,compress=lzo,subvol=@build /dev/nvme0n1p3 /mnt/var/lib/build

mount -o noatime,nodiratime,compress=lzo,subvol=/ /dev/nvme0n1p3 /mnt/btrfs-root
```
此时不要忘记将efi分区挂载上
```
mount /dev/nvme0n1p1 /mnt/boot/efi
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
useradd -m -u UID -G wheel -s /bin/bash USERNAME
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
最终mount 效果

```
/dev/nvme0n1p3 on / type btrfs (rw,noatime,nodiratime,compress=lzo,ssd,space_cache,subvolid=256,subvol=/@) [btrfs-arch]
/dev/nvme0n1p3 on /tmp type btrfs (rw,noatime,nodiratime,compress=lzo,ssd,space_cache,subvolid=259,subvol=/@tmp) [btrfs-arch]
/dev/nvme0n1p3 on /var/lib/build type btrfs (rw,noatime,nodiratime,compress=lzo,ssd,space_cache,subvolid=263,subvol=/@build) [btrfs-arch]
/dev/nvme0n1p3 on /var/lib/docker type btrfs (rw,noatime,nodiratime,compress=lzo,ssd,space_cache,subvolid=260,subvol=/@docker) [btrfs-arch]
/dev/nvme0n1p3 on /home type btrfs (rw,noatime,nodiratime,compress=lzo,ssd,space_cache,subvolid=257,subvol=/@home) [btrfs-arch]
/dev/nvme0n1p3 on /.snapshots type btrfs (rw,noatime,nodiratime,compress=lzo,ssd,space_cache,subvolid=262,subvol=/@snapshots) [btrfs-arch]
/dev/nvme0n1p3 on /var/log type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=258,subvol=/@logs) [btrfs-arch]
/dev/nvme0n1p3 on /var/cache/pacman type btrfs (rw,noatime,nodiratime,compress=lzo,ssd,space_cache,subvolid=261,subvol=/@pkgs) [btrfs-arch]

/dev/nvme0n1p3 on /btrfs-root type btrfs (rw,noatime,nodiratime,compress=lzo,ssd,space_cache,subvolid=5,subvol=/) [btrfs-arch]
```



[install-arch]: https://snowfrs.com/2017/01/01/archlinux-for-new-user.html