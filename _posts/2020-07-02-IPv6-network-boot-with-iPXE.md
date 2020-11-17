---
title: IPv6 network boot with iPXE
tags: ipv6 iPXE
---
<!--more-->

# IPv6地址获取方式
IPv6地址获取主要分为以下几种

IPv6地址获取方式
|               |预设闸道	|地址分配	|DNS    |
|:--------------|:---------|:--------|:------|
|**手动配置**    |**手动**	 |**手动**  |**手动**|
|SLAAC RDNSS	|RA	       |RA        |RA       |
|SLAAC DHCPv6	|RA        |RA        |DHCP     |
|Stateful DHCPv6|RA	       |DHCP      |DHCP     |


**`RA`**: Router Advertisement

**`SLAAC`**: Stateless Address Autoconfiguration

**`RDNSS`**: Recursive DNS Server

自动获取IPv6地址时获取信息的方式主要通过RA的M位与O位控制.

+ M (Managed Address Configuration)
+ O (Other Configuration)

M = 1 表示 Client需要去DHCPv6获取 IPv6 Prefix

O =1 表示 Client需要去DHCPv6获取DNS等信息

```txt
1. SLAAC + RDNSS M = 0, O = 0 Client将从RA获得prefix, DNS等信息. RA一般由Router提供, 在Linux环境下可以通过radvd服务提供
2. SLAAC + DHCPv6 M = 0, O = 1 Client将从RA获取prefix, 其他信息比如DNS由DHCPv6提供
3. Stateful DHCPv6 M = 1, O = 1 Client将从DHCPv6获取所有信息
```

# IPv6地址获取流程
Client 使用 `UDP 546` 端口, Server 使用 `UDP 547` 端口, 但IPv6不使用 broadcast方式获取地址.

假设

Client的 link-local address 是 fe80::6e92:bfff:fe07:bbb9

Server的 link-local address 是 fe80::21b:21ff:fe36:607e

IPv6 multicast addresses **`ff01::1:2`**

+ DHCPv6 client sends a Solicit from [fe80::6e92:bfff:fe07:bbb9]:546 for [ff01::1:2]:547
+ DHCPv6 server replies with an Advertise from [fe80::21b:21ff:fe36:607e]:547 for [fe80::6e92:bfff:fe07:bbb9]:546
+ DHCPv6 client replies with a Request from [fe80::6e92:bfff:fe07:bbb9]:546 for [ff01::1:2]:547
+ DHCPv6 server finishes with an Reply from [fe80::21b:21ff:fe36:607e]:547 for [fe80::6e92:bfff:fe07:bbb9]:546

![DHCPv6-flow](/assets/img/blog/ipxe/DHCPv6-flow.jpg)

# RADVD
Managed will tell the target host to request its IPv6 address from a DHCPv6 server instead of using SLAAC.

Other is important bit, as it tells the target host that (non-address related) configuration information is available from the DHCPv6 server.

`/etc/radvd.conf`
```txt
interface eth0
{
        AdvSendAdvert on;
        MinRtrAdvInterval 30;
        MaxRtrAdvInterval 100;
        AdvManagedFlag on;
        AdvOtherConfigFlag on;
        prefix 3ffe:501:ffff:100::/64
        {
                AdvOnLink on;
                AdvAutonomous on;
                AdvRouterAddr off;
        };

};
```

启用radvd时必须启用 IPv6 forwarding功能
```bash
echo "net.ipv6.conf.all.forwarding = 1" >> /etc/sysctl.conf | sysctl -p
```

# DHCPv6 (isc-dhcp-server)
`/etc/dhcp/dhcpd6.conf`

