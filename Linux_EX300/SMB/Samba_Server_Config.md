# Samba Server Configuration 

<hr><hr>

#### (Please be sure to set up the server, before attempting client configuration)

### Install necessary Samba server packages

```
[root@masterhost06 ~]# yum search samba
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.grid.uchicago.edu
 * extras: repos.forethought.net
 * updates: centos.s.uw.edu
================================= N/S matched: samba ==================================
kdenetwork-fileshare-samba.x86_64 : Share files via samba
pcp-pmda-samba.x86_64 : Performance Co-Pilot (PCP) metrics for Samba
samba-client.x86_64 : Samba client programs
samba-client-libs.i686 : Samba client libraries
samba-client-libs.x86_64 : Samba client libraries
samba-common.noarch : Files used by both Samba servers and clients
samba-common-libs.x86_64 : Libraries used by both Samba servers and clients
samba-common-tools.x86_64 : Tools for Samba servers and clients
samba-dc.x86_64 : Samba AD Domain Controller
samba-dc-libs.x86_64 : Samba AD Domain Controller Libraries
samba-devel.i686 : Developer tools for Samba libraries
samba-devel.x86_64 : Developer tools for Samba libraries
samba-krb5-printing.x86_64 : Samba CUPS backend for printing with Kerberos
samba-libs.i686 : Samba libraries
samba-libs.x86_64 : Samba libraries
samba-python.x86_64 : Samba Python libraries
samba-python-test.x86_64 : Samba Python libraries
samba-test.x86_64 : Testing tools for Samba servers and clients
samba-test-libs.i686 : Libraries need by the testing tools for Samba servers and
                     : clients
samba-test-libs.x86_64 : Libraries need by the testing tools for Samba servers and
                       : clients
samba-vfs-glusterfs.x86_64 : Samba VFS module for GlusterFS
samba-winbind.x86_64 : Samba winbind
samba-winbind-clients.x86_64 : Samba winbind clients
samba-winbind-krb5-locator.x86_64 : Samba winbind krb5 locator
samba-winbind-modules.i686 : Samba winbind modules
samba-winbind-modules.x86_64 : Samba winbind modules
ctdb.x86_64 : A Clustered Database based on Samba's Trivial Database (TDB)
python-smbc.x86_64 : Python bindings for libsmbclient API from Samba
samba.x86_64 : Server and Client software to interoperate with Windows machines
samba-pidl.noarch : Perl IDL compiler
```

### We will need both samba-client and samba.x86

```
[root@masterhost06 ~]# yum install -y samba samba-client
```

### Start and enable the smb and nmb services:

```
[root@masterhost06 ~]# systemctl start smb nmb
[root@masterhost06 ~]# systemctl enable smb nmb
Created symlink from /etc/systemd/system/multi-user.target.wants/smb.service to /usr/lib/systemd/system/smb.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/nmb.service to /usr/lib/systemd/system/nmb.service.
```

### Allow Samba through the firewall:

```
[root@masterhost06 ~]# firewall-cmd --permanent --add-service=samba
success
[root@masterhost06 ~]# firewall-cmd --reload
success
[root@masterhost06 ~]#
```

### Specify/create a non-login user, who will have access to the Samba share on the client machine:

```
[root@masterhost06 ~]# useradd smbuser -s /sbin/nologin
```

### Create a Samba password for the new user:

```
[root@masterhost06 ~]# smbpasswd -a smbuser
New SMB password:
Retype new SMB password:
Added user smbuser.
```

### Create our shared Samba directory:

```
[root@masterhost06 ~]# mkdir /smbshare
```

### We are now ready to edit the Samba configuration file (/etc/samba/smb.conf).<br>  There is very well documented example version of this file (/etc/samba/smb.conf.example) that includes examples of SELinux configuration parameters as well as many configuration options.

### We will be using the SELinux booleans, to alter our current SELinux settings, as well as adding the following lines to the smb.conf file (in the 'Share Definitions' section):

```
[samba]
comment = samba-share
path = /smbshare
public = yes
writable = yes
```

`[root@masterhost06 ~]# vim /etc/samba/smb.conf`

```
# See smb.conf.example for a more detailed config file or
# read the smb.conf manpage.
# Run 'testparm' to verify the config is correct after
# you modified it.

[global]
        workgroup = SAMBA
        security = user

        passdb backend = tdbsam

        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw

[homes]
        comment = Home Directories
        valid users = %S, %D%w%S
        browseable = No
        read only = No
        inherit acls = Yes

[printers]
        comment = All Printers
        path = /var/tmp
        printable = Yes
        create mask = 0600
        browseable = No

[print$]
        comment = Printer Drivers
        path = /var/lib/samba/drivers
        write list = @printadmin root
        force group = @printadmin
        create mask = 0664
        directory mask = 0775

[samba]
        comment = samba-share
        path = /smbshare
        public = yes
        writable = yes
```

