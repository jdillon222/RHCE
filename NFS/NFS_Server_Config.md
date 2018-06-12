# NFS Server Configuration:

(Server installed on masterhost06 (10.0.2.6))<br>
(Client installed on vm07 (10.0.2.7))
<hr><hr>

### Yum package 'nfs-utils' is the main nfs server requirement:

`[root@masterhost06 ~] yum install -y nfs-utils`

### Create our main nfs sharing directory (for exporting):

`[root@masterhost06 ~] mkdir /share`

### Edit the main nfs export configuration file:

`[root@masterhost06 ~] vim /etc/exports`

```
/share				*(rw, no_root_squash)
```

### Allow nfs and rpcbind through the firewall:

```
[root@masterhost06 ~] firewall-cmd --permanent --add-service=nfs
success
[root@masterhost06 ~] firewall-cmd --permanent --add-service=rpc-bind
success
[root@masterhost06 ~] firewall-cmd --reload
success
```

### Configure mount and status ports within /etc/sysconfig/nfs, by uncommenting the following lines (this will primarily allow clients to scan the server for available nfs mounts):

`[root@masterhost06 ~]# vim /etc/sysconfig/nfs`

```
MOUNTD_PORT=892
STATD_PORT=662
```

`[root@masterhost06 ~]# systemctl restart nfs-server`

### Open these ports through the firewall (both tcp and udp):

```
[root@masterhost06 ~]firewall-cmd --zone=public --add-port=892/udp --permanent
success
[root@masterhost06 ~]firewall-cmd --zone=public --add-port=892/tcp --permanent
success
[root@masterhost06 ~]firewall-cmd --zone=public --add-port=662/udp --permanent
success
[root@masterhost06 ~]firewall-cmd --zone=public --add-port=662/tcp --permanent
success
[root@masterhost06 ~]firewall-cmd --reload

```
### Start and enable nfs-server and rpc-bind:

```
[root@masterhost06 ~] systemctl start nfs-server rpcbind
[root@masterhost06 ~] systemctl enable nfs-server rpcbind
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
```
### Show currently shared nfs-directories:

```
[root@masterhost06 ~] exportfs -avr
exporting *:/share
[root@masterhost06 ~] showmount -e localhost
Export list for localhost:
/share *
```

## Configuring SELinux with NFS

### Show current SELinux settings for shared directory:

```
[root@masterhost06 ~]# ls -lZd /share
dr-xr-x---. root root unconfined_u:object_r:default_t:s0 /share
```

### The directory is currently working with the default context, we would like to change this via semanage:

```
[root@masterhost06 ~]# semanage fcontext -a -t public_content_rw_t "/share(/.*)?"
[root@masterhost06 ~]# restorecon -Rv /share
restorecon reset /share context unconfined_u:object_r:default_t:s0->unconfined_u:object_r:public_content_rw_t:s0
restorecon reset /share/test context unconfined_u:object_r:default_t:s0->unconfined_u:object_r:public_content_rw_t:s0
```

### Confirm that our new SELinux settings have taken effect for the directory:

```
[root@masterhost06 ~]# ls -lZd /share
drwxr-xr-x. root root unconfined_u:object_r:public_content_rw_t:s0 /share
```

### We must now adjust the boolean settings pertaining to NFS in SELinux.  Querying possible boolean values pertaining to NFS:

