# NFS Client Configuration:
(Client installed on vm07 (10.0.2.7))<br>
(Server installed on masterhost06 (10.0.2.6))
<hr><hr>

### Install the nfs-client package:

`[root@vm07 ~] yum install -y nfs-utils`

### Start and enable the client service:

```
[root@vm07 ~] systemctl start nfs
[root@vm07 ~] systemctl enable nfs
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
```

### Check available nfs shares on the server:

```
[root@vm07 ~] showmount -e 10.0.2.6
rpc mount export: RPC: Unable to receive; errno = No route to host
```

### If the command fails (as above) we can add port 20048 to the public zone of firewalld (also make sure that the server's nfs configuration file (/etc/sysconfig/nfs) has the following lines uncommented):

 >>>  **MOUNTD_PORT=892**<br>
 >>>   **STATD_PORT=662**

```
[root@vm07 ~] firewall-cmd --zone=public --add-port=20048/tcp --permanent
success
[root@vm07 ~] firewall-cmd --reload
success
[root@vm07 ~] showmount -e 10.0.2.6
Export list for 10.0.2.6:
/share *
```

### Mount the shared directory from server, via fstab configuration:

`[root@vm07 ~] vim /etc/fstab`

```
#
# /etc/fstab
# Created by anaconda on Tue May 15 13:13:02 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=fe46a3d6-e5e9-4db5-94cf-3a09d51db785 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0

10.0.2.6:/share			  /nfs		nfs		defaults    0  0

```

### Create the mount point added to /etc/fstab ('/nfs'):

`[root@vm07 ~] mkdir /nfs`

### Mount all available mount points (now including /nfs):

`[root@vm07 ~] mount -a`

### Confirm our nfs mounted directory has been added:

```
df -h
[root@vm07 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  8.0G  3.8G  4.3G  47% /
devtmpfs                 903M     0  903M   0% /dev
tmpfs                    920M     0  920M   0% /dev/shm
tmpfs                    920M  9.1M  911M   1% /run
tmpfs                    920M     0  920M   0% /sys/fs/cgroup
/dev/sda1               1014M  168M  847M  17% /boot
tmpfs                    184M   12K  184M   1% /run/user/42
tmpfs                    184M     0  184M   0% /run/user/0
10.0.2.6:/share          8.0G  3.8G  4.3G  47% /nfs
```

## Kerberized NFS Authentication:

### This section assumes that you have worked through the steps highlighted in 'NFS\_Server_Config', up to the section alerting '<span style="color:red;background:black;font-family:Courier;">!Refer to client configuration page</span>'

(If not, please refer to the page and set up set up the Kerberos server as specified).
<hr>

### <a name="request">Request a new Kerberos ticket</a> (for the host), with an NFS principle:

#### (If arriving here from the server config page, please make sure to set up client by following steps from the start of **this** page up to this point)

```
[root@vm07 ~]# kinit -k -t /etc/krb5.keytab nfs/vm07.jdillon.local
```

### Confirm that a ticket has been granted correctly:

```
[root@vm07 ~]# klist
Ticket cache: KEYRING:persistent:0:krb_ccache_4XEpYhW
Default principal: nfs/vm07.jdillon.local@JDILLON.LOCAL

Valid starting       Expires              Service principal
05/19/2018 15:05:45  05/20/2018 15:05:45  krbtgt/JDILLON.LOCAL@JDILLON.LOCAL
```

## <a name="restart"> Restart the NFS-client service</a>

```
[root@vm07 ~]# systemctl restart nfs-client.target
[root@vm07 ~]# systemctl enable nfs-client.target
```

### Check available exports from the Kerberos/NFS server:

```
Export list for masterhost06:
/nfs   *
/share *
```

### Mount /nfs on a newly created mount point (/krb_nfs):

`[root@vm07 ~]# mkdir /krb_nfs`

### Mount the directory via /etc/fstab:

`[root@vm07 ~]# vim /etc/fstab`

