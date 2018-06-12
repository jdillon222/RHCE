# Network Teaming:

<hr><hr>

## Link Aggregation:

* Link aggregation is defined as the ability to aggregate multiple network links into one link, in order to increase reachability, redundancy and load-balancing of the network.
* In Centos7 & RHEL7 it is referred to as teaming, whereas in version 6 it was referred to as bonding.
* Multiple aggregation modes:
  * 1). Roundrobin:
    * The default mode, it balances the load to all ports per network request.
  * 2). Broadcast:
    * All traffic will be sent over all ports.
  * 3). Active-backp:
    * One interface will be handling network requests, while the other is in passive state in case any failover changes.

<hr><hr>

## Manual configuration of team interface:

### Install the teamd daemon:

```
[root@vm08 ~]# yum install -y teamd
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: mirror.hostduplex.com
 * extras: mirror.keystealth.org
 * updates: mirror.hostduplex.com
base                                                            | 3.6 kB  00:00:00
extras                                                          | 3.4 kB  00:00:00
updates                                                         | 3.4 kB  00:00:00
(1/4): extras/7/x86_64/primary_db                               | 147 kB  00:00:00
(2/4): base/7/x86_64/group_gz                                   | 166 kB  00:00:00
(3/4): updates/7/x86_64/primary_db                              | 2.0 MB  00:00:00
(4/4): base/7/x86_64/primary_db                                 | 5.9 MB  00:00:00
Package teamd-1.27-4.el7.x86_64 already installed and latest version
Nothing to do
```

### Using nmcli, add a team interface (utlizing 'activebackup'):

```
[root@vm08 ~]# nmcli connection add type team con-name team0 ifname team0 autoconnect yes config '{"runner":{"name":"activebackup"}}'
Connection 'team0' (7a17b44e-c86d-46ca-9a0a-68202aa2ce47) successfully added.
```

### Confirm our current interfaces:

```
NAME    UUID                                  TYPE      DEVICE
enp0s3  fad7c00c-cff2-43bd-b76e-1f01c409a8a8  ethernet  enp0s3
team0   7a17b44e-c86d-46ca-9a0a-68202aa2ce47  team      team0
enp0s8  5cf51933-3886-4a4e-95bc-14e1422a4b1e  ethernet  --
```

### Disconnect enp0s3:

```
[root@vm08 ~]# nmcli con down enp0s3
```

### Note that if you had ssh'd into the host, connection would be lost at this point.  Thus, these steps would need to be performed physically at the host terminal.  

### Add interfaces enp0s3 and enp0s8 as slaves to team0:

```
[root@vm08 ~]# nmcli con add type team-slave ifname enp0s3 con-name team-enp0s3 autoconnect yes master team0
Connection 'team-enp0s3' (b140884f-f3ef-4f4a-9889-88f370f9ddad) successfully added
[root@vm08 ~]# nmcli con add type team-slave ifname enp0s8 con-name team-enp0s8 autoconnect yes master team0
Connection 'team-enp0s8' (1e91ad31-5b14-46a9-9022-9b8c523908f3) successfully added.
```

### Bring team0 up:

```
[root@vm08 ~]# nmcli con up team0
Connection successfully activated (master waiting for slaves) (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
```

### Set team0's ip address to the original static ip of the host (10.0.2.8):

```
[root@vm08 ~]# nmcli con modify team0 ipv4.addresses 10.0.2.8/24 ipv4.gateway 10.0.2.1
[root@vm08 ~]# nmcli connection modify team0 ipv4 method manual
```

### Bring up the slave interfaces:

```
[root@vm08 ~]# nmcli connection up team-enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
[root@vm08 ~]# nmcli connection up team-enp0s8
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
```

### Check out the connection configuration:

```
[root@vm08 ~]# nmcli con show
NAME         UUID                                  TYPE      DEVICE
team-enp0s3  b140884f-f3ef-4f4a-9889-88f370f9ddad  ethernet  enp0s3
team-enp0s8  1e91ad31-5b14-46a9-9022-9b8c523908f3  ethernet  enp0s8
team0        7a17b44e-c86d-46ca-9a0a-68202aa2ce47  team      team0
enp0s3       fad7c00c-cff2-43bd-b76e-1f01c409a8a8  ethernet  --
enp0s8       5cf51933-3886-4a4e-95bc-14e1422a4b1e  ethernet  --
```

### Confirm our teamed network:

```
[root@vm08 ~]# teamdctl team0 stat
setup:
  runner: activebackup
ports:
  enp0s3
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
        down count: 0
  enp0s8
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
        down count: 0
runner:
  active port: enp0s3
```

### We can see that enp0s3 is currently the active interface, if it were to go down enp0s8 is now configured to take over network connections.

### Since we previously had a working static configuration, we should restart the network and confirm that team0 now has the desired static configuration:

```
[root@vm08 Desktop]# systemctl restart network
[root@vm08 Desktop]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:24:4c:62 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.8/24 brd 10.0.2.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::eb16:b78f:db88:6459/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master team0 state UP group default qlen 1000
    link/ether 08:00:27:6c:1e:9d brd ff:ff:ff:ff:ff:ff
6: team0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:6c:1e:9d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.8/24 brd 10.0.2.255 scope global noprefixroute team0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe6c:1e9d/64 scope link
       valid_lft forever preferred_lft forever
```

### Very good..

<hr><hr>
