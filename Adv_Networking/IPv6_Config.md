# IPv6 Configuration:

<hr><hr>

### On vm09 (10.0.10.9), we will manually add an IPv6 address.  We can do this by manually adding an address to our new br0 bridged connection:

```
[root@vm09 ~]# cd /etc/sysconfig/network-scripts
[root@vm09 network-scripts]# ls
ifcfg-br0      ifdown-ippp    ifdown-TeamPort  ifup-isdn    ifup-TeamPort
ifcfg-br0-nic  ifdown-ipv6    ifdown-tunnel    ifup-plip    ifup-tunnel
ifcfg-enp0s3   ifdown-isdn    ifup             ifup-plusb   ifup-wireless
ifcfg-enp0s8   ifdown-post    ifup-aliases     ifup-post    init.ipv6-global
ifcfg-lo       ifdown-ppp     ifup-bnep        ifup-ppp     network-functions
ifdown         ifdown-routes  ifup-eth         ifup-routes  network-functions-ipv6
ifdown-bnep    ifdown-sit     ifup-ippp        ifup-sit
ifdown-eth     ifdown-Team    ifup-ipv6        ifup-Team
[root@vm09 network-scripts]# vim *br0
```

```
STP=yes
BRIDGING_OPTS=priority=32768
TYPE=Bridge
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=br0
UUID=b5383822-fc56-4e58-b784-b7ab880018a6
DEVICE=br0
ONBOOT=yes
IPADDR=10.0.2.9
PREFIX=24
GATEWAY=10.0.2.1
###adding below###
IPV6ADDR= #add desired IPV6address
IPV6_DEFAULTGW= #add default gateway
```

### We can also configure IPV6 via the nmcli command:

```
[root@vm09 network-scripts]# nmcli con modify br0 ipv6.addresses 'fe80::20c:29ff:fe27:f2b6/64'
[root@vm09 network-scripts]# nmcli con modify br0 ipv6.method manual
[root@vm09 network-scripts]# nmcli con reload
[root@vm09 network-scripts]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br0 state UP group default qlen 1000
    link/ether 08:00:27:24:4c:62 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:6c:1e:9d brd ff:ff:ff:ff:ff:ff
5: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:24:4c:62 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.9/24 brd 10.0.2.255 scope global noprefixroute br0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe24:4c62/64 scope link
       valid_lft forever preferred_lft forever
```

### Ping6 on the loopback device:

```
[root@vm09 network-scripts]# ping6 ::1
PING ::1(::1) 56 data bytes
64 bytes from ::1: icmp_seq=1 ttl=64 time=0.041 ms
64 bytes from ::1: icmp_seq=2 ttl=64 time=0.059 ms
64 bytes from ::1: icmp_seq=3 ttl=64 time=0.049 ms
64 bytes from ::1: icmp_seq=4 ttl=64 time=0.050 ms
64 bytes from ::1: icmp_seq=5 ttl=64 time=0.058 ms
```

### Check for ports and services listening on IPV6:

```
[root@vm09 network-scripts]# netstat -antup6
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp6       0      0 :::111                  :::*                    LISTEN      654/rpcbind
tcp6       0      0 :::22                   :::*                    LISTEN      1071/sshd
tcp6       0      0 ::1:631                 :::*                    LISTEN      1072/cupsd
tcp6       0      0 ::1:25                  :::*                    LISTEN      1472/master
udp6       0      0 ::1:323                 :::*                                685/chronyd
udp6       0      0 :::819                  :::*                                654/rpcbind
udp6       0      0 :::111                  :::*                                654/rpcbind
```

<hr><hr>
