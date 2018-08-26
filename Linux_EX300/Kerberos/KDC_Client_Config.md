# **<span style='color:purple;font-family:Courier'>Kerberos Client Configuration:</span>**
<hr><hr>

#### Please be sure to set up the server, before attempting client configuration:

<em><span style='color:red'>Server lives on masterhost06.jdillon.local (10.0.2.6)
Client authentication will be taking place on vm07.jdillon.local (10.0.2.7)</span></em>

### Configure kernel level domain and host names on vm07.jdillon.local (10.0.2.7):

`[root@vm07 ~]# vim /etc/sysctl.conf`

```
kernel.hostname = vm07.jdillon.local
kernel.domainname = jdillon.local
```

`[root@vm07 ~]# sysctl -p`

```
kernel.hostname = vm07.jdillon.local<br>
kernel.domainname = jdillon.local
```

### We are depending upon the hosts file, since <span style='color:red'>DNS</span> has yet to be configured within the network

`[root@vm07 ~]# vim /etc/hosts`

```
127.0.0.1   localhost localhost.localdomain localhost4
localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.2.5    vm05.jdillon.local vm05
10.0.2.6    masterhost06.jdillon.local masterhost06
10.0.2.7    vm07.jdillon.local vm07
10.0.2.8    vm08.jdillon.local vm08
10.0.2.9    vm09.jdillon.local vm09
```
### Install Kerberos client package:

`[root@vm07 ~]# yum install -y krb5-workstation`

### See which packages were installed:

`[root@vm07 ~]# rpm -qa | grep krb5`

```
sssd-krb5-1.16.0-19.el7.x86_64
sssd-krb5-common-1.16.0-19.el7.x86_64
krb5-workstation-1.15.1-19.el7.x86_64
krb5-libs-1.15.1-19.el7.x86_64
```

### Kerberos authentication requires the 'pam_krb5' package, which may not installed by default. We see above that it is missing. 
### Forcing the install:

`[root@vm07 ~]# yum -y install pam_krb5*`

```
sssd-krb5-1.16.0-19.el7.x86_64
sssd-krb5-common-1.16.0-19.el7.x86_64
krb5-workstation-1.15.1-19.el7.x86_64
pam_krb5-2.4.8-6.el7.x86_64
krb5-libs-1.15.1-19.el7.x86_64
```

### There are three methods for authentication: 

   1. Authconfig-tui: Minimal graphic install
   2. Authconfig-gtk: Full graphic install
   3. Authoconfig-cli: Command-line utility

### <span style='color:red'>Authconfig-gtk</span> will be deprecated in RHEL08, we will use the command-line utility:

```
[root@vm07 ~]# authconfig --update --enablekrb5 --krb5kdc=10.0.2.6 --krb5adminserver=10.0.2.6 --krb5realm=JDILLON.LOCAL
```

### We can verify our configuration using the base gui:

`[root@vm07 ~]# authconfig-tui`

>>>(will display gui, and should reflect settings from original authconfig command)

### <span style="color:red">Firewalld</span> rules must now be changed to allow Kerberos through the firewall:

```
[root@vm07 ~] firewall-cmd --permanent --add-service=kerberos
success
[root@vm07 ~] firewall-cmd --permanent --add-service=kadmin
success
[root@vm07 ~] firewalld-cmd --permanent --add-port=749/tcp
success
[root@vm07 ~] firewall-cmd --reload
success
```

### KDC principle has been established for user root, we will now login to the <span style='color:red'>Kerberos</span> admin interface on the client:

`[root@vm07 ~]# kadmin`

```
Authenticating as principal root/admin@JDILLON.LOCAL with password.
Password for root/admin@JDILLON.LOCAL: ######
```

### Adding a principle in <span style='color:red'>Kerberos</span> for this client/host:

`$kadmin: addprinc host/vm07.jdillon.local`

```
WARNING: no policy specified for host/vm07.jdillon.local@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "host/vm07.jdillon.local@JDILLON.LOCAL": #####
Re-enter password for principal "host/vm07.jdillon.local@JDILLON.LOCAL": #####
Principal "host/vm07.jdillon.local@JDILLON.LOCAL" created.
```

### The client needs a copy from the server's keytab file for authentication:

`$kadmin: ktadd host/vm07.jdillon.local`

```
Entry for principal host/vm07.jdillon.local with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/vm07.jdillon.local with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/vm07.jdillon.local with kvno 2, encryption type des3-cbc-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/vm07.jdillon.local with kvno 2, encryption type arcfour-hmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/vm07.jdillon.local with kvno 2, encryption type camellia256-cts-cmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/vm07.jdillon.local with kvno 2, encryption type camellia128-cts-cmac added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/vm07.jdillon.local with kvno 2, encryption type des-hmac-sha1 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/vm07.jdillon.local with kvno 2, encryption type des-cbc-md5 added to keytab FILE:/etc/krb5.keytab.
```

`$kadmin: exit`

### We will create a test user, to verify ticketing is working:

```
[root@vm07 ~]# useradd kdctest
[root@vm07 ~]# passwd kdctest
```
```
New password: #####
Retype new password: #####
passwd: all authentication tokens updated successfully.
```

`[root@vm07 ~]# kadmin`

### Adding a principle for the new user:
 
`$kadmin:  addprinc kdctest`

```
WARNING: no policy specified for kdctest@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "kdctest@JDILLON.LOCAL":
Re-enter password for principal "kdctest@JDILLON.LOCAL":
Principal "kdctest@JDILLON.LOCAL" created.
```

`$kadmin: exit`

### Request a ticket as user kdctest from the server:

```
[root@vm07 ~]# su kdctest
[kdctest@vm07 root]$ kinit
```
```
Password for kdctest@JDILLON.LOCAL: #####

```

#### 'klist' command will show current tickets for user:

`[kdctest@vm07 root]$ klist`

```
Ticket cache: KEYRING:persistent:1001:1001
Default principal: kdctest@JDILLON.LOCAL

Valid starting       Expires              Service principal
05/17/2018 06:46:21  05/18/2018 06:46:21  krbtgt/JDILLON.LOCAL@JDILLON.LOCAL
```

#### <span style="color:red">Confirming authentication</span>: connecting to kdc server using ssh w/ credentials supplied by KDC ticket:

Kerberos is able to authenticate ssh sessions, via the PAM module.  The client's ssh configuration must have GSSAPI authentication enabled for this to work.

`$ vim /etc/ssh/ssh_config`

```
GSSAPIAuthentication yes
GSSAPIDelegateCredentials yes
```

#### User kdctest should now be able to log onto kdc server (10.0.2.6) without providing password (until ticket expires):

```
[kdctest@vm07 root]$ ssh masterhost06
```
```
Last login: Thu May 17 17:21:00 2018 from vm07.jdillon.local
[kdctest@masterhost06 ~]$
```
<hr><hr>
## Great success++
