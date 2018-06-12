# Samba Client Configuration

<hr><hr>

<a name="client">
### Install 'CIFS-utils', and the Samba client package:
</a>

```
[root@vm07 ~]# yum -y install cifs-utils samba-client
```

### Create a mount point for the Samba share:

`[root@vm07 ~]# mkdir /smbmount`

### Mount the SMB share via the /etc/fstab file:

`[root@vm07 ~]# vim /etc/fstab`

```
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

//10.0.2.6/samba  /smbmount    cifs    username=smbuser,passsword=#######   0 0

```

### Notice the peculiarities of a CIFS based mount, we are not adding the name of the directory on the server.  The Samba share is literally identified by the name 'Samba'.  In this case, we are putting the Samba user's password directly into fstab.  Alternatively, we could create a credentials file with this information.

### Mount the Samba share, to confirm this is working:

```
[root@vm07 ~]# mount -a
[root@vm07 ~]# cd /smbmount
[root@vm07 smbmount]# touch testfile
```

### Back on the server, we can confirm that a file has been created in the shared directory:

```
[root@vm07 smbmount]# ssh masterhost06
root@masterhost06's password:
Last login: Sat May 26 10:11:44 2018 from vm05.jdillon.local
[root@masterhost06 ~]# ls -l /smbshare
total 0
-rw-r--r--. 1 smbuser smbuser 0 May 27 13:27 testfile
```

### We see that the file has been created in the shared directory, as the Samba user 'smbuser'

### Next, we will set up the Samba server specifically for [group collaboration](Samba_Server_Config#group)

<hr><hr>

<a name="setgid"></a>

### Having established our setgid settings for the Samba share on the client, and added two Samba users we are now ready to test group collaboration

### We will unmount the directory, and remount (via fstab) as one of our newly created Samba users:

```
[root@vm07 /]# umount /smbmount
[root@vm07 /]# vim /etc/fstab
```

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

//10.0.2.6/samba   /smbmount  cifs  username=sambauser1,password=######   0 0

```

### Confirming successful mount:

```
[root@vm07 /]# mount -a
[root@vm07 /]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  8.0G  3.8G  4.3G  47% /
devtmpfs                 903M     0  903M   0% /dev
tmpfs                    920M     0  920M   0% /dev/shm
tmpfs                    920M  9.1M  911M   1% /run
tmpfs                    920M     0  920M   0% /sys/fs/cgroup
/dev/sda1               1014M  168M  847M  17% /boot
10.0.2.6:/nfs            8.0G  3.8G  4.3G  48% /krb_nfs
tmpfs                    184M   12K  184M   1% /run/user/42
tmpfs                    184M     0  184M   0% /run/user/0
10.0.2.6:/share          8.0G  3.8G  4.3G  48% /nfs
//10.0.2.6/samba         8.0G  3.8G  4.3G  48% /smbmount
```

### Create a new file in the Samba shared directory, and check specifications:

```
[root@vm07 /]# cd /smbmount
[root@vm07 smbmount]# ls
testfile
[root@vm07 smbmount]# touch newfile
[root@vm07 smbmount]# ls -ld
drwxrwsrwx. 2 root 1005 0 May 27 14:13 .
[root@vm07 smbmount]# ls -l
total 0
-rw-r--r--. 1    1004    1005 0 May 27 14:13 newfile
-rw-r--r--. 1 smbuser smbuser 0 May 27 13:27 testfile
```

### We see that our new file has been created under the group identity 1005 (=smbgroup on the server)

```
[root@vm07 smbmount]# ssh masterhost06
root@masterhost06's password:
Last login: Sun May 27 13:28:27 2018 from vm07.jdillon.local
[root@masterhost06 ~]# grep smbgroup /etc/group
smbgroup:x:1005:sambauser1,sambauser2
```

### We have confirmed that setgid functionality is working for our Samba share.
