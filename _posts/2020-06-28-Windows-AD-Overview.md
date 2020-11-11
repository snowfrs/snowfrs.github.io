---
title: Windows AD 介绍
tags: AD
---
<!--more-->

# 林(Forest) 域树(Tree) 域(Domain) 子域(Child Domain)
建立的第一个域是根域 (Root Domain)，与此同时，也建立了第一个林和域树。因而这个根域既是林根域也是树根域，因而在网络中建立全新的域时，其实就是建立一个新林，在配置域控的时候不要选错了。根域也是域，只是地位特殊，一个林中只有一个林根域，但可以有多个树根域。拥有共同命名空间的根域和子域构成域树，拥有不同命名空间的域树构成林。域树的名称与第一个域相同，林的名称与第一个域树相同，也与第一个域相同。 

![AD_1](/assets/img/blog/AD/AD_1.jpg)

# DNS服务器 全局编录服务器(GC) 只读域控制器(RODC)
DNS服务器即域名服务器。在域中计算机、集群等名称的解析，需要DNS服务的支持。建立域必须在域中提供DNS服务，如果在配置域过程中勾选DNS服务器，则本机将被配置为DNS服务器（配置程序会检测当前DNS 基础结构来决定DNS服务是否默认勾选）。

全局编录服务器（Global Catalog, GC）可以理解为林中只读的全局缓存，缓存中存储了林中本域内的所有对象的所有属性和其他域的所有对象的部分属性。“全局编录使用户能够在林中的所有域上搜索目录信息，无论数据存储在什么位置。将以最大的速度和最低的网络流量在林中执行搜索。”如果在配置中勾选全局编录服务器，将会使这台域控制器同时成为全局编录服务器。

只读域控制器（Read Only Domain Controller, RODC）。“只读域控制器（RODC）是 Windows Server 2008 操作系统中的一种新类型的域控制器。借助RODC，组织可以在无法保证物理安全性的位置中轻松部署域控制器。RODC 承载Active Directory域服务（AD DS）数据库的只读分区。”“物理安全性不足是考虑部署 RODC 的最常见原因。RODC 提供了一种在要求快速、可靠的身份验证服务但不能确保可写域控制器的物理安全性的位置中更安全地部署域控制器的方法。” 
# AD数据库 日志文件 SYSVOL文件夹
Active Directory使用文件型数据库，数据库引擎是基于JET开发的Extensible Storage Engine(ESE)，也叫做JET Blue。JET Blue计划用于升级Access的数据库引擎JET Red的，但却用于Microsoft的其他产品中，如AD，WINS，Exchange Server等。ESE有能力扩展至16TB容量，容纳10亿对象。数据库所有相关文件默认在%systemroot%\ntds\文件夹内，主要包括：

ntds.dit 数据库文件。有兴趣可以查阅 Active Directory database file NTDS.DIT 详细了解。

edb.chk 检查点文件。对数据库的增删改，在提交更新到数据库之前，检查点文件会记录事务完成情况，如果事务完成就从日志文件提交更新到数据库。

edb.log和edbxxxxx.log等为日志文件。每个日志文件10MB，edb.log文件填满之后，会被重新命名为edbxxxxx.log，文件名编号自增。对数据库的增删改会写入日志文件，用以事务处理。

edbresxxxxx.jrs 为日志保留文件。为日志文件占据磁盘空间，仅当日志文件所在的磁盘空间不足时使用。

edbtmp.log 临时日志文件。当前edb.log被填满时，会创建edbtmp.log继续记录日志，同时当前edb.log被重命名为edbxxxxx.log，而后edbtmp.log被重名为edb.log。

Active Directory使用SYSVOL文件夹（需要放置在NTFS分区中）在DC间共享公共文件，包括登录脚本和策略配置文件等。详细可参考 Sysvol and netlogon share importance in Active Directory 
# FSMO主机角色
Active Directory在多个域控制器（DC）间进行数据复制的时有两种模式：单主机复制模式（Single-Master Model）和多主机复制模式（Multi-Master Model）。

DC间数据复制多采用的多主机复制模式，即允许在任意一台DC上更新数据，然后复制到其他DC，当出现数据冲突时采用一些算法解决（如以最后被写入的数据为准）。多主机复制模式实现了DC间负载均衡和高可用的目的。但对一些数据多主机复制模式带来了难以解决的数据冲突或解决冲突需要付出太大代价的问题，这时Active Directory会采用单主机复制模式，即允许只有一台DC更新数据，然后复制到其他DC。这些数据主要由5种操作主机角色来承担更新的职责，这些角色可以被分配到林中不同DC中，这些角色也称为灵活的单主机操作角色（Flexible Single Master Operation, FSMO），他们分别是：

## 林级别（在林中只能有一台DC拥有该角色）：

架构主机（Schema Master）：架构是林中所有对象和属性的定义，具有架构主机角色的域控制器（DC）才允许更新架构。架构更新会从架构主机复制到林中的其它域控制器中。整个林中只能有一个架构主机。

域命名主机（Domain Naming Master）：具有域命名主机角色的域控制器才允许在林中新增和删除域或新增和删除域的外部引用。整个林中只能由一个域命名主机。

## 域级别（在域中只有一台DC拥有该角色）：

