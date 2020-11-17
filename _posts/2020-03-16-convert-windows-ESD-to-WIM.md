---
title: convert Windows ESD file to WIM file
tags: esd wim
---
<!--more-->

新版win10 image中已经使用ESD压缩方式而不是以前的WIM，新的压缩方式效率更高。

对于使用WDS/MDT/SCCM工具部署windows的，需要将ESD文件转换为wim文件才能使用。



使用DISM命令转换
```powershell
C:\WINDOWS\system32>dism /Get-WimInfo /WimFile:"D:\Temp\win10\sources\install.esd"
 
C:\WINDOWS\system32>dism /export-image /SourceImageFile:"D:\Temp\win10\sources\install.esd" /SourceIndex:6 /DestinationImageFile:install.wim /Compress:max /CheckIntegrity

```
转换后的install.vim 文件在 
```powershell
C:\WINDOWS\system32 
```