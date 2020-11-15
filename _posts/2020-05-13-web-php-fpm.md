---
title: PHP-FPM
tags: Web PHP-FPM
---
<!--more-->

# 介绍
PHP-FPM 用于解析 PHP 并通过 Nginx 向 Web 提供 fastcgi.
# Installation
一般选择 ![Remi][remi]提供的repo
切换php安装源
```bash
dnf module list php 
dnf module reset php 
dnf module enable php:7.x
```
# Configuration
一般在 `/etc/php.ini` 和 `/etc/php-fpm/xxx.conf`
# 典型配置
`/etc/php.ini`
- upload_max_filesize
- max_post_size
- date.timezone
- short_open_tag 一般设置为Off
- expose_php 设置为Off 避免php版本信息泄漏
- memory_limit
- max_input_vars

`/etc/php-fpm/xxx.conf`
- 可以设置多个 pool
- pm 选项有 `static` `dynamic` `ondemand`
- pm.max_children
- pm.start_servers, pm.min_spare_servers, pm.max_spare_servers
- pm.max_requests

# OpCode Cache
默认已启用 配置文件 `/etc/php.d/10-opcache.ini` 可以适当调整

- opcache.memory_consumption
- opcache.max_accelerated_files
- opcache.max_wasted_percentage

[remi]: https://blog.remirepo.net/pages/Config-en