**PDC模拟器（PDC Emulator）**：Windows Server 2000及以后版本中域控制器不再区分PDC(Primary Domain Controller)和BDC(Backup Domain Controller)，但为兼容旧系统和实现PDC上的一些功能，就需要PDC模拟器发挥作用了。这些包括：密码实时更新；域内时间同步；兼容旧有系统（如NT4和Win98）等。

**RID主机（RID Master）**：在Windows系统中，安全主体（如用户和用户组）的唯一标识取决于SID（如用户名不同但是SID相同的用户Windows仍然认为是同一用户）。SID由域SID（同域内都一样）和RID组成。RID主机的作用是负责为安全主体生成唯一的RID。为避免安全主体SID重复，造成安全问题，RID统一从RID主机分配的RID池中生成。

**结构主机（Infrastructure Master）**：结构主机的作用是负责对跨域对象引用进行更新，以确保所有域间操作对象的一致性。在活动目录中有可能一些用户从一个OU转移到另外一个OU，那么用户的DN名就发生变化，这时其它域对于这个用户引用也要发生变化。这种变化就是由基础结构主机来完成。如果架构主机与GC在同一台DC上，架构主机将不会更新任何对象，因为GC已经拥有所有对象和属性的拷贝。所以在多域情况下，建议不要将架构主机设为GC。

以下列表描述了 Active Directory 林中五种唯一的 FSMO 角色，以及这些角色执行的相关操作：

+ 架构主机 (Schema master) － 架构主机角色是林范围的角色，每个林一个。 此角色用于扩展 Active Directory 林的架构或运行 adprep /domainprep 命令。
+ 域命名主机 (Domain naming master) － 域命名主机角色是林范围的角色，每个林一个。 此角色用于向林中添加或从林中删除域或应用程序分区。
+ RID 主机 (RID master) － RID 主机角色是域范围的角色，每个域一个。 此角色用于分配 RID 池，以便新的或现有的域控制器能够创建用户帐户、计算机帐户或安全组。
+ PDC 模拟器 (PDC emulator) － PDC 模拟器角色是域范围的角色，每个域一个。 将数据库更新发送到 Windows NT 备份域控制器的域控制器需要具备这个角色。 此外，拥有此角色的域控制器也是某些管理工具的目标，它还可以更新用户帐户密码和计算机帐户密码。
+ 结构主机 (Infrastructure master) － 结构主机角色是域范围的角色，每个域一个。 此角色供域控制器使用，用于成功运行 adprep /forestprep 命令，以及更新跨域引用的对象的 SID 属性和可分辨名称属性。
# 功能级别
Active Directory新建林时需要确定林和域的功能级别，功能级别决定了Active Directory域服务（AD DS）的功能，也决定了哪些Windows Server操作系统可以被林和域支持成为域控制器。Windows Server在历次改版，也对Active Directory进行改进，形成了不同功能级别，更高的功能级别提供更多的功能，目前已有功能级别包括：Windows 2000 本机模式、Windows Server 2003、Windows Server 2008、Windows Server 2012等。

运行Windows Server 2008的操作系统上可以设定林和域的功能级别为Windows Server 2003，运行Windows Server 2003操作系统的服务器可以加入成为域控制器。但设定林和域的功能级别为Windows Server 2008，运行Windows Server 2003操作系统的服务器将无法加入成为域控制器，但运行Windows Server 2012操作系统的服务器可以。

另外设定的功能级别可以升级不能降价，域功能级别不能低于林的功能级别。 
# 域新任
域信任就是在域之间建立一种关系，使得一个域中的用户可以在另一个域的域控制器上进行验证，但建立信任仅仅是为实现跨域访问资源提供了可能，只有在资源上对用户进行了授权才能最终实现跨域访问。

域信任分为单向和双向，单向就是我信任你但是你不信任我或者反之，双向就是相互信任。另外域信任可配置为具有可传递，就是我信任你所信任的（第三方），可传递的信任省去了在复杂域环境中配置信任关系工作。

同一个林中父域和子域默认存在双向的可传递的信任。域树之间默认存在双向的可传递的信任（Tree Trust），两个不同域树中的域之间可以建立快捷信任（Shortcut Trust），以加快验证过程。不同林之间可以建立林信任（Forest Trust）。

与其他使用Kerberos做验证的目录系统可以建立领域信任（Realm Trust）。
与较老的NT4系统可以建立外部信任（External Trust).

![AD_2](/assets/img/blog/AD/AD_2.jpg)

# 站点
理论上Windows域与物理网络拓扑无关，域中多个域控制器只要满足能够相互通信的条件，可以在同一个子网，也可以分属不同子网；可以在同一个物理位置，也可以分别在不同的物理位置。但域控制器以及域中的计算机之间的通信最终受制于物理的网络拓扑，如域控制器之间的复制和账户验证等与物理位置关系密切。

站点可以看成是域中高速连接的一组计算机，按物理位置将域控制器和计算机部署在不同站点内，可以提高域内域控制器间复制和账户验证的效率。例如一个域中，北京站点有两个域控A和B，上海站点有两个域控C和D，他们之间的复制如果按照BCDA的顺序复制，那将是没有效率的。按ABCD顺序，同一个站点内的域控相互复制，站点间只要复制一次即可。 

**从林中删除域**
https://chrisbeams.wordpress.com/tag/remove-a-domain-using-ntdsutil/

