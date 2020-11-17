---
title: NetApp Clustered ONTAP CLI
tags: NetApp ONTAP
---
<!--more-->

# MISC
```bash
set -privilege advanced (Enter into privilege mode)

set -privilege diagnostic (Enter into diagnostic mode)

set -privilege admin (Enter into admin mode)

system timeout modify 30 (Sets system timeout to 30 minutes)

system node run – node local sysconfig -a (Run sysconfig on the local node)
```

The symbol ! means other than in clustered ontap i.e.
```bash
storage aggregate show -state !online (show all aggregates that are not online)

node run -node <node_name> -command sysstat -c 10 -x 3 (Running the sysstat performance tool with cluster mode)

system node image show (Show the running Data Ontap versions and which is the default boot)

dashboard performance show (Shows a summary of cluster performance including interconnect traffic)

node run * environment shelf (Shows information about the Shelves Connected including Model Number)

network options switchless-cluster show (Displays if nodes are setup for cluster switchless or switched – need to be in advanced mode)

network options switchless-cluster modify true (Sets the nodes to use cluster switchless, setting to false sets the node to use cluster switches – need to be in advanced mode)

security login banner show (Show the current login banner)

security login banner modify -message “Only Authorized users allowed!” (Set the login banner to Only Authorized users allowed)

security login banner modify -message “” (Clears the login banner)

security login motd show (Shows the current Message of the day)

security login motd modify -vserver vserver1 (Modify the Message of the day, use the variable below)

- Operating System = s
- Software Version = r
- Node Name = n
- Username = N
- Time = t
- Date = d

security login motd modify -vserver vserver1 -message “” (Clears the current Message of the Day)
```

# DIAGNOSTICS USER
```bash
security login unlock -username diag (Unlock the diag user)

security login password -username diag (Set a password for the diag user)

security login show -username diag (Show the diag user)
```

# SYSTEM CONFIGURATION BACKUPS
```bash
system configuration backup create -backup-name node1-backup -node node1 (Create a cluster backup from node1)

system configuration backup create -backup-name node1-backup -node node1 -backup-type node (Create a node backup of node1)

system configuration backup settings modify -destination ftp://192.168.1.10/ -username backups (Sets scheduled backups to go to this destination URL)

system configuration backup settings set-password (Set’s the backup password for the destination URL above)
```

# LOGS
To look at the logs within clustered ontap you must log in as the diag user to a specific node
```bash
set -privilege advanced

systemshell -node <node_name>
```
username: diag

password: `<your diag password>`

cd /mroot/etc/mlog

cat command-history.log | grep volume (searches the command-history.log file for the keyword volume)

exit (exits out of diag mode)

http://<cluster-ip address>/spi (loging with your username and password, from here you can browse logs and core dumps)

# DISK SHELVES
```bash
storage shelf show (an 8.3+ command that displays the loops and shelf information)
```

# AUTOSUPPORT
```bash
system node autosupport budget show -node local (In diag mode – displays current time and size budgets)

system node autosupport budget modify -node local -subsystem wafl -size-limit 0 -time-limit 10m (In diag mode – modification as per Netapp KB1014211)

system node autosupport show -node local -fields max-http-size,max-smtp-size (Displays max http and smtp sizes)

system node autosupport modify -node local -max-http-size 0 -max-smtp-size 8MB (modification as per Netapp KB1014211)
```

# NTP
```bash
cluster time-service ntp server create (Configure an NTP server or multiple NTP servers)

cluster time-service ntp server show (Show the current NTP servers)

cluster time-service ntp server modify (Modify the NTP server list)

cluster time-service ntp server delete (Deletes an NTP server)

cluster time-service ntp server reset (Resets configuration, removes all existing NTP servers)

cluster date show (Displays the cluster date)

cluster date modify (Modify the cluster date)
```

# NODES
```bash
system node rename -node <current_node_name> -newname <new_node_name>

system node reboot -node NODENAME -reason ENTER REASON (Reboot node with a given reason. NOTE: check ha policy)
```

# Network Subnets
```bash
network subnet create -subnet-name vmnet2 -broadcast-domain Default -subnet 172.25.2.0/24 -gateway 172.25.2.254
```

# Export-Policy
```bash
vserver export-policy show

vserver export-policy rule show

vserver export-policy rule create  -policyname default -ruleindex 1 -vserver lab -clientmatch 0.0.0.0/0 -protocol any -rorule any -rwrule never -superuser none

vserver export-policy create -policyname home -vserver lab

vserver export-policy rule create -policyname home -clientmatch 172.25.2.0/24 -rorule any -rwrule any -vserver lab -ruleindex 1 -protocol nfs -superuser any
```

# NFS
```bash
nfs create -access true -v3 enabled -v4.0 enabled -tcp enabled -v4.0-acl enabled -v4.0-read-delegation disabled -v4.0-write-delegation disabled -v4-id-domain depA.lab -v4-grace-seconds 45 -v4-acl-preserve enabled -v4.1 disabled -rquota disabled -v4.1-acl disabled -vstorage disabled -v4-numeric-ids enabled -v4.1-read-delegation disabled -v4.1-write-delegation disabled -mount-rootonly enabled -nfs-rootonly disabled -permitted-enc-types des,des3,aes-128,aes-256 -showmount enabled -name-service-lookup-protocol udp -idle-connection-timeout 360 -allow-idle-connection disabled -v3-hide-snapshot disabled -showmount-rootonly disabled -udp enabled -vserver lab
```

# Network Interface
```bash
network interface create -vserver lab -lif cluster1-01_data -role data -service-policy default-data-files -data-protocol nfs,cifs,fcache -address 172.25.2.100 -netmask 255.255.255.0 -home-node cluster1-01 -home-port e0d -status-admin up -failover-policy system-defined -is-dns-update-enabled true -firewall-policy data
```

# Broadcast Domain
```bash
broadcast-domain split -broadcast-domain Default -new-broadcast-domain mgmt -ports cluster1-01:e0c,cluster1-02:e0c -ipspace Default

network port broadcast-domain split -broadcast-domain Default -new-broadcast-domain vmnet2-data -ports cluster1-01:e0d,cluster1-01:e0e,cluster1-02:e0d,cluster1-02:e0e -ipspace Default
```

# iscsi
```bash
iscsi create -target-alias lab -status-admin up -vserver lab

network interface create -vserver lab -lif cluster1-01_iscsi  -role data -data-protocol iscsi -address 172.25.2.101 -netmask-length 24 -home-node cluster1-01 -home-port e0d -status-admin up -broadcast-domain vmnet2-data -failover-policy disabled

iscsi interface enable -vserver lab -lif cluster1-01_iscsi

vserver export-policy create -policyname data -vserver lab
```

# volume
```bash
volume create -volume data -aggregate FACL_cluster1_01 -size 1G -state online -security-style ntfs -policy data -type RW -snapshot-policy default -foreground true -tiering-policy none -vserver lab -junction-path /data
```

# route
```bash 
network route create -vserver lab -destination 172.25.2.0/24 -gateway 172.25.2.254
```

Reference:

https://www.sysadmintutorials.com/