---
title: AD Attribute Editor
tags: AD 389ds
---
<!--more-->

# 如何在AD中填写unix属性并同步到389ds
需要在AD账户标签 **`Attribute Editor`** 中填写以下属性
```txt
uid: ithelpdesk
uidNumber: 99999 (magic number 99999, 389ds会自动创建uidNumber; 需要预先在389ds配置magic number)
unixHomeDirectory: /home/ithelpdesk
gidNumber: 100
loginShell: /bin/tcsh
objectClass: top;person;organizationalPerson;user;inetOrgPerson;**`posixAccount`** (最重要，手动添加)
c:CN
l:shanghai
st:SH
cn: ithelpdesk
o:China
```