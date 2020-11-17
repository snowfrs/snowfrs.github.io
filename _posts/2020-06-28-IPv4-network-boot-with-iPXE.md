---
title: IPv4 network boot with iPXE
tags: ipv4 iPXE
---
<!--more-->

# DHCPv4(isc-dhcp-server)
`/etc/dhcp/dhcpd.conf`
```bash
option domain-name "example.com";
option domain-name-servers 8.8.8.8,8.8.4.4;
option ntp-servers X.X.X.X,X.X.X.X;
option client-arch code 93 = unsigned integer 16;
include "/etc/dhcp/ipxe-option.conf";
default-lease-time 14400;
max-lease-time 86400;
ignore client-updates;
ddns-update-style none;
authoritative;
log-facility local7;
subnet 192.168.1.0 netmask 255.255.255.0 {
  range dynamic-bootp 192.168.1.50 192.168.1.99;
  option subnet-mask 255.255.255.0;
  option routers 192.168.1.1;
  default-lease-time 7200;
  max-lease-time 86400;
  include "/etc/dhcp/ipxe-green.conf";
  include "/etc/dhcp/macaddr.1.conf";
}
```

`/etc/dhcp/ipxe-option.conf`
```bash
# Declare the iPXE/gPXE/Etherboot option space
option space ipxe;
option ipxe-encap-opts code 175 = encapsulate ipxe;

# iPXE options, can be set in DHCP response packet
option ipxe.priority         code   1 = signed integer 8;
option ipxe.keep-san         code   8 = unsigned integer 8;
option ipxe.skip-san-boot    code   9 = unsigned integer 8;
option ipxe.syslogs          code  85 = string;
option ipxe.cert             code  91 = string;
option ipxe.privkey          code  92 = string;
option ipxe.crosscert        code  93 = string;
option ipxe.no-pxedhcp       code 176 = unsigned integer 8;
option ipxe.bus-id           code 177 = string;
option ipxe.san-filename     code 188 = string;
option ipxe.bios-drive       code 189 = unsigned integer 8;
option ipxe.username         code 190 = string;
option ipxe.password         code 191 = string;
option ipxe.reverse-username code 192 = string;
option ipxe.reverse-password code 193 = string;
option ipxe.version          code 235 = string;
option iscsi-initiator-iqn   code 203 = string;

# iPXE feature flags, set in DHCP request packet
option ipxe.pxeext    code 16 = unsigned integer 8;
option ipxe.iscsi     code 17 = unsigned integer 8;
option ipxe.aoe       code 18 = unsigned integer 8;
option ipxe.http      code 19 = unsigned integer 8;
option ipxe.https     code 20 = unsigned integer 8;
option ipxe.tftp      code 21 = unsigned integer 8;
option ipxe.ftp       code 22 = unsigned integer 8;
option ipxe.dns       code 23 = unsigned integer 8;
option ipxe.bzimage   code 24 = unsigned integer 8;
option ipxe.multiboot code 25 = unsigned integer 8;
option ipxe.slam      code 26 = unsigned integer 8;
option ipxe.srp       code 27 = unsigned integer 8;
option ipxe.nbi       code 32 = unsigned integer 8;
option ipxe.pxe       code 33 = unsigned integer 8;
option ipxe.elf       code 34 = unsigned integer 8;
option ipxe.comboot   code 35 = unsigned integer 8;
option ipxe.efi       code 36 = unsigned integer 8;
option ipxe.fcoe      code 37 = unsigned integer 8;
option ipxe.vlan      code 38 = unsigned integer 8;
option ipxe.menu      code 39 = unsigned integer 8;
option ipxe.sdi       code 40 = unsigned integer 8;
option ipxe.nfs       code 41 = unsigned integer 8;

# Other useful general options
# http://www.ietf.org/assignments/dhcpv6-parameters/dhcpv6-parameters.txt
option arch code 93 = unsigned integer 16;
```

