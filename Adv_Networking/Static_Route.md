# Static Routing:

<hr><hr>

### Static routing is for traffic that must not, or should not go through the default gateway.  This can include connections that must pass through an encrypted VPN tunnel, or traffic that should take a specific route for reasons of cost or security.

### We will configure a static route on vm08 (10.0.2.8) to the network 128.55.211.0/24 via a new gateway 10.0.2.100:

```
[root@vm08 ~]# nmcli con show
NAME    UUID                                  TYPE      DEVICE
enp0s3  fad7c00c-cff2-43bd-b76e-1f01c409a8a8  ethernet  enp0s3
enp0s8  5cf51933-3886-4a4e-95bc-14e1422a4b1e  ethernet  --
```

### Notice the '+' prepended to the ipv4.routes argument, this will add a new route to the existing interface:

```
[root@vm08 ~]# nmcli con modify enp0s3 +ipv4.routes '128.55.211.0/24 10.0.2.100'
[root@vm08 ~]# nmcli con down enp0s3
Connection 'enp0s3' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/1)

[root@vm08 ~]# nmcli con modify enp0s3 +ipv4.routes '128.55.211.0/24 10.0.2.100'
[root@vm08 ~]# nmcli con down enp0s3
Connection 'enp0s3' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/1)
```

### Confirm that our static route was created:

```
[root@vm08 ~]# ip route
default via 10.0.2.1 dev enp0s3 proto static metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.8 metric 100
128.55.211.0/24 via 10.0.2.100 dev enp0s3 proto static metric 100
```

### We can also check the /etc/sysconfig/network-scripts directory, to confirm that our nmcli configuration has in fact created a new route file:

```
[root@vm08 ~]# ls /etc/sysconfig/network-scripts
ifcfg-enp0s3  ifdown-isdn      ifup          ifup-plusb     ifup-wireless
ifcfg-enp0s8  ifdown-post      ifup-aliases  ifup-post      init.ipv6-global
ifcfg-lo      ifdown-ppp       ifup-bnep     ifup-ppp       network-functions
ifdown        ifdown-routes    ifup-eth      ifup-routes    network-functions-ipv6
ifdown-bnep   ifdown-sit       ifup-ippp     ifup-sit       route-enp0s3
ifdown-eth    ifdown-Team      ifup-ipv6     ifup-Team
ifdown-ippp   ifdown-TeamPort  ifup-isdn     ifup-TeamPort
ifdown-ipv6   ifdown-tunnel    ifup-plip     ifup-tunnel
```

### Notice the route-enp0s3 file, querying its contents:

```
[root@vm08 ~]# cat /etc/sysconfig/network-scripts/route-enp0s3
ADDRESS0=128.55.211.0
NETMASK0=255.255.255.0
GATEWAY0=10.0.2.100
```

### We see that the file configuration reflects our nmcli command exactly

### We can initiate an interactive nmcli session, and remove the static route we created for enp0s3:

```
[root@vm08 ~]# nmcli con edit enp0s3
^C
===| nmcli interactive connection editor |===

Editing existing '802-3-ethernet' connection: 'enp0s3'

Type 'help' or '?' for available commands.
Type 'describe [<setting>.<prop>]' for detailed property description.

You may edit the following settings: connection, 802-3-ethernet (ethernet), 802-1x, dcb, ipv4, ipv6, tc, proxy
nmcli> ^C
nmcli> remove ipv4.routes
nmcli> save
Connection 'enp0s3' (fad7c00c-cff2-43bd-b76e-1f01c409a8a8) successfully updated.
nmcli> quit
```

### Stop and restart the connection via nmcli:

```
[root@vm08 ~]# nmcli con down enp0s3
Connection 'enp0s3' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
[root@vm08 ~]# nmcli con up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
```

### Confirm our static route has been removed

```
[root@vm08 ~]# ip route
default via 10.0.2.1 dev enp0s3 proto static metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.8 metric 100
```

<hr><hr>
