---
title: Windows Boot Error 
tags:
  - BCD
  - Windows
---
<!--more-->
Win10/UEFI 引导启动时, EFI 信息存放在 `EFI\Microsoft\Boot`
1. 使用win10系统安装盘进入
2. 选择 **Repair your computer** 而不是 **Install now** 然后选择 **Troubleshoot>Advanced  Options>Startup Repair** 按照提示选择你要修复的OS
3. 如果上一步不能修复 则进入命令行模式 **Troubleshoot>Advanced Options> Command Prompt**
4. 进入命令行模式后 首先需要找到磁盘和分区 (diskpart工具)
```
X:\Windows\System32> diskpart
list disk
select disk 0 #(选择你的磁盘序号)
list partition
select partition 1 #(选择对应EFI分区的分区号 一般标识为system分区)
list vol #(查看letter标识 )
assign letter=F #(将刚才选中的EFI分区分配一个盘符F)
```
5. `Shift+F10`重新打开一个命令行窗口 直接清除EFI中关于boot的部分 然后重新生成
```
cd /d F:\EFI\Microsoft
ren Boot Boot.old
BOOTREC /RebuildBcd
BCDBOOT C:\Windows /l en-us /s F: /f ALL
```
6. 此时可以查看BCD信息
```
bcdedit /store F:\EFI\Microsoft\Boot\BCD /enum
```

https://answers.microsoft.com/en-us/windows/forum/all/restoring-windows-efi-files-without-wiping-linux/a2e46763-dac0-4541-bbc8-af0085cb67a9

如果你需要修改BCD信息
https://docs.microsoft.com/en-us/azure/virtual-machines/troubleshooting/boot-error-status-not-found#add-the-osdevice-variable

以下方法适合win7/MBR方式
1. Find a Windows System installation disk, start it when you power on your PC;
2. Choose repair computer, then use command line
```
   cd boot
   attrib bcd -s -h -r
   ren
   c:\boot\bcd bcd.old
   bootrec /rebuildbcd
```
3. reboot