```
[root@masterhost06 ~]# semanage boolean -l | grep nfs
xen_use_nfs                    (off  ,  off)  Allow xen to use nfs
mpd_use_nfs                    (off  ,  off)  Allow mpd to use nfs
virt_use_nfs                   (off  ,  off)  Allow virt to use nfs
use_nfs_home_dirs              (off  ,  off)  Allow use to nfs home dirs
ksmtuned_use_nfs               (off  ,  off)  Allow ksmtuned to use nfs
nfsd_anon_write                (off  ,  off)  Allow nfsd to anon write
git_system_use_nfs             (off  ,  off)  Allow git to system use nfs
git_cgi_use_nfs                (off  ,  off)  Allow git to cgi use nfs
logrotate_use_nfs              (off  ,  off)  Allow logrotate to use nfs
conman_use_nfs                 (off  ,  off)  Allow conman to use nfs
cobbler_use_nfs                (off  ,  off)  Allow cobbler to use nfs
httpd_use_nfs                  (off  ,  off)  Allow httpd to use nfs
sge_use_nfs                    (off  ,  off)  Allow sge to use nfs
sanlock_use_nfs                (off  ,  off)  Allow sanlock to use nfs
samba_share_nfs                (off  ,  off)  Allow samba to share nfs
nagios_use_nfs                 (off  ,  off)  Allow nagios to use nfs
ftpd_use_nfs                   (off  ,  off)  Allow ftpd to use nfs
openshift_use_nfs              (off  ,  off)  Allow openshift to use nfs
polipo_use_nfs                 (off  ,  off)  Allow polipo to use nfs
tmpreaper_use_nfs              (off  ,  off)  Allow tmpreaper to use nfs
nfs_export_all_rw              (on   ,   on)  Allow nfs to export all rw
nfs_export_all_ro              (on   ,   on)  Allow nfs to export all ro
```

### If we would like to change SELinux boolean settings directly, we could always use commands such as:

```
setsebool -P nfs_export_all_rw off
[root@masterhost06 ~]# semanage boolean -l | grep nfs
xen_use_nfs                    (off  ,  off)  Allow xen to use nfs
mpd_use_nfs                    (off  ,  off)  Allow mpd to use nfs
virt_use_nfs                   (off  ,  off)  Allow virt to use nfs
use_nfs_home_dirs              (off  ,  off)  Allow use to nfs home dirs
ksmtuned_use_nfs               (off  ,  off)  Allow ksmtuned to use nfs
nfsd_anon_write                (off  ,  off)  Allow nfsd to anon write
git_system_use_nfs             (off  ,  off)  Allow git to system use nfs
git_cgi_use_nfs                (off  ,  off)  Allow git to cgi use nfs
logrotate_use_nfs              (off  ,  off)  Allow logrotate to use nfs
conman_use_nfs                 (off  ,  off)  Allow conman to use nfs
cobbler_use_nfs                (off  ,  off)  Allow cobbler to use nfs
httpd_use_nfs                  (off  ,  off)  Allow httpd to use nfs
sge_use_nfs                    (off  ,  off)  Allow sge to use nfs
sanlock_use_nfs                (off  ,  off)  Allow sanlock to use nfs
samba_share_nfs                (off  ,  off)  Allow samba to share nfs
nagios_use_nfs                 (off  ,  off)  Allow nagios to use nfs
ftpd_use_nfs                   (off  ,  off)  Allow ftpd to use nfs
openshift_use_nfs              (off  ,  off)  Allow openshift to use nfs
polipo_use_nfs                 (off  ,  off)  Allow polipo to use nfs
tmpreaper_use_nfs              (off  ,  off)  Allow tmpreaper to use nfs
nfs_export_all_rw              (off  ,  off)  Allow nfs to export all rw
nfs_export_all_ro              (on   ,   on)  Allow nfs to export all ro
[root@masterhost06 ~]# setsebool -P nfs_export_all_rw on
[root@masterhost06 ~]# semanage boolean -l | grep nfs
xen_use_nfs                    (off  ,  off)  Allow xen to use nfs
mpd_use_nfs                    (off  ,  off)  Allow mpd to use nfs
virt_use_nfs                   (off  ,  off)  Allow virt to use nfs
use_nfs_home_dirs              (off  ,  off)  Allow use to nfs home dirs
ksmtuned_use_nfs               (off  ,  off)  Allow ksmtuned to use nfs
nfsd_anon_write                (off  ,  off)  Allow nfsd to anon write
git_system_use_nfs             (off  ,  off)  Allow git to system use nfs
git_cgi_use_nfs                (off  ,  off)  Allow git to cgi use nfs
logrotate_use_nfs              (off  ,  off)  Allow logrotate to use nfs
conman_use_nfs                 (off  ,  off)  Allow conman to use nfs
cobbler_use_nfs                (off  ,  off)  Allow cobbler to use nfs
httpd_use_nfs                  (off  ,  off)  Allow httpd to use nfs
sge_use_nfs                    (off  ,  off)  Allow sge to use nfs
sanlock_use_nfs                (off  ,  off)  Allow sanlock to use nfs
samba_share_nfs                (off  ,  off)  Allow samba to share nfs
nagios_use_nfs                 (off  ,  off)  Allow nagios to use nfs
ftpd_use_nfs                   (off  ,  off)  Allow ftpd to use nfs
openshift_use_nfs              (off  ,  off)  Allow openshift to use nfs
polipo_use_nfs                 (off  ,  off)  Allow polipo to use nfs
tmpreaper_use_nfs              (off  ,  off)  Allow tmpreaper to use nfs
nfs_export_all_rw              (on   ,   on)  Allow nfs to export all rw
nfs_export_all_ro              (on   ,   on)  Allow nfs to export all ro
```

