---
title: Remove a Domain Controller from AD
tags: AD Domain
---
<!--more-->

# Determine FSMO role

If the DC which you want to demote holds any FSMO role, you need to transfer the FSMO roles to another DC.

The 5 FSMO roles are:

**Schema Master** (forest-wide)

**Domain Naming Master** (forest-wide)

**RID Master** (domain-specific)

**PDC Emulator** (domain-specific)

**Infrastructure Master** (Domain-specific)


Login as **Domain Administrator** on one DC

In a command prompt window, type
```txt
netdom query fsmo 
```
![remove_domain_controller_1](/assets/img/blog/AD/Remove_Domain_Controller_1.png)

The powershell commands

To determine the domain-specific FSMO roles for a domain

```txt
Get-ADDomain | Select-Object InfrastructureMaster, RIDMaster, PDCEmulator
```

To determine the forest-specific FSMO roles for a Forest
```txt
Get-ADForest | Select-Object DomainNamingMaster, SchemaMaster
```

to view a list of all DCs that have FSMO roles
```txt
Get-ADDomainController -Filter * | Select-Object Name, Domain, Forest, OperationMasterRoles | Where-Object {$_.OperationMasterRoles} 
```

![remove_domain_controller_2](/assets/img/blog/AD/Remove_Domain_Controller_2.png)

# Transfer the FSMO roles to another DC (Optional)

If the DC you want to demote doesn’t hold any FSMO role, you can skip this step.

Login as **Domain Administrator** on one DC

PowerShell commands

```txt
Move-ADDirectoryServerOperationMasterRole -Identity <targetDC> -OperationMasterRole pdcemulator, ridmaster, infrastructuremaster, schemamaster, domainnamingmaster
```

or

```txt
Move-ADDirectoryServerOperationMasterRole -Identity <targetDC> -OperationMasterRole 0,1,2,3,4 
```

![remove_domain_controller_3](/assets/img/blog/AD/Remove_Domain_Controller_3.png)

# Dry-run
```txt
Test-ADDSDomainControllerUninstallation
```

![remove_domain_controller_4](/assets/img/blog/AD/Remove_Domain_Controller_4.png)

# Demote Domain Controller using PowerShell

## demote domain controller

First, open PowerShell with Administrator privileges. Then type the following command and press Enter. You will be prompted to type in the local administrator’s account twice, and then confirm your action by pressing **Y** or **A**, depending on your preferences.

```txt
Uninstall-ADDSDomainController
```

Immediately afterward, the demotion of the Domain Controller will proceed and the server will be restarted automatically.

## uninstall the role

Once you log in again by opening Server Manager, you will notice that there is the corresponding notification for you to promote the server to a Domain Controller. Obviously, once the Active Directory Domain Services role is still in place.


To uninstall it, use the following command in PowerShell.

```txt
Uninstall-WindowsFeature AD-Domain-Services
```

That’s it! After restarting, your server is no longer a Domain Controller, but just an Active Directory domain member server. 
