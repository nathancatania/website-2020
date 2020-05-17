---
title: "Unlock or change the super password for Junos Space 17.2, 18.X"
date: 2018-02-19
description: "In the 17.2R1 release for Junos Space, the process for unlocking or changing the password for the super account has changed."
#cover: banner.png
tags: [juniper, guides]
comments: false
toc: false
---

## Introduction
For Junos Space and its associated applications, if you have forgotten the password to the __super__ account (or have locked yourself out of the account), you can reset the password and/or unlock the account by modifying the super user's entry in the `build_db` DB of MySQL; via the Unix CLI in Space's "debug" mode.

However in release 17.2R1, the default password of the __jboss__ user (which is required in order to access the MySQL CLI in the first place) has changed. ~~As far as I can tell, Juniper has not documented this anywhere.~~

> Update July 2018:
> - Juniper have now documented this changed in a [KB article][KB] (J-net login required).
> - This process is will also work with Junos Space releases 18.1 and 18.2.<br>


Previously, you would access the MySQL CLI using the user __jboss__ and the password __netscreenos__:

```shell
[root@space-005056a51a72 ~]# mysql -u jboss -pnetscreenos build_db
...
...
Welcome to the MySQL monitor.  Commands end with ; or \g.
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.
mysql>
```

In Space 17.2R1 (and above), the same credentials do not work:

```shell
[root@space-005056a51a72 ~]# mysql -u jboss -pnetscreenos build_db
ERROR 1045 (28000): Access denied for user 'jboss'@'localhost' (using password: YES)
```

So if you've locked your __super__ user account or forgotten its password, what do you do now?

## New SQL password
It seems as of 17.2, during installation, Space will automatically generate a random password for the __jboss__ user of the MySQL database.

Lucky for us (?), Space also stores this generated password in plaintext (!! 😱) for us to reference.

The passwords are located at: `/etc/sysconfig/JunosSpace/pwd`

Dumping them to screen, we can copy the password for the __jboss__ user:

```shell
[root@space-005056a51a72 ~]# cat /etc/sysconfig/JunosSpace/pwd

mysql.repUser=-BkGJCdQSEktjxIVRlF4rBOx9g3LHgCM
mysql.repAdmin=p8AGI_-jAbsNlKq4cnnSVb0rh1aeOLO-
postgres.postgres=postgres
postgres.opennms=opennms
postgres.replication=replication
cassandra.jboss=netscreen
jboss.admin=LJrZaVdj++++50395922
mysql.root=BCiIAlkj5_NZj83VQpuX4E2oSRBHTlWA
mysql.jboss=AEYZIMccv0Uq8ct_WoSgbo0BB4Wwu8RH
```

You want the `mysql.jboss=` string, which in the example above is: `AEYZIMccv0Uq8ct_WoSgbo0BB4Wwu8RH`

__This is your password to authenticate the MySQL jboss user!__

You can now access the MySQL CLI:

```shell
[root@space-005056a51a72 ~]# mysql -u jboss -p<YOU-JBOSS-PASSWORD> build_db
...
...
Welcome to the MySQL monitor.  Commands end with ; or \g.
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.
mysql>
```

Replace the `<YOUR-JBOSS-PASSWORD>` with the string retrieved above. You will now have access to the CLI.

## Fixing the super account
Now that we have the password to access our MySQL CLI, we can finally unlock or reset the __super__ account to regain access to our Space applications.


### Unlocking the super account
If the super account is __locked__, you can unlock it from the MySQL CLI. Enter the following two DB queries/commands at the `mysql>` shell prompt:

```shell
select * from USER_IP_ADDRESS where user_id in (select id from USER where name LIKE '%super%')\G
```

```shell
DELETE FROM USER_IP_ADDRESS where user_id in (select id from USER where name LIKE '%super%');
```

The super account should now be unlocked.

### Changing the super account password
If you have forgotten the password to the super account, you can reset it by altering the database entry for the 'super' user. The password field takes an encrypted value. __We will reset the password to the default value `juniper123`__. This will allow you to login and then change it to something more secure through the GUI.

To login to the DB and change the password in one command:

```shell
mysql -ujboss -p$(grep mysql.jboss /etc/sysconfig/JunosSpace/pwd | awk -F= '{print $2}') build_db -e "UPDATE USER set password='ok89Nva6qHxytSHsP8AeLg==' where name='super'"
```

This fetches the jboss password from the `pwd` file and executes the SQL statement to change the super user's password to `ok89Nva6qHxytSHsP8AeLg==` which represents `juniper123`


### Finish
You should now be able to login to the Junos Space GUI using the username `super` and password `juniper123`.

__Make sure to change the super password immediately to something more secure!__

[KB]:https://kb.juniper.net/InfoCenter/index?page=content&id=KB17582
