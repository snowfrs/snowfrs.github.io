---
title: Weechat
tags:
---

**啊!!! 古老的IRC**

<!--more-->

# 安裝

```
# archlinux
$sudo pacman -S weechat
```

# 启动

```
weechat
```

# 配置

```
# 添加server
/server add freenode chat.freenode.net
# 帮助命令help
/help server
```

## 自定义配置

命令格式

```
/set config.section.option value
```

```
/set irc.server.freenode.nicks "mynick,mynick1,mynick2"
/set irc.server.freenode.username "username"
/set irc.server.freenode.realname "realname"
/set irc.server.freenode.autoconnect on
/set irc.server.freenode.autojoin "#channel1,#channel2"
```

连接/加入/离开/关闭/断开

```
# 连接server
/connect freenode
# 加入频道
/join #channel
# 离开频道
/part [quit message]
# 关闭server,channel,private buff
/close
# disconnect form server, on the server buffer
/disconnect
```

注册

```
/msg NickServ REGISTER <password> <email@example.com>
```

然后到邮箱查看，执行正文中一条验证邮箱的命令就完成注册了

重置密码

```
/msg NickServ SENDPASS youraccountnamehere
```

按照邮箱指令输入命令即可重置

设置自动验证身份

```
/set irc.server.freenode.command "/msg NickServ identify yourpasswd"
```



