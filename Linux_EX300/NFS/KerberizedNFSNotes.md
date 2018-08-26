# NFS v4 Notes:
<hr><hr>

## Directory Contents (Please configure steps in this order):

* 1.) [NFS Server Configuration](NFS_Server_Config)
* 2.) [NFS Client Configuration](NFS_Client_Config)

<hr><hr>

### Features:
--

* Kerberos Integration
* Uses port 2049, thereby aids in firewall configuration
* Higher performance than NFSv3
* Supports the ACL using nfs4_setfacl and nfs4_getfacl commands.
* Main Packages:
  * 1. **nfs_utils**: Provides the NFS Daemon for NFS server.
  * 2. **nfs4-acl-tools**: Command line ACL utility for NFSv4 client.

## Configuring NFS Using Kerberos Authentication

### We must consider both the Kerberos, and NFS sides of the configuration:

### Kerberos:
<hr>
**Creating Kerberos Principals for NFS:**

* We will be creating our Kerberos and NFS servers on masterhost06.jdillon.local (10.0.2.6)
* Our client will be on vm07.jdillon.local (10.0.2.7)

>>>within our Kerberos server:

```
[root@masterhost06 ~]# kadmin.local
```
### Adding a host principal (for both server and client):
```
kadmin$ addprinc host/masterhost06.jdillon.local
kadmin$ addprinc host/vm07.jdillon.local
```
### Adding an NFS principal (for both server and client):
```
kadmin$ addprinc nfs/masterhost06.jdillon.local
kadmin$ addprinc nfs/vm07.jdillon.local
```
### Creating a local copy of the keytab file for the server:

```
kadmin$ ktadd host/masterhost06.jdillon.local
kadmin$ ktadd nfs/masterhost06.jdillon.local
```

### Creating a local copy of the keytab file for the client:

```
kadmin$ ktadd -k /vm07.keytab host/vm07.jdillon.local
kadmin$ ktadd -k /vm07.keytab nfs/vm07.jdillon.local
```

### Exit from the Kadmin interface and copy the client keytab file from /vm07.keytab to our client:

`[root@masterhost06 ~]# scp /vm07.keytab vm07.jdillon.local:/etc/krb5.keytab`

>>>On client vm07.jdillon.local,we must verify that the client keytab file is authorizing correctly.

```
[root@vm07 ~]# kinit -k -t /etc/krb5.keytab nfs/vm07.jdillon.local
[root@vm07 ~]# klist
```

### Kerberized NFS:

>>>Pertaining to the /etc/exports configuration file for NFS:

**sec=krb5:**

* This option provides Kerberos user authentication ONLY while the client request is confirmed with he keytab file

**sec=krb5i:**

* In addition to the krb5 option, it provides integrity communication with Kerberos.

**sec=krb5p:**

* The most secure of available options.
