---
title: Windows error code 0xc0000098 when start you PC
tags:
  - BCD
  - Windows
date: 2016-11-26 18:45:06
permalink: windows-error-code-0xc0000098
---

1. Find a Windows System installation disk, start it when you power on your PC;


2. Choose repair computer, then use command line

   cd boot
   attrib bcd -s -h -r
   ren
   c:\boot\bcd bcd.old
   bootrec /rebuildbcd

3. reboot