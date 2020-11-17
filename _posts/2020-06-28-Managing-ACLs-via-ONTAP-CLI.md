---
title: Managing ACLs via ONTAP CLI
tags: ONTAP ACL
---
<!--more-->

# Viewing permissions in multiprotocol NAS
There are options to display permissions from both types of clients. For viewing UNIX permissions from Windows property tabs, use the cifs option is-unix-nt-acl-enabled.

```bash
cluster::*> cifs option show -vserver parisi-fields is-unix-nt-acl-enabled
vserver is-unix-nt-acl-enabled
----------- ----------------------
parisi     true
```
When using this option, the Windows clients will show a security tab entry that approximates the UNIX mode bits into ACLs. It will show the owner, group and “other” permissions. It will also attempt to convert the UNIX UID into a Windows-friendly SID so the client can display it. The Windows user will look like this:

![Unix-windows-acl1](assets/img/blog/ontap/Unix-windows-acl1.png)

That user is a “fake SID” that is tied to the cluster’s Storage Virtual Machine. It translates to a SID that ONTAP creates based on the numeric ID of the user or group. The Windows client uses that SID to translate into a name.

For example:
```bash
cluster::*> diag secd authentication translate -node node1 -vserver SVM -win-name UNIXPermUid\root
S-1-5-21-2038298172-1297133386-11111-0

cluster::*> diag secd authentication translate -node node1 -vserver SVM -unix-user-name root
0

cluster::*> diag secd authentication translate -node node1 -vserver SVM -win-name UNIXPermUid\user3
S-1-5-21-2038298172-1297133386-11111-703

cluster::*> diag secd authentication translate -node node1 -vserver SVM -unix-user-name user3
703

cluster::*> diag secd authentication translate -node node1 -vserver SVM -win-name UNIXPermGid\homedirs
S-1-5-21-2038298172-1297133386-22222-1002

cluster::*> diag secd authentication translate -node node1 -vserver SVM -unix-group-name homedirs
1002
```

From Windows, we can see the level of access for the users from the “Change Permissions” window:

![Unix-windows-acl2](assets/img/blog/ontap/Unix-windows-acl2.png)

On the NFS side, mode bits have no clue how to translate NTFS permission concepts like extended attributes. Instead, the clients only know Read, Write, Execute, Traverse, etc. It’s possible to show an approximation of those mode bits in UNIX for NTFS security style volumes with this option:

```bash
cluster::*> nfs server show -fields ntacl-display-permissive-perms
vserver ntacl-display-permissive-perms
----------- ------------------------------
parisi     disabled
```
When that option is disabled, NTFS ACLs show up as closely to UNIX permissions as they can. In the following example, I have an NTFS security style folder that allowed only the owner to have full control, but allows read to “Everyone.” With the option mentioned, we see that reflected as “755” in permissions:

![Unix-windows-acl3](assets/img/blog/ontap/Unix-windows-acl3.png)

```bash
drwxr-xr-x 3 user1 homedirs 4096 Nov 8 14:15 user1
```

# Translating NTFS style DACLs

As previously mentioned, in ONTAP we can view the Windows ACLs on a file, folder or volume using vserver security file-directory show.

```bash
cluster::*> vserver security file-directory show -vserver SVM-path /homedir1/user1

Vserver: SVM
 File Path: /homedir1/user1
 File Inode Number: 10363
 Security Style: mixed
 Effective Style: ntfs
 DOS Attributes: 10
 DOS Attributes in Text: ----D---
Expanded Dos Attributes: -
 UNIX User Id: 701
 UNIX Group Id: 1002
 UNIX Mode Bits: 777
 UNIX Mode Bits in Text: rwxrwxrwx
 ACLs: NTFS Security Descriptor
 Control:0x8004
 Owner:CPOC\user1
 Group:CPOC\Domain Users
 DACL - ACEs
 ALLOW-CPOC\Administrator-0xe0000040-OI|IO
 ALLOW-CPOC\Administrator-0x1201ff-CI
 ALLOW-CPOC\user1-0x10000000-OI|IO
 ALLOW-CPOC\user1-0x1f01ff-CI
 ALLOW-Everyone-0xa0000000-OI|IO
 ALLOW-Everyone-0x1200a9-CI
```

