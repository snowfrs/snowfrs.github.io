---
title: OpenStack FAQ
tags: OpenStack
---

[OpenStack][openstack]
<!--more-->

+ OpenStack是什么？

OpenStack is a cloud operating system that controls large pools of compute, storage, and networking resources throughout a datacenter, all managed and provisioned through APIs with common authentication mechanisms.

一个开源云操作系统, 用于构建云平台.
+ 资源抽象: OpenStack 将各类硬件资源,通过虚拟化与软件定义的形式，抽象成虚拟的资源池;
+ 资源调度: OpenStack 根据管理员/用户的需求,将资源分配给不同的用户,承载不同的应用;
+ 应用生命周期管理: OpenStack 可以提供基本的应用部署/撤销、自动规模调整等功能;
+ 系统运维: OpenStack 可以提供一定的系统监控能力;
+ 交互: OpenStack 提供接口, 外界通过API/CLI或者GUI方式交互.

+ OpenStack主要组件简单介绍 
  - Nova: 用于在计算级别管理虚拟机, 并在计算或管理程序级别执行其他计算任务
  - Neutron: 为虚拟机、计算和控制节点提供网络功能
  - Keystone: 为所有云用户和OpenStack云服务提供身份认证服务
  - Horizon: 提供图形用户界面，使用图形化界面可以很轻松完成各种日常操作
  - Cinder: 提供块存储功能 
  - Swift: 提供对象存储功能
  - Glance: 提供镜像服务，使用glance的管理平台上传和下载云镜像
  - Heat: 提供编排服务。使用Heat管理平台可以轻松将虚拟机作为堆栈，并根据需要将虚拟机扩展或收缩
  - Ceilometer: 提供计量和监控功能
  
+ 哪些些服务通常运行在控制节点 
  - 身份认证、访问控制(KeyStone) 镜像服务 (Glance)
  - Nova服务 Nova API, Nova Scheduler, Nova DB
  - 块存储 (Cinder)和对象存储服务 (Swift) 
  - Ceilometer计量和监控服务 
  - MariaDB/MySQL 和 RabbitMQ 服务 
  - 网络(Neutron)和网络代理的管理服务 
  - 编排服务 (Heat)
  
+ 哪些服务通常运行在计算节点 
  - Nova Compute 
  - 网络服务 比如OVS

+ 计算节点上虚拟机的默认存放路径是哪里 
  - 虚拟机存储在计算节点的 `/var/lib/nova/instances`

+ Glance镜像的默认存放路径是哪里 
  - 一般存放在控制节点的 `/var/lib/glance/images`

+ 使用命令行启动一个虚拟机 
  ```bash
  # openstack server create --flavor {flavor-name} --image {Image-Name-Or-Image-ID}  --nic net-id={Network-ID} --security-group {Security_Group_ID} –key-name {Keypair-Name} <VM_Name>
  ```

+ 如何在OpenStack中显示用户的网络命名空间列表 
  ```bash
  # ip netns list
  ```

+ 如何在OpenStack中执行网络命名空间内的命令 
  ```bash
  #ip netns exec {network-space} <command>
  ```

+ Glance服务中如何使用命令上传和下载镜像 
  ```bash
  # openstack image create --disk-format qcow2 --container-format bare   --public --file {Name-Cloud-Image}.qcow2     <Cloud-Image-Name>

  # glance image-download --file <Cloud-Image-Name> --progress  <Image-ID>
  ```

+ OpenStack如何将虚拟机从错误状态转为活动状态 
  ```bash 
  # nova reset-state --active {Instance_id}
  ```

+ 使用命令行获取可使用的浮动ip列表 
  ```bash 
  # openstack ip floating list |grep None| head -10
  ```

+ 如何在特定可用区域或计算节点上配置虚拟机 
  ```bash 
  # openstack server create --flavor m1.tiny --image cirros --nic net-id=e0be93b8-728b-4d4d-a272-7d672b2560a6 --security-group NonProd_SG  --key-name linuxtec --availability-zone NonProduction:compute-02  nonprod_testvm
  ```

+ 在特定计算节点上获取配置的虚拟机列表 
  ```bash
  # openstack server list --all-projects --long -c Name -c Host |grep -i {Compute-Node-Name}
  ```

+ 如何使用命令行查看实例的控制台日志 
  ```bash 
  # openstack console log show {Instance-id}
  ```

+ 如何获取实例的控制台URL地址 
  ```bash 
  # openstack console url show {Instance-id}
  ```

+ 如何使用命令行创建可启动的存储卷 
  ```bash
  ##获取镜像列表 
  # openstack image list |grep -i cirros 
  ##使用image创建8G的可启动存储卷 
  # cinder create --image-id {Image-id} --display-name {Display-name} 8
  ```

