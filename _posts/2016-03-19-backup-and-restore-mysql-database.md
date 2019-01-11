---
title: Backup and Restore MySQL Database
tags:
  - MySQL
date: 2016-03-19 11:24:23
permalink: Backup-and-Restore-MYSQL-Database-using-mysqldump
---

简单记录

<!--more-->

**备份MySQL数据库并恢复**

备份的方法很多, 官方文档 http://dev.mysql.com/doc/

本文主要讲如何使用mysqldump备份并恢复MySQL. mysqldump是一个高效的备份MySQL数据库的工具.它创建一个*.sql文件,该文件带有原数据库的DROP table, CREATE table 和INSERT into sql-statements. 使用mysqldump,你可以用一条命令备份本地数据库并且同时恢复到远端服务器.

基本用法:
backup
\#mysqldump -u root -p[password][database_name] &gt; dumpfilename.sql

restore
\# mysqldump -u root -p[password][database_name] &lt; dumpfilename.sql
注意 -u 后面有空格 -p没有空格直接跟密码 restore数据库时 目标数据库必须存在 也就是先要创建一个数据库 才能做restore的动作

下面开始举例子:

##### 1.备份单个数据库

**mysqldump -u root -p[password] store &gt; store.sql**

将数据库sotre备份至当前目录并且命名为store.sql. store.sql将包含store的drop table, create table and insert命令的所有tables.

2.备份多个数据库
如果你想备份多个数据库,首先请指定你想备份的数据库的名字

**mysql -u root -p[password] 进入数据库**

mysql&gt; show database;
+--------------------+
| Database &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |
+--------------------+
| information_schema |
| bugs &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |
| mysql &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|
| store &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|
+--------------------+
&nbsp;4 rows in set (0.00 sec)

你想备份数据库store和bugs, 执行命令

**mysqldump -u root -p[password] --databases bugs store &gt; bugs_store.sql**

验证bugs_store.sql包含两个数据库

**grep -i "Current database:" /tmp/bus_store.sql**

-- Current Database: `mysql`
-- Current Database: `store`

##### 3.备份所有数据库

**mysqldump -u root -p[password] --all-databases &gt; /tmp/all-databases.sql**

##### 4.备份指定table

备份数据库store中的accounts_contacts到目录/tmp/store_accounts_contacts.sql

**mysqldump -u root -p[password] store accounts_contacts \**

&nbsp; &nbsp; &nbsp; &gt; /tmp/store_accounts_contacts.sql

#### 如何恢复数据库

##### 1.恢复数据库store.sql到数据库store

首先,数据库store必须存在,创建数据库store如下

mysql -u root -p[password]

mysql&gt; create database store;
mysql&gt; quit;
恢复数据库命令

**mysql -u root -p[password] store &lt; /tmp/store.sql**

##### 2.备份本地数据库的同时恢复数据库到远端服务器

备份本地数据库store &nbsp;恢复store到远端服务器并且重命名为store1
请注意必须先在远端服务器创建数据库store1

**mysqldump -u root -p[password] store | mysql -u root -p[password] --host=remote-server -C store1**

好了,问题来了:
备份多个数据库到一个文件bugs_store.sql后, 如何只恢复其中一个数据库store呢?

操作过程中出现的问题:

备份后 &nbsp;在远端恢复数据之后 打开网站 &nbsp;显示报错
Call to undefined function: mysqli_connect(). Please install the MySQL Connector for PHP

于是打开info.php页面, 发现Additional.ini files parsed一项比原来少了很多, 一项一项安装添加比较 &nbsp;始终还是报同样的错误
重新配置php5-mysql好了,执行命令
apt-get purge php5-mysql
apt-get autoremove
apt-get install php5-mysql
输出以下信息
Selecting previously unselected package php5-mysql.
(Reading database ... 331424 files and directories currently installed.)
Preparing to unpack .../php5-mysql_5.6.14+dfsg-0+deb8u1_amd64.deb ...
Unpacking php5-mysql (5.6.14+dfsg-0+deb8u1) ...
Processing triggers for php5-fpm (5.6.14+dfsg-0+deb8u1) ...
Setting up php5-mysql (5.6.14+dfsg-0+deb8u1) ...

Creating config file /etc/php5/mods-available/mysql.ini with new version
php5_invoke: Enable module mysql for fpm SAPI
php5_invoke: Enable module mysql for cli SAPI

Creating config file /etc/php5/mods-available/mysqli.ini with new version
php5_invoke: Enable module mysqli for fpm SAPI
php5_invoke: Enable module mysqli for cli SAPI

Creating config file /etc/php5/mods-available/pdo_mysql.ini with new version
php5_invoke: Enable module pdo_mysql for fpm SAPI
php5_invoke: Enable module pdo_mysql for cli SAPI
Processing triggers for php5-fpm (5.6.14+dfsg-0+deb8u1) ...
重新配置完毕, 打开info.php页面,发现好多配置瞬间有了. 再次访问server_ip &nbsp;正常访问. 问题解决.