However, as you can see, those ACLs don’t make a ton of sense unless you can read hexadecimal. (If you can, more power to ya.)

Let’s break down the ACLs a bit to understand them better.

+ First, DACL means “Discretionary Access Control List.” From MSDN:
+ An access control list that is controlled by the owner of an object and that specifies the access particular users or groups can have to the object.
+ In the DACLs above, we can see whether the DACL is an ALLOW or a DENY ACL. (Deny ACLs override ALLOWS.) We can also see the user or group being allowed access. After that, the information isn’t really in a “human readable” format.
+ The CI, IO, OI values are “ACE strings” and tell us whether the ACL was inherited by the owner or container. MSDN has a handy list of those here: ACE Strings

The rest of the ACLs are hexadecimal values and translate into what the actual permissions that were set were.

# Expanding ACLs
Rather than try to decode all of those, ONTAP has an option on the **file-directory show** command that allows you to expand the ACL mask from the CLI (-expand-mask). This actually cracks open the DACLs and shows an expanded view of what actual permissions are allowed.

For example:
```bash
cluster::> vserver security file-directory show -vserver parisi -path /cifs -expand-mask true

Vserver: parisi
 File Path: /cifs
 File Inode Number: 64
 Security Style: ntfs
 Effective Style: ntfs
 DOS Attributes: 10
 DOS Attributes in Text: ----D---
Expanded Dos Attributes: 0x10
 ...0 .... .... .... = Offline
 .... ..0. .... .... = Sparse
 .... .... 0... .... = Normal
 .... .... ..0. .... = Archive
 .... .... ...1 .... = Directory
 .... .... .... .0.. = System
 .... .... .... ..0. = Hidden
 .... .... .... ...0 = Read Only
 UNIX User Id: 0
 UNIX Group Id: 0
 UNIX Mode Bits: 777
 UNIX Mode Bits in Text: rwxrwxrwx
 ACLs: NTFS Security Descriptor
 Control:0x8004

1... .... .... .... = Self Relative
 .0.. .... .... .... = RM Control Valid
 ..0. .... .... .... = SACL Protected
 ...0 .... .... .... = DACL Protected
 .... 0... .... .... = SACL Inherited
 .... .0.. .... .... = DACL Inherited
 .... ..0. .... .... = SACL Inherit Required
 .... ...0 .... .... = DACL Inherit Required
 .... .... ..0. .... = SACL Defaulted
 .... .... ...0 .... = SACL Present
 .... .... .... 0... = DACL Defaulted
 .... .... .... .1.. = DACL Present
 .... .... .... ..0. = Group Defaulted
 .... .... .... ...0 = Owner Defaulted

Owner:BUILTIN\Administrators
 Group:BUILTIN\Administrators
 DACL - ACEs
 ALLOW-Everyone-0x1f01ff
 0... .... .... .... .... .... .... .... = Generic Read
 .0.. .... .... .... .... .... .... .... = Generic Write
 ..0. .... .... .... .... .... .... .... = Generic Execute
 ...0 .... .... .... .... .... .... .... = Generic All
 .... ...0 .... .... .... .... .... .... = System Security
 .... .... ...1 .... .... .... .... .... = Synchronize
 .... .... .... 1... .... .... .... .... = Write Owner
 .... .... .... .1.. .... .... .... .... = Write DAC
 .... .... .... ..1. .... .... .... .... = Read Control
 .... .... .... ...1 .... .... .... .... = Delete
 .... .... .... .... .... ...1 .... .... = Write Attributes
 .... .... .... .... .... .... 1... .... = Read Attributes
 .... .... .... .... .... .... .1.. .... = Delete Child
 .... .... .... .... .... .... ..1. .... = Execute
 .... .... .... .... .... .... ...1 .... = Write EA
 .... .... .... .... .... .... .... 1... = Read EA
 .... .... .... .... .... .... .... .1.. = Append
 .... .... .... .... .... .... .... ..1. = Write
 .... .... .... .... .... .... .... ...1 = Read

ALLOW-Everyone-0x10000000-OI|CI|IO
 0... .... .... .... .... .... .... .... = Generic Read
 .0.. .... .... .... .... .... .... .... = Generic Write
 ..0. .... .... .... .... .... .... .... = Generic Execute
 ...1 .... .... .... .... .... .... .... = Generic All
 .... ...0 .... .... .... .... .... .... = System Security
 .... .... ...0 .... .... .... .... .... = Synchronize
 .... .... .... 0... .... .... .... .... = Write Owner
 .... .... .... .0.. .... .... .... .... = Write DAC
 .... .... .... ..0. .... .... .... .... = Read Control
 .... .... .... ...0 .... .... .... .... = Delete
 .... .... .... .... .... ...0 .... .... = Write Attributes
 .... .... .... .... .... .... 0... .... = Read Attributes
 .... .... .... .... .... .... .0.. .... = Delete Child
 .... .... .... .... .... .... ..0. .... = Execute
 .... .... .... .... .... .... ...0 .... = Write EA
 .... .... .... .... .... .... .... 0... = Read EA
 .... .... .... .... .... .... .... .0.. = Append
 .... .... .... .... .... .... .... ..0. = Write
 .... .... .... .... .... .... .... ...0 = Read

```
This also works with NFSv4 ACLs:
```bash
cluster::*> vserver security file-directory show -vserver DEMO -path /shared/unix -expand-mask true

                Vserver: DEMO
              File Path: /shared/unix
      File Inode Number: 20034
         Security Style: unix
        Effective Style: unix
         DOS Attributes: 10
 DOS Attributes in Text: ----D---
Expanded Dos Attributes: 0x10
     ...0 .... .... .... = Offline
     .... ..0. .... .... = Sparse
     .... .... 0... .... = Normal
     .... .... ..0. .... = Archive
     .... .... ...1 .... = Directory
     .... .... .... .0.. = System
     .... .... .... ..0. = Hidden
     .... .... .... ...0 = Read Only
           UNIX User Id: 1100
          UNIX Group Id: 1101
         UNIX Mode Bits: 770
 UNIX Mode Bits in Text: rwxrwx---
                   ACLs: NFSV4 Security Descriptor
                         Control:0x8014
                              1... .... .... .... = Self Relative
                              .0.. .... .... .... = RM Control Valid
                              ..0. .... .... .... = SACL Protected
                              ...0 .... .... .... = DACL Protected
                              .... 0... .... .... = SACL Inherited
                              .... .0.. .... .... = DACL Inherited
                              .... ..0. .... .... = SACL Inherit Required
                              .... ...0 .... .... = DACL Inherit Required
                              .... .... ..0. .... = SACL Defaulted
                              .... .... ...1 .... = SACL Present
                              .... .... .... 0... = DACL Defaulted
                              .... .... .... .1.. = DACL Present
                              .... .... .... ..0. = Group Defaulted
                              .... .... .... ...0 = Owner Defaulted

                         DACL - ACEs
                           ALLOW-OWNER@-0x1601ff
                              0... .... .... .... .... .... .... .... = Generic Read
                              .0.. .... .... .... .... .... .... .... = Generic Write
                              ..0. .... .... .... .... .... .... .... = Generic Execute
                              ...0 .... .... .... .... .... .... .... = Generic All
                              .... ...0 .... .... .... .... .... .... = System Security
                              .... .... ...1 .... .... .... .... .... = Synchronize
                              .... .... .... 0... .... .... .... .... = Write Owner
                              .... .... .... .1.. .... .... .... .... = Write DAC
                              .... .... .... ..1. .... .... .... .... = Read Control
                              .... .... .... ...0 .... .... .... .... = Delete
                              .... .... .... .... .... ...1 .... .... = Write Attributes
                              .... .... .... .... .... .... 1... .... = Read Attributes
                              .... .... .... .... .... .... .1.. .... = Delete Child
                              .... .... .... .... .... .... ..1. .... = Execute
                              .... .... .... .... .... .... ...1 .... = Write EA
                              .... .... .... .... .... .... .... 1... = Read EA
                              .... .... .... .... .... .... .... .1.. = Append
                              .... .... .... .... .... .... .... ..1. = Write
                              .... .... .... .... .... .... .... ...1 = Read

                           ALLOW-user-prof1-0x1601ff
                              0... .... .... .... .... .... .... .... = Generic Read
                              .0.. .... .... .... .... .... .... .... = Generic Write
                              ..0. .... .... .... .... .... .... .... = Generic Execute
                              ...0 .... .... .... .... .... .... .... = Generic All
                              .... ...0 .... .... .... .... .... .... = System Security
                              .... .... ...1 .... .... .... .... .... = Synchronize
                              .... .... .... 0... .... .... .... .... = Write Owner
                              .... .... .... .1.. .... .... .... .... = Write DAC
                              .... .... .... ..1. .... .... .... .... = Read Control
                              .... .... .... ...0 .... .... .... .... = Delete
                              .... .... .... .... .... ...1 .... .... = Write Attributes
                              .... .... .... .... .... .... 1... .... = Read Attributes
                              .... .... .... .... .... .... .1.. .... = Delete Child
                              .... .... .... .... .... .... ..1. .... = Execute
                              .... .... .... .... .... .... ...1 .... = Write EA
                              .... .... .... .... .... .... .... 1... = Read EA
                              .... .... .... .... .... .... .... .1.. = Append
                              .... .... .... .... .... .... .... ..1. = Write
                              .... .... .... .... .... .... .... ...1 = Read

                           ALLOW-GROUP@-0x1201ff-IG
                              0... .... .... .... .... .... .... .... = Generic Read
                              .0.. .... .... .... .... .... .... .... = Generic Write
                              ..0. .... .... .... .... .... .... .... = Generic Execute
                              ...0 .... .... .... .... .... .... .... = Generic All
                              .... ...0 .... .... .... .... .... .... = System Security
                              .... .... ...1 .... .... .... .... .... = Synchronize
                              .... .... .... 0... .... .... .... .... = Write Owner
                              .... .... .... .0.. .... .... .... .... = Write DAC
                              .... .... .... ..1. .... .... .... .... = Read Control
                              .... .... .... ...0 .... .... .... .... = Delete
                              .... .... .... .... .... ...1 .... .... = Write Attributes
                              .... .... .... .... .... .... 1... .... = Read Attributes
                              .... .... .... .... .... .... .1.. .... = Delete Child
                              .... .... .... .... .... .... ..1. .... = Execute
                              .... .... .... .... .... .... ...1 .... = Write EA
                              .... .... .... .... .... .... .... 1... = Read EA
                              .... .... .... .... .... .... .... .1.. = Append
                              .... .... .... .... .... .... .... ..1. = Write
                              .... .... .... .... .... .... .... ...1 = Read

                           ALLOW-EVERYONE@-0x120080
                              0... .... .... .... .... .... .... .... = Generic Read
                              .0.. .... .... .... .... .... .... .... = Generic Write
                              ..0. .... .... .... .... .... .... .... = Generic Execute
                              ...0 .... .... .... .... .... .... .... = Generic All
                              .... ...0 .... .... .... .... .... .... = System Security
                              .... .... ...1 .... .... .... .... .... = Synchronize
                              .... .... .... 0... .... .... .... .... = Write Owner
                              .... .... .... .0.. .... .... .... .... = Write DAC
                              .... .... .... ..1. .... .... .... .... = Read Control
                              .... .... .... ...0 .... .... .... .... = Delete
                              .... .... .... .... .... ...0 .... .... = Write Attributes
                              .... .... .... .... .... .... 1... .... = Read Attributes
                              .... .... .... .... .... .... .0.. .... = Delete Child
                              .... .... .... .... .... .... ..0. .... = Execute
                              .... .... .... .... .... .... ...0 .... = Write EA
                              .... .... .... .... .... .... .... 0... = Read EA
                              .... .... .... .... .... .... .... .0.. = Append
                              .... .... .... .... .... .... .... ..0. = Write
                              .... .... .... .... .... .... .... ...0 = Read

```

