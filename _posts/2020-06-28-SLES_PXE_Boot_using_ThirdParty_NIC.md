---
title: SLES PXE Boot using ThirdParty NIC
tags: PXE SLES
---
<!--more-->

# 使用第三方网卡通过PXE网络安装SUSE
## SUSE 安装流程
正常情况下SUSE安装流程是这样的:

+ 加载 kernel 和 initrd
+ kernel 扫描硬件并加载相关 modules
+ udev 启动
+ linuxrc 启动并做一些附加的硬件检测
+ yast2 启动并开始安装

使用第三方网卡的情况下, 网卡必须在最开始就能工作。我们需要 DUD (Driver Update Disk). DUD 可以在启动和安装过程中为某些需要的硬件加载驱动.

## Update Media 种类
+ **Driver Update**: 通过 Driver Update, 可以在创建发行版时不支持的硬件上安装 SUSE. 并且以后可以引导已安装的操作系统, 无需在安装系统后手动安装设备驱动程序.
+ **Vendor Update**: 使用 YaST2 安装不包含在发行版中的软件或者设备驱动. 可以在安装结束时或者使用 YaST Vendor CD 模块在运行的系统中随时进行此操作.
+ **YaST Update**: 可以在安装之前更新 YaST2 程序的某些部分.

PXE网络安装的情况下, 显然采用第一种方法是妥当的.

此时安装流程为:

+ 加载 kernel 和 initrd
+ kernel 扫描硬件并加载相关 modules
+ udev 启动
+ linuxrc 启动并做一些附加的硬件检测
+ linuxrc 加载 DUD 并检测相关 modules
+ yast2 启动并开始安装

## 制作DU
需要 **SUSEDriverTools** (sdt)
```bash
sdt dud create -u /root/update/ -b /media/ ./mydud
```
`sdt dud create` 用来创建 DUD,

`--updatedir` 指定需要扫描并加入到driver kit的目录,

`--base BASE_MEDIA` 一般将发行版iso挂载到 /media 使用

生成的文件放到 `./mydud` 目录下

更多信息请参考 **man sdt-dud**

```bash
# tree ./mydud
mydud
└── linux
    └── suse
        └── x86_64-sles12
            ├── dud.config
            ├── install
            │   ├── rpms
            │   │   ├── ngbe-1.0.3-1.x86_64.rpm
            │   │   └── ngbe-kmp-default-1.0.3_k4.12.14_120-1.x86_64.rpm
            │   ├── update.post2
            │   ├── update.post2.d
            │   │   └── remove_dud_repo
            │   ├── update.pre
            │   └── update.pre.d
            │       └── create_add_on_products
            └── modules
                └── ngbe.ko
```

## 制作initrd
```bash
sdt initrd update -u ./mydud/ -b /media/ -m network:ngbe,INITRD -m network:txgbe,INITRD newinitrd
```
更多信息请参考 **man sdt-initrd**

需要特别注意 `--modspec` 选项

-m MODSPEC, –modspec MODSPEC

              Additional module to add to module.config such that it’s added to initrd and linuxrc considers it while probing hardware.  MODSPEC is of the following format:

              **SECTION:MODULE,DESCR,PARAM,PRE_INST,POST_INST,INITRD,AUTO**

              If the kernel doesn’t contain any new driver which should be loaded for installation compared to the kernel contained in the base ISO, this option isn’t necessary.

SECTION 可选的有

- SUSE Driver Tools:INFO: module.config: processing section: autoload
- SUSE Driver Tools:INFO: module.config: processing section: IDE/RAID/SCSI
- SUSE Driver Tools:INFO: module.config: processing section: network
- SUSE Driver Tools:INFO: module.config: processing section: PCMCIA
- SUSE Driver Tools:INFO: module.config: processing section: WLAN
- SUSE Driver Tools:INFO: module.config: processing section: USB
- SUSE Driver Tools:INFO: module.config: processing section: FireWire
- SUSE Driver Tools:INFO: module.config: processing section: file system
- SUSE Driver Tools:INFO: module.config: processing section: acpi
- SUSE Driver Tools:INFO: module.config: processing section: other

将newinitrd上传到PXE服务器

## 其他
此时比较 发行版自带的 initrd和新制作的 newinitrd
```bash
sdt initrd compare /media/boot/x86_64/loader/initrd newinitrd
```
会看到
```txt
------
- /media/boot/x86_64/loader/initrd://lib/modules/<kernel>lib/modules/initrd/ngbe.ko: NOT FOUND
+                      newinitrd.5://lib/modules/<kernel>lib/modules/initrd/ngbe.ko: MODE: N/A UID: None GID: None hash: N/A
------
- /media/boot/x86_64/loader/initrd://linux/suse/x86_64-sles12/install/rpms/ngbe-1.0.3-1.x86_64.rpm: NOT FOUND
+                      newinitrd.5://linux/suse/x86_64-sles12/install/rpms/ngbe-1.0.3-1.x86_64.rpm: MODE: -rw-r--r-- UID: 0 GID: 0 hash: 3fa556f992dcb44cee9e9d99ea993523b619915f
------
- /media/boot/x86_64/loader/initrd://linux/suse/x86_64-sles12/install/rpms/ngbe-kmp-default-1.0.3_k4.12.14_120-1.x86_64.rpm: NOT FOUND
+                      newinitrd.5://linux/suse/x86_64-sles12/install/rpms/ngbe-kmp-default-1.0.3_k4.12.14_120-1.x86_64.rpm: MODE: -rw-r--r-- UID: 0 GID: 0 hash: f2f10db6c3e78d7949d997c1ba6553977043c71b 
------
```
解压newinitrd 到一个空目录 test
```bash
sdt initrd unpack newinitrd test
```
```bash
# tree test -L 1
test
├── bin
├── dev
├── download
├── etc
├── init
├── lib
├── lib64
├── linux
├── linuxrc.config
├── mnt
├── modules -> lib/modules/4.12.14-120-default/initrd
├── mounts
├── parts
├── proc
├── root
├── run
├── sbin
├── scripts
├── sys
├── tmp
├── usr
└── var
```
注意看 `test/linux/suse/x86_64-sles12/install/` 目录下的某些脚本 
`update.pre.d` 脚本会创建repo源用来安装DUD里面的rpm包
`update.post2.d` 脚本会删除DUD里的安装源

在test/linux/suse/x86_64-sles12/install

整个过程大概是这样的:

PXE网络启动时获取kernel和initrd(修改过包含第三方网卡驱动 name.ko ), linuxrc检测DUD并加载网卡驱动 name.ko, yast2启动并安装操作系统，安装DUD源里的rpm包, 安装完成.

安装DUD源里的rpm包时，并非所有 **test/linux/suse/x86_64-sles12/install/rpms** 下存在的rpm包都会被安装，只有被kernel和linuxrc检测到的硬件对应的rpm包会被安装.


Reference:

http://ftp.suse.com/pub/people/hvogel/Update-Media-HOWTO