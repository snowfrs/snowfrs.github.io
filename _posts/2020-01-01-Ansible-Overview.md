---
title: Ansible 简介
tags: ansible
---
<!--more-->

# What is Ansible

Ansible is an automation and configuration management technology used to provision, deploy, and manage compute infrastructure across cloud, virtual, and physical environments.

Ansible 是一种自动化和配置管理技术, 用于跨云, 虚拟和物理环境配置, 部署和管理计算基础架构.

# Feature

+ **`Agentless`**: 被控端只需要运行sshd,不需要额外安装客户端
+ **`Serverless`**: 主控端无需启动任何服务
+ **`SSH by default`**: 默认使用ssh控制节点
+ **`YAML`**: 使用YAML语言定制playbook
+ **`Modules in any language`**: 基于模块工作
+ **`Strong multi-tier solution`**: 可多级控制

# 基本组件

![ansible_overview](/assets/img/blog/ansible/ansible1.png)

+ 核心: ansible
+ 核心模块(Core Modules): ansible自带模块
+ 扩展模块(Custom Modules):
+ 插件(Plugins): 模块功能补充
+ Playbooks:
+ 连接插件(Connection Plugins): ansible基于Connection Plugins连接到各个主机.默认使用ssh连接方式,同时也支持其他连接方式.
+ 主机群(Host Inventory):

# 工作机制

Ansible在主控节点将ansible模块通过ssh协议(或者其他比如kerberos,LDAP)推送到被控端执行，执行完之后自动删除，可以使用版本控制系统来管理自定义模块及playbooks.

![ansible_workflow](/assets/img/blog/ansible/ansible2.png)

## 安装及配置
```bash
dnf install ansible -y
```
## 配置文件优先级

    1. 变量 ANSIBLE_CONFIG
    2. 当前工作目录下的 ./ansible.cfg文件
    3. 当前用户家目录下的 ~/.ansible.cfg文件
    4. 系统默认 /etc/ansible/ansible.cfg文件

## 配置文件字段

+ [defaults] : 通用配置
+ [inventory] : 与主机清单相关的配置
+ [privilege_escalation] : 特权相关的配置
+ [paramiko_connection] : paramiko是RHEL6及之前的版本中默认使用的ssh连接方式
+ [ssh_connection] : openssh连接的相关配置, openssh是RHEL6之后的ansible默认的ssh连接方式
+ [persistent_connetion] : 持久连接的配置项
+ [accelerate] : 加速模式配置项
+ [selinux] : selinux相关的配置项
+ [colors] : ansible命令输出的颜色相关的配置项
+ [diff] : 定义运行时打印diff(变更前后差异)