```
#
# /etc/fstab
# Created by anaconda on Tue May 15 13:13:02 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=fe46a3d6-e5e9-4db5-94cf-3a09d51db785 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0

10.0.2.6:/share                /nfs        nfs            defaults            0 0

10.0.2.6:/nfs                  /krb_nfs    nfs            sec=krb5p  0 0

```

#### We see our standard NFS mounted directory from the server (/share mounted locally on /nfs), and the server's /nfs directory requiring Kerberos authentication mounted on the client's /krb_nfs directory.  This will be utilizing the most secure Kerberos enforcement 'krb5p'.

### Confirm the Kerberized NFS share is mounting correctly:

```
[root@vm07 ~]# mount -a
[root@vm07 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  8.0G  3.8G  4.3G  47% /
devtmpfs                 903M     0  903M   0% /dev
tmpfs                    920M     0  920M   0% /dev/shm
tmpfs                    920M  9.2M  911M   1% /run
tmpfs                    920M     0  920M   0% /sys/fs/cgroup
/dev/sda1               1014M  168M  847M  17% /boot
tmpfs                    184M   12K  184M   1% /run/user/42
10.0.2.6:/share          8.0G  3.8G  4.3G  48% /nfs
tmpfs                    184M     0  184M   0% /run/user/0
10.0.2.6:/nfs            8.0G  3.8G  4.3G  48% /krb_nfs
```

### Confirming the share has worked, we must create a KDC user principle to allow user access to the shared directory

### Refer to server configuration page/section [NFS-user-principle](NFS_Server_Config.md#user)
<hr><hr>

### Having added a <a name="principle">principle for user 'jdillon'</a> on the server, we will request a new Kerberos ticket as the user and attempt to access the Kerberized NFS share:

```
[root@vm07 ~]# su jdillon
[jdillon@vm07 root]$ klist
klist: Credentials cache keyring 'persistent:1000:1000' not found
[jdillon@vm07 root]$ ls /krb*
ls: cannot access /krb_nfs: Permission denied
[jdillon@vm07 root]$ kinit
Password for jdillon@JDILLON.LOCAL:
[jdillon@vm07 root]$ klist
Ticket cache: KEYRING:persistent:1000:1000
Default principal: jdillon@JDILLON.LOCAL

Valid starting       Expires              Service principal
05/19/2018 16:02:29  05/20/2018 16:02:29  krbtgt/JDILLON.LOCAL@JDILLON.LOCAL
[jdillon@vm07 root]$ ls /krb*
[jdillon@vm07 root]$ cd /krb*
[jdillon@vm07 krb_nfs]$ pwd
/krb_nfs
[jdillon@vm07 krb_nfs]$ echo "ummmm...winning.."
ummmm...winning..
```
<hr><hr>

## Utlizing a <a name="setgid">Collaborative NFS Share:</a>

### Confirm shares on the server:

```
[root@vm07 ~]# showmount -e masterhost06
Export list for masterhost06:
/nfs   *
/share *
``` 

### Create the nfs-group matching the server:

`[root@vm07 ~]# groupadd nfsgroup`

### Unmount and remount the server's /share directory mounted on /nfs:

```
[root@vm07 ~]# umount /nfs
[root@vm07 ~]# mount -a
[root@vm07 ~]# ls -ld /nfs
drwxrws---. 2 root nfsgroup 6 May 19 22:16 /nfs
```

### We see that the client's setgid is being honored on the server

### We can test this, by adding a new directory the /nfs and checking its group setting:

```
[root@vm07 ~]# ls -la /nfs
total 0
drwxrws---.  3 root nfsgroup  21 May 26 10:43 .
dr-xr-xr-x. 19 root root     250 May 19 15:39 ..
drwxr-sr-x.  2 root nfsgroup   6 May 26 10:43 testdir
```

### We have confirmed that anything created within the /nfs directory, will be added under the nfsgroup.