### Create KDC principals for server via Kadmin interface:

```
[root@masterhost06 ~]# kadmin.local
Authenticating as principal root/admin@JDILLON.LOCAL with password.
```

```
kadmin.local:  addprinc host/masterhost06.jdillon.local
WARNING: no policy specified for host/masterhost06.jdillon.local@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "host/masterhost06.jdillon.local@JDILLON.LOCAL": #####
Re-enter password for principal "host/masterhost06.jdillon.local@JDILLON.LOCAL": #####
add_principal: Principal or policy already exists while creating "host/masterhost06.jdillon.local@JDILLON.LOCAL".

```

### Create the NFS principal for the server:

```
kadmin.local:  addprinc nfs/masterhost06.jdillon.local
WARNING: no policy specified for nfs/masterhost06.jdillon.local@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "nfs/masterhost06.jdillon.local@JDILLON.LOCAL":
Re-enter password for principal "nfs/masterhost06.jdillon.local@JDILLON.LOCAL":
Principal "nfs/masterhost06.jdillon.local@JDILLON.LOCAL" created.
```

### Create a local copy of the keytab file for the server and NFS (keytab file holds all service principals, to be communicated to the KDC admin server):

```
kadmin.local:  ktadd host/masterhost06.jdillon.local
Entry for principal host/masterhost06.jdillon.local with kvno 3, encryption type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/masterhost06.jdillon.local with kvno 3, encryption type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/masterhost06.jdillon.local with kvno 3, encryption type des3-cbc-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/masterhost06.jdillon.local with kvno 3, encryption type arcfour-hmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/masterhost06.jdillon.local with kvno 3, encryption type camellia256-cts-cmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/masterhost06.jdillon.local with kvno 3, encryption type camellia128-cts-cmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/masterhost06.jdillon.local with kvno 3, encryption type des-hmac-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/masterhost06.jdillon.local with kvno 3, encryption type des-cbc-md5 added to keytab FILE:/etc/krb5.keytab.
```
```
kadmin.local:  ktadd nfs/masterhost06.jdillon.local
Entry for principal nfs/masterhost06.jdillon.local with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal nfs/masterhost06.jdillon.local with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal nfs/masterhost06.jdillon.local with kvno 2, encryption type des3-cbc-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal nfs/masterhost06.jdillon.local with kvno 2, encryption type arcfour-hmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal nfs/masterhost06.jdillon.local with kvno 2, encryption type camellia256-cts-cmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal nfs/masterhost06.jdillon.local with kvno 2, encryption type camellia128-cts-cmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal nfs/masterhost06.jdillon.local with kvno 2, encryption type des-hmac-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal nfs/masterhost06.jdillon.local with kvno 2, encryption type des-cbc-md5 added to keytab FILE:/etc/krb5.keytab.
```

