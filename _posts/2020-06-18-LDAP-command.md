---
title: LDAP Command
tags: LDAP
---
<!--more-->

# add-user.ldif
```txt
dn: uid=ithelpdesk,ou=People,dc=example,dc=com
uid: ithelpdesk
objectClass: top
objectClass: account
objectClass: posixaccount
objectClass: inetOrgPerson
objectClass: person
objectClass: inetUser
objectClass: organizationalPerson
uidNumber: 1025
gidNumber: 101
homeDirectory: /home/ithelpdesk
loginShell: /bin/bash
userPassword: 123456
sn: IT
givenname: Helpdesk
cn: IT Helpdesk
l: Hangzhou
mail: ithelpdesk@example.com
description: ithelpdesk
```

```bash
~]# ldapadd -f add-user.ldif -h dsserver -p 389 -D "cn=Directory Manager" -x -W
Enter LDAP Password:
adding new entry "uid=ithelpdesk,ou=People,dc=example,dc=com"
```

# add-group.ldif
```txt
dn: cn=users,ou=Groups,dc=example,dc=com
objectClass: top
objectClass: posixGroup
objectClass: groupOfUniqueNames
gidNumber:   101
cn: users
```

```bash
~]# ldapadd -f add-group.ldif -h dsserver -p 389 -D "cn=Directory Manager" -x -W
Enter LDAP Password:
adding new entry "cn=users,ou=Groups,dc=example,dc=com"
```

# ldapadd ldapmodify ldapdelete
交互式操作

**ldapmodify**
```bash
# ldapmodify -D "cn=Directory Manager" -W -p 389 -h server.example.com -x
 
dn: uid=user,ou=people,dc=example,dc=com
changetype: modify
delete: telephoneNumber
-
add: manager
manager: cn=manager_name,ou=people,dc=example,dc=com
^D
 
 
^D 或者 按两次enter 表示退出交互式命令
```