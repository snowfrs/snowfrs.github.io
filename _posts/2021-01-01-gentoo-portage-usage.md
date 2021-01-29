---
title: Gentoo Portage
tags: Gentoo Portage
---

关于Gentoo系统安装 请参考

https://wiki.gentoo.org/wiki/Handbook:AMD64

https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet

<!--more-->

# 常用软件
1. 系统常用工具
```
emerge -a htop atop iotop lsof dstat bind-tools sudo curl wget rsync zsync git python nfs-utils cifs-utils colordiff meld neofetch tmux pcmanfm xfce4-terminal remmina aria2 mlocate pciutils zram-init xrandr
```
2. 编辑器
```
emerge -a vim visual-studio-code pycharm-community
```
3. 输入法
```
emerge -a fcitx5-rime
```
4. 浏览器/办公软件
```
emerge -a firefox chromium libreoffice mupdf
```
5. 截图工具
```
emerge -a flameshot
```
6. 多媒体
```
emerge -a feh  ffmpeg obs-studio mplayer plex-media-server plex-media-player
```
7. 字体
```
emerge -a wqy-bitmapfont
```
8. 自动补全
```
emerge -a bash-completion gentoo-bashcomp zsh zsh-completions zsh-syntax-highlighting gentoo-zsh-completions
```
9. 邮件客户端
```
emerge -a neomutt 
```
10. 轻量级wiki
```
emerge -a zim
```

# Portage 用法总结  
1. 查询软件包

    `emerge --search [PACKAGE]`

2. 更新 Portage 树

    `emerge --sync`

3. 预编译

    `emerge --pretend [PACKAGE]` 用来查看将要安装到系统中的依赖包

4. 只获取源码

    `emerge --fetchonly [PACKAGE]` 下载软件及依赖软件的源码到本地

5. 安装软件的依赖关系

    `emerge --onlydeps [PACKAGE]`

6. 更新系统软件包

    `emerge -avu [PACKAGE]`

7. 更新整个系统

    

8. 卸载软件

    `emerge --deselect [PACKAGE]; emerge -c`

9. 列出安装到系统的文件和目录(软件已安装)

    `equery files [PACKAGE]`


# Package management
1. 同步所有repositories
```
emerge --sync
# OR
eix-sync #(using eix)
# OR
layman -S #(using layman)
```
安装`app-portage/eix`

    `emerge -a app-portage/eix`

安装`app-portage/layman`

    `emerge -a app-portage/layman`


2. 列出包

    `qlist` 命令由 `app-portage/portage-utils` 提供
+ 列出所有已安装的包 带版本号 带使用的overlay

    `qlist -IRv`
+ 使用`eix`列出`@world`集包含的包(带版本号)

    `eix --world | less`

    `eix --color -c --world | less -R`
3. 包安装
以`www-client/firefox`为例
+ 列出将要安装的包但不安装(预安装)

    `emerge -pv www-client/firefox`
+ 安装指定版本的包

    `emerge =www-client/firefox-84.0.2`
+ 安装包但不添加到world文件(一次安装)

    `emerge --oneshot www-client/firefox` (或者)
    
    `emerge -1 www-client/firefox` 
4. 包卸载
+ 首先将包从`world`集去掉

    `emerge --deselect www-client/firefox`
+ 接着预卸载

    `emerge --depclean -vp`
+ 确认无误后

    `emerge --depclean -v` (或者)

    `emerge -cv`
5. 更新系统

    `emerge --update --changed-use --deep --quiet @world` (或者)

    `emerge --ask --verbose --update --deep --changed-use @world`

    当`USE` flags仅仅从repository添加或去掉时 这样做可以避免不必要的rebuild 加上`--quiet`可以更快地执行

6. Troubleshooting
+ 检查并修复缺失的libraries (一般情况下不需要)
    
    `revdep-rebuild -v`
+ 使用`equery`查询已安装的包中哪个包提供了指定的命令
```
    equery b `which vim` 
    # OR
    e-file vim
```
`equery`由 `app-portage/gentoolkit`提供

`app-portage/gentoolkit`也提供 `eclean` `euse` 等

`e-file`由 `app-portage/pfl`提供
+ 使用`equery`查询哪个包依赖指定的包

    `equery d www-client/firefox`
+ 使用`eix`查询关于某个包的信息

    `eix www-client/firefox`
+ eclean distfiles 只删除过期的包
7. 安装/更新后

    `perl-cleaner --all`

# USE flags
+ 使用`euse`获取USE flag `X`的描述和使用情况

    `euse -i X`
+ 检查哪个包有 'mysql' USE flag

    `equery hasuse mysql`
+ 查询已安装的包中在构建是使用了 `mysql` USE flag

    `eix --installed-with-use mysql`
+ 查询指定包的可用 USE flag

    `equery uses [PACKAGE]`



# 升级 Gentoo 系统

# 升级 Gentoo Kernel

# 重要的 Portage 文件
+ `/etc/portage/make.conf` - 全局设定(USE flags, compiler options)
+ `/etc/portage/package.use` - 单个包的 USE flags, 也可以是包含多个文件的目录
+ `/etc/portage/package.accept_keywords` - 单个包的关键字 `~amd64` `~x86` `~arm`
+ `/etc/portage/package.license` - 接受的licenses
+ `/var/lib/portage/world` - 明确安装的软件包原子列表
+ `/var/db/pkg` - 包含每个已安装包的信息 以及有关安装的一组文件

# Portage日志管理
`app-portage/genlop` 是Portage日志处理程序，还可以估算出emerge软件包的构建时间
+ 查看最后10个emerges(installs)

    `genlop -l | tail -n 10`
+ 查看emerge firefox花费的时间(firefox已安装)

    `genlop -t firefox`
+ 估算命令 `emerge -uND --with-bdeps=y @world` 将花费的时间

    `emerge -pU @world | genlop --pretend`
+ Watch the latest merging ebuild during system upgrade

    `watch genlop -unc`

# Overlays
需要安装 `app-eselect/eselect-repository`
+ 列出所有存在的overlays

    `eselect repository list`
+ 列出所有安装(启用)的overlays

    `eselect repository list -i`

# Layman
需要安装 `app-portage/layman`
+ 列出所有存在的overlays

    `layman -L`
+ 列出所有安装(启用)的overlays

    `layman -l`

# Yubikey
`emerge -av app-crypt/libu2f-host sys-auth/pam_u2f app-crypt/yubikey-manager-qt`