### Creating principals for the client:

```
kadmin.local:  addprinc host/vm07.jdillon.local
WARNING: no policy specified for host/vm07.jdillon.local@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "host/vm07.jdillon.local@JDILLON.LOCAL":
Re-enter password for principal "host/vm07.jdillon.local@JDILLON.LOCAL":
add_principal: Principal or policy already exists while creating "host/vm07.jdillon.local@JDILLON.LOCAL".
```
```
kadmin.local:  addprinc nfs/vm07.jdillon.local
WARNING: no policy specified for nfs/vm07.jdillon.local@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "nfs/vm07.jdillon.local@JDILLON.LOCAL":
Re-enter password for principal "nfs/vm07.jdillon.local@JDILLON.LOCAL":
Principal "nfs/vm07.jdillon.local@JDILLON.LOCAL" created.
```

### Create a copy of the keytab file for the client:

```
kadmin.local:  ktadd -k /vm07.keytab host/vm07.jdillon.local
Entry for principal host/vm07.jdillon.local with kvno 3, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:/vm07.keytab.
Entry for principal host/vm07.jdillon.local with kvno 3, encryption type aes128-cts-hmac-sha1-96 added to keytab WRFILE:/vm07.keytab.
Entry for principal host/vm07.jdillon.local with kvno 3, encryption type des3-cbc-sha1 added to keytab WRFILE:/vm07.keytab.
Entry for principal host/vm07.jdillon.local with kvno 3, encryption type arcfour-hmac added to keytab WRFILE:/vm07.keytab.
Entry for principal host/vm07.jdillon.local with kvno 3, encryption type camellia256-cts-cmac added to keytab WRFILE:/vm07.keytab.
Entry for principal host/vm07.jdillon.local with kvno 3, encryption type camellia128-cts-cmac added to keytab WRFILE:/vm07.keytab.
Entry for principal host/vm07.jdillon.local with kvno 3, encryption type des-hmac-sha1 added to keytab WRFILE:/vm07.keytab.
Entry for principal host/vm07.jdillon.local with kvno 3, encryption type des-cbc-md5 added to keytab WRFILE:/vm07.keytab.
```
```
kadmin.local:  ktadd -k /vm07.keytab nfs/vm07.jdillon.local
Entry for principal nfs/vm07.jdillon.local with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:/vm07.keytab.
Entry for principal nfs/vm07.jdillon.local with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab WRFILE:/vm07.keytab.
Entry for principal nfs/vm07.jdillon.local with kvno 2, encryption type des3-cbc-sha1 added to keytab WRFILE:/vm07.keytab.
Entry for principal nfs/vm07.jdillon.local with kvno 2, encryption type arcfour-hmac added to keytab WRFILE:/vm07.keytab.
Entry for principal nfs/vm07.jdillon.local with kvno 2, encryption type camellia256-cts-cmac added to keytab WRFILE:/vm07.keytab.
Entry for principal nfs/vm07.jdillon.local with kvno 2, encryption type camellia128-cts-cmac added to keytab WRFILE:/vm07.keytab.
Entry for principal nfs/vm07.jdillon.local with kvno 2, encryption type des-hmac-sha1 added to keytab WRFILE:/vm07.keytab.
Entry for principal nfs/vm07.jdillon.local with kvno 2, encryption type des-cbc-md5 added to keytab WRFILE:/vm07.keytab.
```
### Exit the KDC management console and copy the keytab files to the client:

```
[root@masterhost06 ~]# scp /vm07.keytab vm07.jdillon.local:/etc/krb5.keytab
root@vm07.jdillon.local's password: #####
vm07.keytab

```
### Refer to the client configuration page, to confirm that the Kerberos server is granting a [ticket for the client](NFS_Client_Config#request) and that authentication is working correctly.

<hr><hr>