However, with a ton of ACLs on an object, this could get a bit overwhelming. So, translating the hex might be better overall. This blog covers it in a bit more detail:

About the ACCESS_MASK structure

In the above ACL, we see 0x1f01ff for Everyone. That’s Full Control. In addition, 0x10000000 is considered GENERIC_ALL.

# Applying ACLs to objects from the storage
In addition to displaying ACLs, **vserver security file-directory** commands can be used to apply SACLs and DACLs to objects from the cluster’s CLI.

The general steps are covered in this KB article:

https://kb.netapp.com/support/s/article/how-to-modify-permissions-on-files-and-folders-in-clustered-data-ontap-when-there-is-no-permission-to-take-ownership?t=1484836401866

The following shows an example of doing this on a single qtree in ONTAP.

This is a qtree called “mixed.” It has an effective security style of UNIX, unix permissions 770 and root:sharedgroup as the owners.

```bash
cluster::*> vserver security file-directory show -vserver DEMO -path /shared/mixed

                Vserver: DEMO
              File Path: /shared/mixed
      File Inode Number: 20035
         Security Style: mixed
        Effective Style: unix
         DOS Attributes: 10
 DOS Attributes in Text: ----D---
Expanded Dos Attributes: -
           UNIX User Id: 0
          UNIX Group Id: 1206
         UNIX Mode Bits: 770
 UNIX Mode Bits in Text: rwxrwx---
                   ACLs: -
```

