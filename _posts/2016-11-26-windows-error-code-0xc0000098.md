---
title: Windows error code 0xc0000098
tags:
  - BCD
  - Windows
---
<!--more-->
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