## Configure NFS Options for Exporting a directory utilizing Kerberos authentication:

### Create a new directory for Kerberized NFS export:

`[root@masterhost06 ~]# mkdir /nfs`

### Edit the server's /etc/exports file, adding export rules for the new directory with options pertaining to Kerberos:

`[root@masterhost06 ~]# vim /etc/exports`

```
/share   *(rw,no_root_squash)
/nfs     *(rw,no_root_squash,sec=krb5p)
```

### Restart the NFS-server:
`[root@masterhost06 ~]# systemctl restart nfs-server`

### Restart the RPC-bind service:
`[root@masterhost06 ~]# systemctl restart rpcbind`

### Adjust the firewall settings (nfs and rpc-bind were previously added, but showing full command for confirmation):

```
[root@masterhost06 ~]# firewall-cmd --permanent --add-service=nfs --add-service=mountd  --add-service=rpc-bind
Warning: ALREADY_ENABLED: nfs
Warning: ALREADY_ENABLED: rpc-bind
success
[root@masterhost06 ~]# firewall-cmd --reload
success
```

### Adjust SELinux settings for the /nfs directory (changing context of the directory from default to public read/write):

```
[root@masterhost06 ~]# semanage fcontext -a -t public_content_rw_t "/nfs(/.*)?"
[root@masterhost06 ~]# restorecon -Rv /nfs
restorecon reset /nfs context unconfined_u:object_r:default_t:s0->unconfined_u:object_r:public_content_rw_t:s0
[root@masterhost06 ~]# ls -lZd /nfs
drwxr-xr-x. root root unconfined_u:object_r:public_content_rw_t:s0 /nfs
```
### Adjust /nfs directory permissions:

`[root@masterhost06 ~]# chmod 777 /nfs`

### [Restart the NFS-client service.](NFS_Client_Config.md#restart)
<hr><hr>

### Having confirmed proper mounting of the Kerberos authenticated directory, we are ready to establish user principles for the KDC server (granting access to the shared directory).
## <a name="user">NFS-user-principle</a>
A principle will be added for user 'jdillon', this is assuming 'jdillon' is a non-root user that has already been set up on the client.

```
[root@masterhost06 ~]# kadmin.local
Authenticating as principal root/admin@JDILLON.LOCAL with password.
```

```
kadmin.local:  addprinc jdillon
WARNING: no policy specified for jdillon@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "jdillon@JDILLON.LOCAL":
Re-enter password for principal "jdillon@JDILLON.LOCAL":
add_principal: Principal or policy already exists while creating "jdillon@JDILLON.LOCAL".
kadmin.local:  exit
```
### Refer to client configuration page/section [principle for user 'jdillon'](NFS_Client_Config.md#principle) and attempt to get access to the Kerberized NFS share:

<hr><hr>

## Configuring a Colloborative NFS Share (non-Kerberized): 

### It is often useful, to add a 'setgid' parameter to an NFS shared directory.  This will allow for members of a group to have access to all created contents.

### Confirm our current exports:

```
[root@masterhost06 ~]# exportfs -avr
exporting *:/nfs
exporting *:/share
[root@masterhost06 ~]# showmount -e localhost
Export list for localhost:
/nfs   *
/share *
```

(Remember that /nfs is our Kerberized share, and /share is not using Kerberos principles:

```
[root@masterhost06 ~]# cat /etc/exports
/share   *(rw,no_root_squash)
/nfs     *(rw,no_root_squash,sec=krb5p)
```

### Configure setgid settings for exported directory

```
[root@masterhost06 ~]# groupadd nfsgroup
[root@masterhost06 ~]# chmod 2770 /share
[root@masterhost06 ~]# chgrp nfsgroup /share
[root@masterhost06 ~]# ls -ld /share
drwxrws---. 2 root nfsgroup 6 May 19 22:16 /share
```

### Now that setgid has been configured on the directory, we can [configure our client](NFS_Client_Config.md#setgid)