To change permissions on this object (or other objects, if desired), first create a security policy:

```bash
cluster::*> file-directory policy create -vserver DEMO -policy-name Policy1
  (vserver security file-directory policy create)
 
cluster::*> vserver security file-directory policy show -vserver DEMO -instance
    Vserver: DEMO
Policy Name: Policy1
```

Then, create a security descriptor, which allows a storage admin to add access control entries (ACEs) to the discretionary access control list (DACL) and the system access control list (SACL). This provides the ability to add, in bulk, an owner, group or control flags in raw hex:

```bash
cluster::*> vserver security file-directory ntfs create -vserver DEMO -ntfs-sd sdname 
 -owner ntfsonly

cluster::*> vserver security file-directory ntfs show -instance
                      Vserver: DEMO
NTFS Security Descriptor Name: sdname
                        Owner: NTAP\ntfsonly
                Primary Group: -
            Raw Control Flags: -

```
Next, create one or more DACLs or SACLs. In this case, I’ve created 2 DACLs. This command allows the following:
```bash
cluster::*> vserver security file-directory ntfs dacl add ?    -vserver                                                   Vserver   [-ntfs-sd]                                             NTFS Security Descriptor Name   [-access-type] {deny|allow}                                               Allow or Deny   [-account]                                                   Account Name or SID  { [[-rights] {no-access|full-control|modify|read-and-execute|read|write}]  DACL ACE's Access Rights  | [ -advanced-rights , ... ]                        DACL ACE's Advanced Access Rights  | [ -rights-raw  ] }                                          *DACL ACE's Raw Access Rights  [ -apply-to {this-folder|sub-folders|files}, ... ]                         Apply DACL Entry
```