### Restart the service:

`[root@masterhost06 ~]# systemctl restart smb nmb`

### We can test the configuration parameters we have set up, using the 'testparm' command:

```
[root@masterhost06 ~]# testparm
Load smb config files from /etc/samba/smb.conf
rlimit_max: increasing rlimit_max (1024) to minimum Windows limit (16384)
Processing section "[homes]"
Processing section "[printers]"
Processing section "[print$]"
Processing section "[samba]"
Loaded services file OK.
Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions

# Global parameters
[global]
	printcap name = cups
	security = USER
	workgroup = SAMBA
	idmap config * : backend = tdb
	cups options = raw


[homes]
	browseable = No
	comment = Home Directories
	inherit acls = Yes
	read only = No
	valid users = %S %D%w%S


[printers]
	browseable = No
	comment = All Printers
	create mask = 0600
	path = /var/tmp
	printable = Yes


[print$]
	comment = Printer Drivers
	create mask = 0664
	directory mask = 0775
	force group = @printadmin
	path = /var/lib/samba/drivers
	write list = @printadmin root


[samba]
	comment = samba-share
	guest ok = Yes
	path = /smbshare
	read only = No
```

### We are primarily interested in the line 'Loaded services file OK.', indicating a successful file configuration

### The share can be tested by running the 'smbclient' package locally:

```
[root@masterhost06 ~]# smbclient -L localhost
Enter SAMBA\root's password:
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	samba           Disk      samba-share
	IPC$            IPC       IPC Service (Samba 4.7.1)
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	SAMBA                MASTERHOST06
```

### Finally, we must adjust our SELinux settings, to work with the Samba share:

```
[root@masterhost06 ~]# semanage fcontext -a -t samba_share_t "/smbshare(/.*)?"
[root@masterhost06 ~]# restorecon -Rv /smbshare
restorecon reset /smbshare context unconfined_u:object_r:default_t:s0->unconfined_u:object_r:samba_share_t:s0
[root@masterhost06 ~]# ls -lZd /smbshare
drwxr-xr-x. root root unconfined_u:object_r:samba_share_t:s0 /smbshare
[root@masterhost06 ~]# ls -ld /smbshare
drwxrwxrwx. 2 root root 6 May 27 11:57 /smbshare
```

### We are now ready to mount the Samba share on the [client machine](Samba_Client_Config#client)

## Samba Server Configuration for <a name="group">Group Collaboration</a>

<hr><hr>

### We will create a Samba group identity (smbgroup) along with some new Samba users, and use setgid to control file creation

```
[root@masterhost06 smbshare]# groupadd smbgroup
[root@masterhost06 smbshare]# useradd sambauser1 -s /sbin/nologin
[root@masterhost06 smbshare]# useradd sambauser2 -s /sbin/nologin
```

### Create Samba passwords for these new users:

```
[root@masterhost06 smbshare]# smbpasswd -a sambauser1
New SMB password:
Retype new SMB password:
Added user sambauser1.
[root@masterhost06 smbshare]# smbpasswd -a sambauser2
New SMB password:
Retype new SMB password:
Added user sambauser2
```

### Add the users to the Samba group:

```
[root@masterhost06 smbshare]# usermod -aG smbgroup sambauser1
[root@masterhost06 smbshare]# usermod -aG smbgroup sambauser2
```

### Query the share name of our Samba share:

```
[root@masterhost06 smbshare]# smbclient -L localhost
Enter MYGROUP\root's password:
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	samba           Disk      samba-share
	IPC$            IPC       IPC Service (Samba Server Version 4.7.1)
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	MYGROUP              MASTERHOST06
```

### Change the group identity of the Samba shared directory /smbshare to our Samba user group 'smbgroup':

```
[root@masterhost06 smbshare]# chgrp smbgroup /smbshare/
```

### Add a setgid element to this directory:

`[root@masterhost06 smbshare]# chmod 2777 /smbshare`

### Add our new user group to a definition in the Samba configuration file (/etc/samba/smb.conf):

`[root@masterhost06 smbshare]# vim /etc/samba/smb.conf`

```
        [samba]
        comment = samba-share
        path = /smbshare
        public = yes
        writable = yes
        write list = @smbgroup
```

### Restart smb and nmb services:

`[root@masterhost06 smbshare]# systemctl restart smb nmb`

### We are now ready to test the group share, by accessing as one of the users [on our client](Samba_Client_Config#setgid)
