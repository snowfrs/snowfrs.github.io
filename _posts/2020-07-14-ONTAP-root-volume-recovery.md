---
title: ONTAP root volume recovery
tags: NetApp ONTAP
---
<!--more-->

I have two nodes cluster(vsim 9.7) running on VMware WorkStation Pro 15.5.6. Suddenly one node is offline, but I don't know how it happens. After reboot the node I get

```txt
***********************

**  SYSTEM MESSAGES  **

***********************

Internal error: Cannot open corrupt replicated database. Automatic recovery attempt has failed or is disabled. Check the event logs for details. This node is not fully operational. Contact support personnel for the root volume recovery procedures.
```

**Solution**:

Shut down the non-epsilon node, halt the epsilon node and boot to the loader. unset the boot_recovery bootarg and see if it will come back up.

Press the **SPACEBAR** key immediately when you see the message 
**Hit [Enter] to boot immediately, or any other key for command prompt**.

```txt
VLOADER> printenv bootarg.init.boot_recovery

VLOADER> unsetenv bootarg.init.boot_recovery

VLOADER> printenv bootarg.rdb_corrupt

VLOADER> unsetenv bootarg.rdb_corrupt

VLOADER> boot
```

other root.vol.recovery.reqd condition:

+ mgmtgwd.rootvol.recovery.new
+ mgmtgwd.rootvol.recovery.different
+ mgmtgwd.rootvol.recovery.changed
+ mgmtgwd.rootvol.recovery.takeover.changed
+ rdb.env.replicaCorrupt
+ rdb.corruption.reported
+ mgmtgwd.rootvol.recovery.low.space