The users I’m adding are ntfsonly and student1. Ntfsonly gets full control; student1 gets readonly access. I’m applying the DACL to all objects (this-folder, sub-folders, files).

**NOTE**: If you don’t apply the DACL to the top level folder, you run the risk of denying access to everyone because the owner doesn’t get set properly.

```bash
ontap9-tme-8040::*> vserver security file-directory ntfs dacl add -vserver DEMO -ntfs-sd sdname -access-type allow -account ntfsonly -apply-to this-folder,sub-folders,files -advanced-rights full-control

ontap9-tme-8040::*> vserver security file-directory ntfs dacl add -vserver DEMO -ntfs-sd sdname -access-type allow -account student1 -rights read -apply-to this-folder,sub-folders,files
```

In addition to the ACLs we define, we also get default built-in DACLs. Feel free to delete those as needed.

```bash
ontap9-tme-8040::*> vserver security file-directory ntfs dacl show -vserver DEMO -instance


                      Vserver: DEMO
NTFS Security Descriptor Name: sdname
                Allow or Deny: allow
          Account Name or SID: BUILTIN\Administrators
                Access Rights: full-control
            Raw Access Rights: -
       Advanced Access Rights: -
             Apply DACL Entry: this-folder, sub-folders, files
                Access Rights: full-control

                      Vserver: DEMO
NTFS Security Descriptor Name: sdname
                Allow or Deny: allow
          Account Name or SID: BUILTIN\Users
                Access Rights: full-control
            Raw Access Rights: -
       Advanced Access Rights: -
             Apply DACL Entry: this-folder, sub-folders, files
                Access Rights: full-control

                      Vserver: DEMO
NTFS Security Descriptor Name: sdname
                Allow or Deny: allow
          Account Name or SID: CREATOR OWNER
                Access Rights: full-control
            Raw Access Rights: -
       Advanced Access Rights: -
             Apply DACL Entry: this-folder, sub-folders, files
                Access Rights: full-control

                      Vserver: DEMO
NTFS Security Descriptor Name: sdname
                Allow or Deny: allow
          Account Name or SID: NT AUTHORITY\SYSTEM
                Access Rights: full-control
            Raw Access Rights: -
       Advanced Access Rights: -
             Apply DACL Entry: this-folder, sub-folders, files
                Access Rights: full-control

                      Vserver: DEMO
NTFS Security Descriptor Name: sdname
                Allow or Deny: allow
          Account Name or SID: NTAP\ntfsonly
                Access Rights: -
            Raw Access Rights: -
       Advanced Access Rights: full-control
             Apply DACL Entry: this-folder, sub-folders, files
                Access Rights: full-control

                      Vserver: DEMO
NTFS Security Descriptor Name: sdname
                Allow or Deny: allow
          Account Name or SID: NTAP\student1
                Access Rights: read
            Raw Access Rights: -
       Advanced Access Rights: -
             Apply DACL Entry: this-folder, sub-folders, files
                Access Rights: read
6 entries were displayed.
```