+ 列出所有在OpenStack中创建的项目或用户 
  ```bash
  # openstack project list --long
  ```

+ 如何显示 OpenStack 服务端点列表 
  
  OpenStack服务端点被分为3类：**公共端点** **内部端点** **管理端点** 
  ```bash
  # openstack catalog list 
  # openstack catalog show keystone
  ```

+ 在控制节点上应该按照什么步骤重启nova服务 
  ```bash
  service nova-api restart 
  service nova-cert restart 
  service nova-conductor restart 
  service nova-consoleauth restart 
  service nova-scheduler restart
  ```

+ 假如计算节点上为数据流量配置了一些DPDK端口，如何检查DPDK端口的状态 
  ```bash
  # ovs-appctl bond/show | grep dpdk 
  active slave mac: 90:38:09:ac:7a:99(dpdk0) 
  slave dpdk0: enabled 
  slave dpdk1: enabled 
  # dpdk-devbind.py --status
  ```

+ 如何使用命令行向已存在的安全组中添加新规则 
  ```bash 
  # neutron security-group-rule-create --protocol <tcp or udp>  --port-range-min <port-number> --port-range-max <port-number> --direction <ingress or egress>  --remote-ip-prefix <IP-address-or-range> Security-Group-Name
  ```

+ 如何查看控制节点和计算节点的OVS桥配置 
  ```bash 
  # ovs-vsctl show
  ```

+ 计算节点上的桥`br-int`作用是什么
  
   集成桥 br-int 对来自和运行在计算节点上的实例的流量执行VLAN标记和取消标记 
   
   数据包从实例的n/w接口发出 使用虚拟接口qvo通过Linux桥(qbr). 
   
   qvb接口用来连接Linux桥，qvo接口用来连接集成桥br-int. 
   
   br-int上的qvo端口有一个内部VLAN标签，这个标签是用于当数据包到达br-int时候贴到数据包头部的.

+ 计算节点上的桥`br-tun`作用是什么 
  
  隧道桥(br-tun) 根据OpenFlow规则将VLAN标记的流量从集成网桥转换为隧道ID
  
  br-tun允许不同网络的实例彼此进行通信. 
  
  隧道有利于封装在非安全网络上传输的流量，它支持两层网络 GRE 和 VXLAN

+ 外部OVS桥`br-ex`的作用是什么 
  
  此网桥转发来自网络的流量，允许外部访问实例. 
  
  br-ex连接物理接口比如eth2, 这样用户网络的浮动IP数据从物理网络接收并路由到用户网络端口

+ OpenFlow规则的作用是什么 
  
  OpenFlow规则是一种机制，这种机制定义了一个数据包如何从源到达目的地。
  
  OpenFlow规则存储在flow表中。flow表是OpenFlow交换机的一部分. 当一个数据包到达交换机就会被第一个flow表检查，如果不匹配flow表中的任何入口, 那么这个数据包会被丢弃或者转发到其他flow表中.

+ 如何查看 OpenFlow 交换机的信息？ 比如端口 表编号 缓存编号 
  ```bash
  # ovs-ofctl show br-int
  ```

+ 如何显示交换机的所有flow的入口 
  ```bash
  # ovs-ofctl dump-flows br-int
  ```

+ 什么是Neutron代理？ 如何显示所有Neutron代理？ 

  OpenStack Neutron服务器充当中心控制器，实际网络配置是在计算节点或者网络节点上执行的。Neutron代理是计算节点或者网络节点上进行配置更新的软件实体。Neutron代理通过Neutron服务和消息队列来和中心Neutron服务通信 
  ```bash
  # openstack network agent list -c 'Agent type' -c Host -c Alive -c State
  ```

+ CPU Pinning是什么 
  
  CPU Pinning是指为某个虚拟机保留物理核心。也称为CPU隔离。 
  
  目的： 确保虚拟机只能在专用核心上运行 确保公共主机进程不在这些核心上运行 也可以认为Pinning是物理核心到一个用户虚拟CPU(vCPU)的一对一映射。


## 其他云计算问题
+ 什么是云计算

云计算是一种采用按量付费的模式，基于虚拟化技术，将相应计算资源（如网络、存储等）池化后，提供便捷的、高可用的、高扩展性的、按需的服务（如计算、存储、应用程序和其他 IT 资源）

+ 云计算的三种服务模式
  - IaaS: 基础设施即服务
  - PaaS: 平台即服务
  - SaaS: 软件即服务

+ 常见的虚拟化厂商即相应的产品
  - VMware      ESXi
  - RedHat      KVM
  - Citrix      XEN
  - Microsoft   Hyper-V
  - Oracle      Oracle VM Server(XEN+LINUX内核)   



[openstack]: www.openstack.org