`/etc/dhcp/ipxe-green.conf`
```bash
allow bootp;
allow booting;
next-server X.X.X.X; # TFTP Server

# Disable ProxyDHCP, we're in control of the primary DHCP server
option ipxe.no-pxedhcp 1;


if exists user-class and option user-class = "iPXE" {
        filename "http://boot.example.com/start.ipxe";
} else {
        if option client-arch = 00:00 {
                filename "undionly.kpxe";  # Standard PC BIOS
        } elsif option client-arch = 00:06 {
                filename "ipxe.efi";       # 32-bit x86 EFI
        } elsif option client-arch = 00:07 {
                filename "ipxe.efi";       # 64-bit x86 EFI
        } elsif option client-arch = 00:09 {
                filename "ipxe.efi";       # 64-bit x86 EFI(obsolete)
        } elsif option client-arch = 00:0a {
               filename "ipxe.efi";        # 32-bit ARM EFI
        } elsif option client-arch = 00:0b {
                filename "arm64.efi";      # 64-bit ARM EFI
        }
}
```

`/etc/dhcp/macaddr.1.conf`
```bash
host hostname1 {
 hardware ethernet xx:xx:xx:xx:xx:xx;
 fixed-address 192.168.1.9;
}
```

# tftp server
xinetd

# iPXE
`start.ipxe`
```txt
#!ipxe
############ boot.example.com ###########
set conn_type http
chain --autofree http://example.com/menu.ipxe || echo HTTP failed, localbooting...
```