Now that the policy is created and I have the desired DACLs and SACLs, I can apply them to whatever paths I want. In the above, I’ve set the DACLs to only apply to the specific folder. To apply the policy, create a new task and define the path you want to re-ACL. The task will “propogate” by default. You can also specify “replace” if desired.

```bash
cluster::*> file-directory policy task add -vserver DEMO -policy-name Policy1 -path /shared/mixed -ntfs-sd sdname
  (vserver security file-directory policy task add)

cluster::*> file-directory policy task show
  (vserver security file-directory policy task show)

Vserver: DEMO
  Policy: Policy1

   Index  File/Folder  Access           Security  NTFS       NTFS Security
          Path         Control          Type      Mode       Descriptor Name
   -----  -----------  ---------------  --------  ---------- ---------------
   1      /shared/mixed
                       file-directory   ntfs      propagate  sdname

```
Once everything appears in order, apply the policy:

```bash
cluster::*> file-directory apply -vserver DEMO -policy-name Policy1
  (vserver security file-directory apply)

[Job 3229] Job is queued: Fsecurity Apply. Use the "job show -id 3229" command to view the status of this operation.
```

If you want status of the progress, use job show. If you want detailed progress, use job show -instance.

Then, check your ACLs. Note how the effective style of the mixed qtree has changed from UNIX to NTFS:

```bash
cluster::*> vserver security file-directory show -vserver DEMO -path /shared/mixed

                Vserver: DEMO
              File Path: /shared/mixed
      File Inode Number: 20035
         Security Style: mixed
        Effective Style: ntfs
         DOS Attributes: 10
 DOS Attributes in Text: ----D---
Expanded Dos Attributes: -
           UNIX User Id: 0
          UNIX Group Id: 0
         UNIX Mode Bits: 777
 UNIX Mode Bits in Text: rwxrwxrwx
                   ACLs: NTFS Security Descriptor
                         Control:0x8014
                         Owner:NTAP\ntfsonly
                         Group:BUILTIN\Administrators
                         DACL - ACEs
                           ALLOW-BUILTIN\Administrators-0x1f01ff-OI|CI
                           ALLOW-BUILTIN\Users-0x1f01ff-OI|CI
                           ALLOW-CREATOR OWNER-0x1f01ff-OI|CI
                           ALLOW-NT AUTHORITY\SYSTEM-0x1f01ff-OI|CI
                           ALLOW-NTAP\ntfsonly-0x1f01ff
                           ALLOW-NTAP\student1-0x120089     

```
If you want to apply the policy to other paths (or multiple paths at once), create new tasks:

