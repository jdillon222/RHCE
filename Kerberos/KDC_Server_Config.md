# **<span style='color:purple;font-family:Courier'>Kerberos Server Configuration:</span>**
<hr><hr>

### <em>(Please be sure to set up the server, before attempting client configuration)</em>:

### Domain and host names should be defined at the kernel level, in /etc/sysctl.conf (advisable to edit /etc/hostname as well):

`[root@masterhost06 ~]# domainname`

```
[none]
```

`[root@masterhost06 ~]# vim /etc/sysctl.conf`

```
kernel.hostname = masterhost06.jdillon.local
kernel.domainname = jdillon.local
```
**confirming**:

`[root@masterhost06 ~]# sysctl -p`

```
kernel.hostname = masterhost06.jdillon.local
kernel.domainname = jdillon.local
```

`[root@masterhost06 ~]# domainname`

```
jdillon.local
```

### Installing Kerberos server and the Kerberos PAM module:

`[root@masterhost06 ~]# yum -y install -y krb5-server pam_krb5`

### Now we must replace any example parameters from Kerberos configuration files (search for EXAMPLE.COM):

`[root@masterhost06 ~]# vim /etc/krb5.conf`

**(Original):**
  
```
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
  default = FILE:/var/log/krb5libs.log
  kdc = FILE:/var/log/krb5kdc.log
  admin_server = FILE:/var/log/kadmind.log

[libdefaults]
  dns_lookup_realm = false
  ticket_lifetime = 24h
  renew_lifetime = 7d
  forwardable = true
  rdns = false
 # default_realm = EXAMPLE.COM
  default_ccache_name = KEYRING:persistent:%{uid}

[realms]
# EXAMPLE.COM = {
#  kdc = kerberos.example.com
#  admin_server = kerberos.example.com
# }

[domain_realm]
# .example.com = EXAMPLE.COM
# example.com = EXAMPLE.COM
```

### We must replace any instances of EXAMPLE.COM with our own realm (JDILLON.LOCAL) and host-name (masterhost06.jdillon.local):

**(Reconfigured):**
 
```
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
  default = FILE:/var/log/krb5libs.log
  kdc = FILE:/var/log/krb5kdc.log
  admin_server = FILE:/var/log/kadmind.log

[libdefaults]
  dns_lookup_realm = false
  ticket_lifetime = 24h
  renew_lifetime = 7d
  forwardable = true
  rdns = false
  default_realm = JDILLON.LOCAL
  default_ccache_name = KEYRING:persistent:%{uid}

[realms]
  JDILLON.LOCAL = {
  kdc = masterhost06.jdillon.local
  admin_server = masterhost06.jdillon.local
}

[domain_realm]
  .jdillon.local = JDILLON.LOCAL
   jdillon.local = JDILLON.LOCAL
```

### Changing the Kerberos access control list specification file (/var/kerberos/krb5kdc/kadm5.acl):

`[root@masterhost06 ~]# vim /var/kerberos/krb5kdc/kadm5.acl`

```
*/admin@JDILLON.LOCAL   *
```

### Changing kdc.conf file:

`[root@masterhost06 ~]# vim /var/kerberos/krb5kdc/kdc.conf`

**(Original):**

```
[kdcdefaults]
  kdc_ports = 88
  kdc_tcp_ports = 88

[realms]
  EXAMPLE.COM = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal 
                       arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal 
                       des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
}
```
**(Reconfigured):**

```
 [kdcdefaults]
  kdc_ports = 88
  kdc_tcp_ports = 88

[realms]
  JDILLON.LOCAL = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal 
                       arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal 
                       des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
} 
```

### Next, the credentials database must be created:

`[root@masterhost06 ~]# kdb5_util create -s -r JDILLON.LOCAL`

```
Loading random data
Initializing database '/var/kerberos/krb5kdc/principal' for realm 'JDILLON.LOCAL',
master key name 'K/M@JDILLON.LOCAL'
You will be prompted for the database Master Password.
It is important that you NOT FORGET this password.
Enter KDC database master key: ######
Re-enter KDC database master key to verify: ######
```

### Kerberos credentials database is now created.  We can now enable and start Kerberos and Kerberos Admin services:

```
[root@masterhost06 ~]# systemctl start krb5kdc kadmin
[root@masterhost06 ~]# systemctl enable krb5kdc kadmin
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/krb5kdc.service to /usr/lib/systemd/system/krb5kdc.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/kadmin.service to /usr/lib/systemd/system/kadmin.service.
```


### <span style="color:red">Firewalld</span> rules must now be changed to allow Kerberos through the firewall:

```
[root@masterhost06 ~]# firewall-cmd --permanent --add-service=kerberos
success
[root@masterhost06 ~]# firewall-cmd --permanent --add-service=kadmin
success
[root@masterhost06 ~]# firewalld-cmd --permanent --add-port=749/tcp
success
[root@masterhost06 ~]# firewall-cmd --reload
success
```

### Kerberos (kdc server) is now configured properly.  We can now log in to the kdc management console (initiates the 'kadmin' interactive session):

`[root@masterhost06 ~]# kadmin.local`

```
Authenticating as principal root/admin@JDILLON.LOCAL with password.
```

#### Listing current server principles (default principles created by Kerberos authentication):

`[root@masterhost06 ~]# kadmin.local:  listprincs`

```
K/M@JDILLON.LOCAL
kadmin/admin@JDILLON.LOCAL
kadmin/changepw@JDILLON.LOCAL
kadmin/masterhost06.jdillon.local@JDILLON.LOCAL
kiprop/masterhost06.jdillon.local@JDILLON.LOCAL
krbtgt/JDILLON.LOCAL@JDILLON.LOCAL
```

#### Adding a Kerberos principle for user 'kdctest':

`$kadmin.local: addprinc kdctest`

```
WARNING: no policy specified for kdctest@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "kdctest@JDILLON.LOCAL": #####
Re-enter password for principal "kdctest@JDILLON.LOCAL": #####
Principal "kdctest@JDILLON.LOCAL" created.
```

### Adding a Kerberos principle for 'root' user (defining root as admin):

```
$kadmin.local:  addprinc root/admin@JDILLON.LOCAL
WARNING: no policy specified for root/admin@JDILLON.LOCAL; defaulting to no policy
Enter password for principal "root/admin@JDILLON.LOCAL":
Re-enter password for principal "root/admin@JDILLON.LOCAL":
Principal "root/admin@JDILLON.LOCAL" created.
```

`$kadmin.local:  exit`

### Update configuration:

`[root@masterhost06 ~]# authconfig --enablekrb5  --update`

<hr><hr>

### We are now ready to configure [our client](KDC_Client_Config.md) vm07.jdillon.local (10.0.2.7)

