---
title: Create a custom live iso
tags: Live-ISO
---
<!--more-->

本文使用 [deepin](http://deepin.org/) 20 来创建一个自定义 deepin iso.

你可以使用任意带有 debootstrap 环境的系统从0开始构建一个自定义live iso，但本文只介绍在已有iso的基础上修改并生成自定义live iso.

本文说的修改主要是指新添加一个第三方驱动包.


**警告**：所有需要使用 **`[chroot]`** 环境的命令 我都会使用加粗字体提前告知.

```bash
sudo mount -t iso9660 deepin.iso /mnt
```
创建工作环境
```bash
mkdir -p $HOME/deepin-live/workdir
```
复制 filesystem.squashfs到本地
```bash
cp -av /mnt/live/filesystem.squashfs $HOME/deepin-live/
```
解压 squashfs
```bash
cd $HOME/deepin-live/workdir

unsquashfs ../filesystem.squashfs
```
会自动解压到当前路径 目录名称 `squashfs-root`

为chroot做必要的操作
```bash
sudo mount --bind /dev squashfs-root/dev

sudo mount --bind /sys squashfs-root/sys

sudo mount --bind /proc squashfs-root/proc
```
准备第三方驱动包
```bash
cp -av package-name.deb $HOME/deepin-live/workdir/squashfs-root/
```

**`警告： [chroot]`**
```bash
chroot $HOME/deepin-live/workdir/squashfs-root/
```
目前已在chroot环境 以下开始chroot环境的操作
```bash
dpkg -i package-name.deb
```
末尾追加一行
```bash
sed -i '$a\driver_name' /etc/initramfs-tools/modules
```
更新initramfs
```bash
update-initramfs -u
```
创建 manifest
```bash
dpkg-query --show --showformat='${Package} ${Version}\n' | tee filesystem.manifest
```

清理战场
```bash
apt clean

history -c

rm -rf /tmp/*

rm -rf package-name.deb
```
退出 `chroot` 环境
```bash
exit 

umount -R $HOME/deepin-live/workdir/squashfs-root
```
生成 squashfs
```bash
# pwd

/root/deepin-live/workdir

# mksquashfs squashfs-root filesystem.squashfs.new -comp xz
```

准备新iso内容
```bash
mkdir -p $HOME/custom-iso

rsync --exclude=live/filesystem.squashfs -az /mnt/ custom-iso/

cd $HOME/custom-iso/live
```

更新 squashfs
```bash
cp -a $HOME/deepin-live/workdir/filesystem.squashfs.new  filesystem.squashfs
```
更新 manifest
```bash
cp -a $HOME/deepin-live/workdir/squashfs-root/filesystem.manifest filesystem.manifest

cp -a $HOME/deepin-live/workdir/squashfs-root/filesystem.manifest filesystem.manifest-desktop
```

更新 filesystem.size
```bash
du -sx --block-size=1 $HOME/deepin-live/workdir/squashfs-root | cut -f1 > filesystem.size
```

更新 initrd.lz (包含第三方驱动)
```bash
cp -a $HOME/deepin-live/workdir/squashfs-root/boot/initrd-XXX.img initrd.lz
```

重新计算 md5
```bash
cd $HOME/custom-iso

rm md5sum.txt

find -type f -print0 | xargs -0 md5sum | tee md5sum.txt
```

生成iso
```bash
# pwd

/root/custom-iso
# xorriso -as mkisofs -V 'Deepin Livecd' -cache-inodes -J -l -joliet-long -b isolinux/isolinux.bin -c isolinux/boot.cat -boot-load-size 4 -boot-info-table -no-emul-boot -eltorito-alt-boot -e boot/efi.img -no-emul-boot -isohybrid-gpt-basdat -isohybrid-apm-hfsplus  -o ../uos20sp1-custom-amd64.iso .
```

也可使用 `genisoimage` 生成新的iso image

参数说明

**`xorriso -as mkisofs -help --`**

```
-V ID, -volid ID            Set Volume ID

-cache-inodes               Cache inodes (needed to detect hard links)

-J, -joliet                 Generate Joliet directory information

-l, -full-iso9660-filenames Allow full 31 character filenames for ISO9660 names

-joliet-long                Allow Joliet file names to be 103 Unicode characters

-b FILE, -eltorito-boot FILE

                              Set El Torito boot image name

-c FILE, -eltorito-catalog FILE

                              Set El Torito boot catalog name

-eltorito-alt-boot          Start specifying alternative El Torito boot parameters


-boot-load-size #           Set numbers of load sectors

-boot-info-table            Patch boot image with info table

-no-emul-boot               Boot image is 'no emulation' image

-isohybrid-gpt-basdat       Mark El Torito boot image as Basic Data in GPT

-isohybrid-gpt-hfsplus      Mark El Torito boot image as HFS+ in GPT

 -isohybrid-apm-hfsplus      Mark El Torito boot image as HFS+ in APM

-o FILE, -output FILE       Set output file name

-iso-level LEVEL            Set ISO9660 conformance level (1..3) or 4 for ISO9660 version 2

-udf                        Generate UDF file system
```

Reference:

https://willhaley.com/blog/custom-debian-live-environment/

https://github.com/pranav1698/Custom-LiveISO-Generator

https://live-team.pages.debian.net/live-manual/