```txt
default-lease-time 2592000;
preferred-lifetime 604800;
option dhcp-renewal-time 3600;
option dhcp-rebinding-time 7200;
allow leasequery;
option dhcp6.preference 255;
option dhcp6.info-refresh-time 21600;
dhcpv6-lease-file-name "/var/lib/dhcpd/dhcpd6.leases";

option dhcp6.name-servers 2001:4860:4860::8888;  # Use Google's Public DNS Server
option dhcp6.domain-search "test.example.com","example.com";
option dhcp6.user-class code 15 = string;
option dhcp6.vendor-class-data code 16 = string;
option dhcp6.bootfile-url code 59 = string;
option dhcp6.client-arch-type code 61 = array of unsigned integer 16;

if exists dhcp6.user-class and
        substring(option dhcp6.user-class, 2, 4) = "iPXE" {
        option dhcp6.bootfile-url "http://[3ffe:501:ffff:100::1]/start.ipxe";
} else {
        if option dhcp6.client-arch-type = 00:00 {
         option dhcp6.bootfile-url "tftp://[3ffe:501:ffff:100::1]/undionly.kpxe"; # Standard PC BIOS
        } elsif option dhcp6.client-arch-type = 00:06 {
         option dhcp6.bootfile-url "tftp://[3ffe:501:ffff:100::1]/ipxe.efi"; # 32-bit x86 EFI
        } elsif option dhcp6.client-arch-type = 00:07 {
         option dhcp6.bootfile-url "tftp://[3ffe:501:ffff:100::1]/ipxe.efi"; # 64-bit x86 EFI
        } elsif option dhcp6.client-arch-type = 00:09 {
         option dhcp6.bootfile-url "tftp://[3ffe:501:ffff:100::1]/ipxe.efi"; # 64-bit x86 EFI(obsolete)
        } elsif option dhcp6.client-arch-type = 00:0a {
         option dhcp6.bootfile-url "tftp://[3ffe:501:ffff:100::1]/ipxe.efi"; # 32-bit ARM EFI
        } elsif option dhcp6.client-arch-type = 00:0b {
         option dhcp6.bootfile-url "tftp://[3ffe:501:ffff:100::1]/arm64.efi"; # 64-bit ARM EFI
        } 
}

subnet6 3ffe:501:ffff:100::/64 {
        range6 3ffe:501:ffff:100::10 3ffe:501:ffff:100::50;
        range6 3ffe:501:ffff:100:: temporary;
        prefix6 3ffe:501:ffff:100:: 3ffe:501:ffff:111:: /64;
}

```

# tftp server
`/etc/xinetd.d/tftp`

flags默认是IPv4 仅支持IPv4; 修改为IPv6后同时支持IPv6和IPv4
```txt
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /var/lib/tftpboot
        disable                 = no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv6
}
```

# http server
Just deploy a simple http server to serve http request.

# iPXE
`start.ipxe`
```txt
#!ipxe
set conn_type http
chain --autofree http://[3ffe:501:ffff:100::1]/menu.ipxe || echo HTTP failed, localbooting...
```

`menu.ipxe`
```txt
#!ipxe
show ip6
set menu-timeout 5000
set submenu-timeout ${menu-timeout}

# Ensure we have menu-default set to something
isset ${menu-default} || set menu-default localhdd

set boot-url http://[3ffe:501:ffff:100::1]
set keep-san 1

######## MAIN MENU #############
:start
menu iPXE Boot Menu
item
item --gap --      -------Advanced configuration -----------
item --key 0 localhdd   [0] Local HDD
item --key 2 linux      [2] Linux Install
item --gap --            ---------------------------

choose --default  ${menu-default} --timeout 30000 target && goto ${target}

##### Main Items ########
:localhdd
exit

:linux
menu Linux Menu
item redhat74   RedHat74

choose target && goto ${target}

:redhat74
echo Install Red Hat 7.4
set centos_url ${boot-url}/linux/redhat/7.4
iseq ${platform} efi && goto centos_efi || goto centos_legacy
        :centos_efi
        echo Starting Install RedHat (UEFI)
        kernel ${centos_url}/images/pxeboot/vmlinuz initrd=initrd.img inst.loglevel=debug repo=${centos_url} noipv4 ip=dhcp6
        initrd ${centos_url}/images/pxeboot/initrd.img
        boot
        :centos_legacy
        echo Starting Install CentOS (Legacy BIOS)
        kernel ${centos_url}/images/pxeboot/vmlinuz  initrd=initrd.img repo=${centos_url}
        initrd ${centos_url}/images/pxeboot/initrd.img
        boot

```

# DHCPv6 Options
The list of assigned values used in DHCPv6 is available at:

https://www.iana.org/assignments/dhcpv6-parameters/dhcpv6-parameters.xhtml

References
 
 [rfc8415](https://tools.ietf.org/html/rfc8415)

 https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/166/b_166_programmability_cg/ipxe.html
 
 https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-comparison_of_dhcpv6_to_radvd
 
 https://wiki.ubuntu.com/UEFI/SecureBoot/PXE-IPv6