```bash
cluster::*> vserver security file-directory show -vserver DEMO -path /shared/security
                Vserver: DEMO
              File Path: /shared/security
      File Inode Number: 96
         Security Style: mixed
        Effective Style: unix
         DOS Attributes: 10
 DOS Attributes in Text: ----D---
Expanded Dos Attributes: -
           UNIX User Id: 0
          UNIX Group Id: 0
         UNIX Mode Bits: 770
 UNIX Mode Bits in Text: rwxrwx---
                   ACLs: -

cluster::*> file-directory policy task add -vserver DEMO -policy-name Policy1 -path /shared/security -ntfs-sd sdname
  (vserver security file-directory policy task add)

cluster::*> file-directory policy task show
  (vserver security file-directory policy task show)
Vserver: DEMO
  Policy: Policy1
   Index  File/Folder  Access           Security  NTFS       NTFS Security
          Path         Control          Type      Mode       Descriptor Name
   -----  -----------  ---------------  --------  ---------- ---------------
   1      /shared/mixed
                       file-directory   ntfs      propagate  sdname
   2      /shared/security
                       file-directory   ntfs      propagate  sdname
2 entries were displayed.

cluster::*> file-directory apply -vserver DEMO -policy-name Policy1
  (vserver security file-directory apply)

[Job 3232] Job is queued: Fsecurity Apply. Use the "job show -id 3232" command to view the status of this operation.

cluster::*> vserver security file-directory show -vserver DEMO -path /shared/security
                Vserver: DEMO
              File Path: /shared/security
      File Inode Number: 96
         Security Style: mixed
        Effective Style: ntfs
         DOS Attributes: 10
 DOS Attributes in Text: ----D---
Expanded Dos Attributes: -
           UNIX User Id: 0
          UNIX Group Id: 0
         UNIX Mode Bits: 777
 UNIX Mode Bits in Text: rwxrwxrwx
                   ACLs: NTFS Security Descriptor
                         Control:0x8014
                         Owner:NTAP\ntfsonly
                         Group:BUILTIN\Administrators
                         DACL - ACEs
                           ALLOW-BUILTIN\Administrators-0x1f01ff-OI|CI
                           ALLOW-BUILTIN\Users-0x1f01ff-OI|CI
                           ALLOW-CREATOR OWNER-0x1f01ff-OI|CI
                           ALLOW-NT AUTHORITY\SYSTEM-0x1f01ff-OI|CI
                           ALLOW-NTAP\ntfsonly-0x1f01ff
                           ALLOW-NTAP\student1-0x120089

```
Example of a running job with more information:

```bash
cluster::*> job show -id 3317 -instance
                      Job ID: 3317
              Owning Vserver: cluster
                        Name: Fsecurity Apply
                 Description: File Directory Security Apply Job
                    Priority: Low
                        Node: cluster02
                    Affinity: Cluster
                    Schedule: @now
                  Queue Time: 01/24 09:45:19
                  Start Time: 01/24 09:45:19
                    End Time: -
              Drop-dead Time: -
                  Restarted?: false
                       State: Running
                 Status Code: 0
           Completion String:
                    Job Type: FSEC_APPLY
                Job Category: FSECURITY
                        UUID: b9e7bf61-e243-11e6-a40c-00a0986b1210
          Execution Progress: Fsecurity Apply processed 46766 files/dirs. Last Processed: /shared/security/files/topdir_77/subdir_81
                   User Name: admin
                     Process: mgwd
  Restart Is or Was Delayed?: false
Restart Is Delayed by Module: -
```

Reference:

1.  https://whyistheinternetbroken.wordpress.com/2017/02/01/managing-acls-ontap-cli/