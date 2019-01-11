---
title: LEMP配置-Debian Jessie
tags:
  - Linux
  - Nginx
  - PHP
date: 2016-03-19 08:48:56
permalink: LEMP-Debian-Jessie
---

LEMP配置手册
Linux + Nginx + MySQL + PHP-FPM 搭建网站平台

<!--more-->

1.更新系统
apt-get update
apt-get upgrade

2.安装nginx
apt-get install nginx
systemctl start nginx
开启nginx服务
systemctl enable nginx
设置开机启动nginx

可能出现的问题:
dpkg: error processing package nginx (--configure):
dependency problems - leaving unconfigured
Processing triggers for systemd (215-17+deb8u1) ...
Errors were encountered while processing:
nginx-full
nginx
E: Sub-process /usr/bin/dpkg returned an error code (1)

解决办法:
编辑 /etc/nginx/sites-available/default 注释掉行 &nbsp;listen [::]:80 default_server;
systemctl restart nginx
apt-get install nginx	目的是配置Nginx

此时访问 server_ip &nbsp;出现welcom表示配置成功.
Welcome to nginx on Debian!
If you see this page, the nginx web server is successfully installed and working on Debian. Further configuration is required.
For online documentation and support please refer to nginx.org
Please use the reportbug tool to report bugs in the nginx package with Debian. However, check existing bug reports before reporting a new bug.
Thank you for using debian and nginx.

3.安装MySQL
apt-get install mysql-server
安装过程中请根据提示设置root密码
安装完成后, 请执行安全安装选项
mysql_secure_installation
输入刚才设置的SQL root密码, 当被问到是否修改root密码时, 请输入n.
过程如下:
Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
... skipping.

By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them. &nbsp;This is intended only for testing, and to make the installation
go a bit smoother. &nbsp;You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
... Success!

Normally, root should only be allowed to connect from 'localhost'. &nbsp;This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
... Success!

By default, MySQL comes with a database named 'test' that anyone can
access. &nbsp;This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
- Dropping test database...
  ERROR 1008 (HY000) at line 1: Can't drop database 'test'; database doesn't exist
  ... Failed! &nbsp;Not critical, keep moving...
- Removing privileges on test database...
  ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
... Success!

Cleaning up...

All done! &nbsp;If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!

设置开机启动mysql
systemctl enable mysql

安装过程可能会报错:
Job for mysql.service failed. See 'systemctl status mysql.service' and 'journalctl -xn' for details.
invoke-rc.d: initscript mysql, action "start" failed.
dpkg: error processing package mysql-server-5.5 (--configure):
&nbsp;subprocess installed post-installation script returned error exit status 1
dpkg: dependency problems prevent configuration of mysql-server:
&nbsp;mysql-server depends on mysql-server-5.5; however:
&nbsp; Package mysql-server-5.5 is not configured yet.

dpkg: error processing package mysql-server (--configure):
&nbsp;dependency problems - leaving unconfigured
Processing triggers for libc-bin (2.19-18+deb8u3) ...
Errors were encountered while processing:
&nbsp;mysql-server-5.5
&nbsp;mysql-server
E: Sub-process /usr/bin/dpkg returned an error code (1)
root@kali:/var/www/html# systemctl status mysql.service
● mysql.service - LSB: Start and stop the mysql database server daemon
&nbsp; &nbsp;Loaded: loaded (/etc/init.d/mysql)
&nbsp; &nbsp;Active: failed (Result: exit-code) since Fri 2016-03-18 16:53:04 CST; 30s ago
&nbsp; Process: 11831 ExecStart=/etc/init.d/mysql start (code=exited, status=127)

Mar 18 16:53:02 kali mysql[11831]: /etc/init.d/mysql: WARNING: /etc/mysql/my.cnf cannot be read. See README.Debian.gz ... (warning).
Mar 18 16:53:04 kali mysql[11831]: Starting MySQL database server: mysqld ..
Mar 18 16:53:04 kali mysql[11831]: /etc/init.d/mysql: line 121: /etc/mysql/debian-start: No such file or directory
Mar 18 16:53:04 kali systemd[1]: mysql.service: control process exited, code=exited status=127
Mar 18 16:53:04 kali systemd[1]: Failed to start LSB: Start and stop the mysql database server daemon.
Mar 18 16:53:04 kali systemd[1]: Unit mysql.service entered failed state.

解决办法:
apt-get purge mysql-server
apt-get autoremove
apt-get clean
apt-get autoclean
apt-get update
apt-get install mysql-server

4.安装PHP-FPM
apt-get install php5-fpm php5-mysql

下一步要做的就是修改nginx配置文件, 修改之前请先备份.
mv /etc/nginx/sites-available/default	/etc/nginx/sites-available/default.old
vim /etc/nginx/sites-available/default
基本配置如下:
server {
&nbsp; &nbsp; &nbsp; &nbsp; listen &nbsp; &nbsp; &nbsp; 80;
&nbsp; &nbsp; &nbsp; &nbsp; server_name &nbsp;your_website_name.com;
&nbsp; &nbsp; &nbsp; &nbsp; root /var/www/html;
&nbsp; &nbsp; &nbsp; &nbsp; index index.php index.html index.htm index.nginx-debian.html;
&nbsp; &nbsp; &nbsp; &nbsp; location / {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; try_files $uri $uri/ =404;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; error_page 404 /404.html;
&nbsp; &nbsp; &nbsp; &nbsp; error_page 500 502 503 504 /50x.html;
&nbsp; &nbsp; &nbsp; &nbsp; location = /50x.html {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; root /var/www/html;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; location ~ \.php$ {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; try_files $uri =404;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; fastcgi_pass unix:/var/run/php5-fpm.sock;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; fastcgi_index index.php;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; include fastcgi_params;
&nbsp; &nbsp; &nbsp; &nbsp; }
}
保存退出.

5.测试
创建一个文件名为info.php的文件 &nbsp;放在目录/var/www/html下
vim /var/www/html/info.php
文件内容如下:
&lt;?php
phpinfo();
?&gt;

重启nginx服务以验证.
systemctl restart nginx
访问页面server_ip/info.php &nbsp;你将会看到PHP版本, 包含的modules 等等.

LEMP配置完成!
Thanks!