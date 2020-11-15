---
title: Remove a Domain Using NTDSUTIL
tags: AD Domain
---
<!--more-->

So say for some reason you want to remove a Domain from Active Directory that no longer exists… how do you do it?

As always with Metadata Cleanups NTDSUTIL is your friend.

To remove the domain you need to remove the following using NTDSUTIL:

1. All Domain Controllers for the domain you want to remove.

2. All Naming Contexts for the Domain you want to remove.

You can then remove the actual Domain itself. Its important to remember when removing the naming contexts that there will be more than one. So for example:

DC=domain,DC=net

DC=DomainDnsZones,DC=domain,DC=net Its the DNS Zone that people tend to forget !!

If you get an error about Leaf Objects you havent removed all the Naming Contexts. You also need to ensure you are connected to the Domain Naming Master to perform the actual Domain Removal.

———————————————————————————————————————————————————————

Please ensure you are 100% certain you want to do the below, and dont do in a production environment without testing first

Below are some screenshots and bullet points on the end to end process:

With NTDSUTIL all of the commands can be abbreviated as long as they are unique, I have put some in brackets next to the full command.

First of all Connect to the Domain Naming Master

```txt
Connect using NTDSUTIL

1. Start up NTDSUTIL from a command prompt
Go into the Metadata Cleanup (M C for short)
2. Metadata Cleanup

Go into connections

3. Connections

Connect to the Domain Naming FSMO holder for your forest

4. Connect to <Server> 
```

![remove_domain_1](/assets/img/blog/AD/Remove_Domain_1.png)
   
First of all remove any Domain Controllers from the Domain you wish to remove.
```txt
Quit Connections (Q)

1. Quit

Select the object you want to remove by using Select Operations Target ( S O T for short)

2. Select Operations Target

List the Domains in your Forest

3. List Domains

Connect to Domain you wish to remove

4. Select Domain <number>

List the Sites in your Forest and Select the Site which contains the first (or only) domain controller you wish to remove

5. Select Site <number> 
```

![remove_domain_2](/assets/img/blog/AD/Remove_Domain_2.png)

List Domain Controllers in the site you connected to above
```txt
1. List Servers in Site

Select Domain Controller you want to remove

2. Select Server <number> 
```

![remove_domain_3](/assets/img/blog/AD/Remove_Domain_3.png)

You are ready to remove the Domain Controllers
```txt
1. Quit

Remove the Domain Controller

2. Remove Selected Server

3. Select Yes on the pop up window 
```

![remove_domain_4](/assets/img/blog/AD/Remove_Domain_4.png)

```txt
4. Select Yes on the pop up windows 
```

![remove_domain_5](/assets/img/blog/AD/Remove_Domain_5.png)

```txt
5. You will get back a message saying the Domain Controller has been removed. 
```

![remove_domain_6](/assets/img/blog/AD/Remove_Domain_6.png)

Then you need to remove the naming contexts for the Domain you wish to remove.

Move back to the objects you can select to select the Naming Context you want to remove

```txt
1. S O T

List the naming contexts for your Forest

2. List Naming Contexts

Select the Naming Context you wish to remove

3. Select Naming Context <number> 
```

![remove_domain_7](/assets/img/blog/AD/Remove_Domain_7.png)

Then quit back to remove the Naming Context
```txt
1. Quit

2. Remove Selected Naming Context

3. Select yes to remove the naming Context 
```

![remove_domain_8](/assets/img/blog/AD/Remove_Domain_8.png)

```txt
4. You will get back a message saying the Naming Context has been removed. 
```

![remove_domain_9](/assets/img/blog/AD/Remove_Domain_9.png)
Repeat the above steps for all Domain Controllers and Naming Contexts for the Domain you wish to remove.

Next you need to remove the Domain itself !!PLEASE TAKE NOTE OF THE MESSAGE !!!
```txt
1. Remove Selected Domain 
```
![remove_domain_10](/assets/img/blog/AD/Remove_Domain_10.png)
And thats it .. should be all gone.. 