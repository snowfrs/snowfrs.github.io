---
title: 初始化 Ruby on Rails
tags: ruby rails
---

安装 Ruby 一般有三种方法:
- rbenv
- RVM or Ruby Version Manager
- Source
<!--more-->

这里使用 rbenv 来管理 Rbuy 版本. 
## rbevn 环境配置
```bash
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
# set up rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ exec $SHELL
# ruby-build plugin
$ git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
```

## 安装 Ruby
```bash
$ rbenv install 2.6.6
$ rbenv global 2.6.6
```
输出内容一般为:
```bash
Downloading ruby-$VERSION.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/x.y/ruby-$VERSION.tar.bz2
Installing ruby-$VERSION...
Installed ruby-$VERSION to /home/<user_name>/.rbenv/versions/$VERSION
```
如果中间提示失败, 请按提示安装依赖
```bash
# Ubuntu
$ sudo apt install -y libreadline-dev zlib1g-dev
```

检查 Ruby 版本
```bash
$ ruby -v
```

### 安装 bundler
```bash
$ gem install bundler
```
安装 bundler后需要执行
```bash
$ rbenv rehash
```

## 安装 Rails
```bash
$ gem install rails -v 6.0.3.3
$ rbenv rehash
```
检查 Rails 版本
```bash
$ rails -v
```