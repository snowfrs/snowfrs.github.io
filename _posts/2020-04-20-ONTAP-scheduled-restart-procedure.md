---
title: ONTAP Scheduled restart procedure
tags: NetApp ONTAP
---
<!--more-->

# Clustered Data ONTAP procedure to bring system down
```txt
1. disconnect and shut down all the connected CIFS/NFS clients
2. if there are any hosts that have FCP or iSCSI-based LUNS, shut them down before shutting down NetAPP
3. for 2-node cluster (useful for us)

::> cluster ha modify –configured false

::> storage failover modify –node * -enabled false

     for 4 +-node cluster

::> storage failover modify -node * -enabled false

4. halt node

: : > system node halt -node trustnetic-cluster-01 -inhibit-takeover true -skip-lif-migration-before-shutdown true -reason "power maintenance"

: : > system node halt -node trustnetic-cluster-02 -inhibit-takeover true -skip-lif-migration-before-shutdown true -reason "power maintenance"

5. Physically power down the head, then all the attached disk shelves as needed.

6. Physically unplug the cables from power supplies on the back of the storage systems and shelves, to avoid any electrical issues when external power is restored.
```

# Procedure to bring the system back online
```txt
1. Reconnect all the power cables if previously disconnected.
2. Power on core switches.
3. Physically power up all disk shelves first. Wait until 30 seconds after the last disk shelf is powered on, then power on the storage system head so that all disks will be available when they are required by Data ONTAP.
4. Verify the storage system is up, all services are running, and network connectivity is present.
5. For clustered ONTAP systems, check cluster show and storage failover show to confirm CFO/SFO is configured/enabled. - If on version prior to 8.2 in which cluster ha and/or storage failover were disabled, run the following commands: 

::> cluster ha modify –configured true ::> storage failover modify –node * –enabled true
```

# Verify and check status
```bash
cluster show
aggr show
lun show
system node show
vserver show
cifs show
volume show
igroup show
cf status
system health status show
system health subsystem show
```