`menu.ipxe`
```txt
#!ipxe
###### set variables ######
set menu-timeout 5000
set submenu-timeout ${menu-timeout}

# Ensure we have menu-default set to something
isset ${menu-default} || set menu-default localhdd

set boot-url http://boot.example.com
set keep-san 1

############################ MAIN MENU ################################
:start
menu iPXE Boot Menu
item
item --gap --           ---------------- Operating Systems ----------------
item --key 0 localhdd                   [0] Local HDD
item --key 1 windows                    [1] Windows Install
item --key 2 linux                      [2] Linux Install
item --key 3 diskutilities              [3] Disk Utilities
item --key 4 backup                     [4] Backup & Recovery
item --key 5 livelinux                  [5] Linux ISO Live
item --key 6 systemrescue               [6] SystemRescue
item --gap --           ---------------- Advanced Options -----------------
item --key c config                     [c] Configure Settings
item --key s shell                      [s] Enter iPxe Shell
item --key r reboot                     [r] Reboot computer
item --key p poweroff                   [p] Power Off
item --key x exit                       [x] Exit iPXE
item --gap --           ---------------------------------------------------
choose --default ${menu-default} --timeout 30000 target && goto ${target}

############################## Main ITEMS ##############################
:localhdd
#sanboot --no-describe --drive 0x80 || goto start
exit

:windows
menu Windows Menu
item win-uefi   Windows Installer(UEFI)
item win-bios   Windows Installer(Legacy BIOS)
item back       Back to Top Level

choose target && goto ${target}

:win-bios
set netX/next-server X.X.X.X  # WDS Server
imgexec tftp://${netX/next-server}//boot//x64//wdsnbp.com

:win-uefi
set netX/next-server X.X.X.X # WDS Server
imgexec tftp://${netX/next-server}//boot//x64//wdsmgfw.efi


:linux
menu Linux Menu
item centos77   CentOS7.7.1908
item redhat74   RedHat 7.4
item back       Back to Top Level

choose target && goto ${target}

:centos77
echo Install CentOS 7.7.1908
set centos_url ${boot-url}/linux/centos/7.7.1908
iseq ${platform} efi && goto centos77_efi || goto centos77_legacy
        :centos77_efi
        echo Starting Install CentOS (UEFI)
        kernel ${centos_url}/images/pxeboot/vmlinuz  initrd=initrd.img repo=${centos_url}
        initrd ${centos_url}/images/pxeboot/initrd.img
        #chain ${centos_url}/EFI/BOOT/BOOTX64.EFI
        boot

        :centos77_legacy
        echo Starting Install CentOS (Legacy BIOS)
        kernel ${centos_url}/images/pxeboot/vmlinuz  initrd=initrd.img repo=${centos_url}
        initrd ${centos_url}/images/pxeboot/initrd.img
        boot
:redhat74
echo Install RedHat 7.4
set centos_url ${boot-url}/linux/redhat/7.4
iseq ${platform} efi && goto redhat74_efi || goto redhat74_legacy
        :redhat74_efi
        echo Starting Install RedHat (UEFI)
        kernel ${centos_url}/images/pxeboot/vmlinuz inst.loglevel=debug inst.dd=hd:LABEL=OEMDRV:/ngbe.iso ip=dhcp
        append initrd=${centos_url}/images/pxeboot/initrd.img
        boot

        :redhat74_legacy
        echo Starting Install RedHat (Legacy BIOS)
        kernel ${centos_url}/images/pxeboot/vmlinuz inst.loglevel=debug inst.dd=hd:LABEL=OEMDRV:/ngbe.iso inst.repo=${centos_url} ip=dhcp
        append initrd=${centos_url}/images/pxeboot/initrd.img
        boot


:diskutilities
menu Disk Utilities
#item gparted Gparted

item back Back to top menu...
iseq ${menu-default} menu-recovery && isset ${submenu-default} && goto menu-recovery-timed ||
choose selected && goto ${selected} || goto start
:menu-recovery-timed
choose --timeout ${submenu-timeout} --default ${submenu-default} selected && goto ${selected} || goto start

:systemrescue
menu SystemRescue
item systemrescue1 SystemRescue
item back Back to top menu...
iseq ${menu-default} menu-recovery && isset ${submenu-default} && goto menu-recovery-timed ||
choose selected && goto ${selected} || goto start
:menu-recovery-timed
choose --timeout ${submenu-timeout} --default ${submenu-default} selected && goto ${selected} || goto start

:systemrescue1
sanboot --drive 0x81 --no-describe ${boot-url}/tools/systemrescuecd-x86-5.2.2.iso


:backup
menu Backup and Recovery
#item clonezilla        Clonezilla
item acronis            Acronis 2016
item back Back to top menu...

iseq ${menu-default} menu-recovery && isset ${submenu-default} && goto menu-recovery-timed ||
choose selected && goto ${selected} || goto start
:menu-recovery-timed
choose --timeout ${submenu-timeout} --default ${submenu-default} selected && goto ${selected} || goto start

:clonezilla
#sanboot --drive 0x81 --no-describe ${boot-url}/tools/clonezilla/clonezilla-live-amd64.iso
set clonezilla_url ${boot-url}/tools/clonezilla
kernel ${clonezilla_url}/live/vmlinuz || read void
initrd ${clonezilla_url}/live/initrd.img || read void
 imgargs vmlinuz initrd=initrd.img boot=live username=root union=overlay config components quiet noswap edd=on nomodeset nodmraid locales=en_US.UTF-8 keyboard-layouts=en ocs_live_run="ocs-live-general" ocs_live_extra_param="" ocs_live_batch=no net.ifnames=0 nosplash noprompt fetch=${clonezilla_url}/live/filesystem.squashfs || read void
#imgargs vmlinuz initrd=initrd.img boot=live username=root locales=en_US.UTF-8 keyboard-layouts=en components fetch=${clonezilla_url}/live/filesystem.squashfs ip=dhcp splash || read void
boot || read void

:acronis
sanboot --drive 0x81 --no-describe ${boot-url}/tools/Acronis-2016.iso
#goto start


:livelinux
menu Linux Live OS
item ubuntu1604 Ubuntu 1604 Live
item back Back to top menu...

iseq ${menu-default} menu-recovery && isset ${submenu-default} && goto menu-recovery-timed ||
choose selected && goto ${selected} || goto start
:menu-recovery-timed
choose --timeout ${submenu-timeout} --default ${submenu-default} selected && goto ${selected} || goto start


:ubuntu1604
set ubuntu_url ${boot-url}/linux/ubuntu
set nfs-server-ip X.X.X.X
set nfs_path      /pxe/tftpboot/linux/ubuntu
kernel ${ubuntu_url}/1604/casper/vmlinuz.efi || read void
initrd ${ubuntu_url}/1604/casper/initrd.lz  || read void
imgargs vmlinuz.efi initrd=initrd.lz root=/dev/nfs boot=casper netboot=nfs nfsroot=${nfs-server-ip}:${nfs_path}/1604 ip=dhcp splash quiet -- || read void
boot || read void

###################### Advanced #######################

:shell
echo Type 'exit' to get back
shell
set menu-timeout 0
set submenu-timeout 0
goto start

:failed
echo Booting failed, dropping to shell
goto shell

:reboot
reboot

:poweroff
poweroff

:exit
exit

:config
config
goto start

:back
set submenu-timeout 0
clear submenu-default
goto start

################################## END ################################
```

# References

https://ipxe.org/

https://netboot.xyz/

https://wiki.ubuntu.com/UEFI/SecureBoot/PXE-IPv6

https://ipxe.org/howto/dhcpd

https://netboot.xyz/