---
title: Ansible Inventory
tags: ansible
---
<!--more-->

# 介绍

批量管理主机的时候, 需要预先定义主机或者组, 这个文件叫 `Inventory`, 默认位于 `/etc/ansible/hosts`.

/etc/ansible/hosts 文件的格式如下:
```bash
mail.example.com

[web]
web[1:5].example.com

[db]
db[1:3].example.com

[prod:children]
web
db
```
+ 指定主机 mail.example.com
+ 指定主机组 web, db
+ 主机组嵌套 prod

**未被分组的主机默认在ungrouped组内**

# 选择主机和组

举例主机清单
```bash
srv1.example.com
srv2.example.com
s1.lab.example.com
s2.lab.example.com

[web]
jupiter.lab.example.com
saturn.example.com

[db]
db1.example.com
db2.example.com
db3.example.com
#db[1:3].example.com

[lb]
lb1.lab.example.com
lb2.lab.example.com

[boston]
db1.example.com
jupiter.lab.example.com
lb2.lab.example.com

[london]
db2.example.com
db3.example.com
file1.lab.example.com
lb1.lab.example.com

[dev]
web1.lab.example.com
db3.example.com

[stage]
file2.example.com
db2.example.com

[prod]
lb2.lab.example.com
db1.example.com
jupiter.lab.example.com

[function:children]
web
db
lb
city

[city:children]
boston
london
environments

[environments:children]
dev
stage
prod
new

[new]
172.25.252.23
172.25.252.44
```
## 匹配所有主机
```bash
ansible all --list-hosts
```
## 匹配指定的主机或主机组
### 匹配单个主机
```bash
ansible db1.example.com --list-hosts
```
### 匹配多个主机
```bash
ansible 'db1.example.com,s1.lab.example.com' --list-hosts
```
### 匹配单个组
```bash
ansible prod --list-hosts
```
### 匹配多个组
```bash
ansbile 'london,boston' --list-hosts
```
### 匹配不属于任何组的主机
```bash
ansible ungrouped --list-hosts
```
## 通配符匹配
```bash
# 匹配 *.example.com 
ansible '*.example.com' --list-hosts
# 匹配以 s 开头的主机或主机组 
ansible 's*' --list-hosts
```
## 通配符组合匹配
```bash
# 匹配包含 *.example.com 但不包含 *.lab.exampl.ecom的主机或主机组
ansible '*.example.com,!*.lab.example.com' --list-hosts
# 匹配属于db组同时属于london组的主机
ansible 'db,&london' --list-hosts
# 匹配在london组或者boston组, 同时在prod组 但不在lb组中的主机 
ansible 'london,boston,&prod,!lb' --list-hosts
```
## 正则表达式匹配

`~` 用来表明这是一个正则表达式匹配

> ansible '~(s|db).*example\.com' --list-hosts

## 通过 --limit 明确指定主机或组
```bash
ansible ungrouped --limit srv1.example.com --list-hosts
ansible ungrouped --limit @retry_hosts.txt --list-hosts 
```
retry_hosts.txt里预先定义内容