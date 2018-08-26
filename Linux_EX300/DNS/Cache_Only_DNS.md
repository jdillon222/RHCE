# Cache-Only DNS Configuration:

<hr><hr>

### Cache-only server will be set up on Masterhost06 (10.0.2.6)
### Client will verify DNS on vm07 (10.0.2.7)
### Install the unbound package on Masterhost06:

```
[root@masterhost06 ~]# yum install -y unbound
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: mirror.sfo12.us.leaseweb.net
 * extras: centos.eecs.wsu.edu
 * updates: mirrors.usc.edu
base                                                                                  | 3.6 kB  00:00:00
extras                                                                                | 3.4 kB  00:00:00
updates                                                                               | 3.4 kB  00:00:00
(1/2): extras/7/x86_64/primary_db                                                     | 150 kB  00:00:00
(2/2): updates/7/x86_64/primary_db                                                    | 3.6 MB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package unbound.x86_64 0:1.6.6-1.el7 will be installed
--> Processing Dependency: unbound-libs(x86-64) = 1.6.6-1.el7 for package: unbound-1.6.6-1.el7.x86_64
--> Processing Dependency: libunbound.so.2()(64bit) for package: unbound-1.6.6-1.el7.x86_64
--> Running transaction check
---> Package unbound-libs.x86_64 0:1.6.6-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================
 Package                      Arch                   Version                      Repository            Size
=============================================================================================================
Installing:
 unbound                      x86_64                 1.6.6-1.el7                  base                 673 k
Installing for dependencies:
 unbound-libs                 x86_64                 1.6.6-1.el7                  base                 405 k

Transaction Summary
=============================================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 1.1 M
Installed size: 3.5 M
Downloading packages:
(1/2): unbound-libs-1.6.6-1.el7.x86_64.rpm                                            | 405 kB  00:00:00
(2/2): unbound-1.6.6-1.el7.x86_64.rpm                                                 | 673 kB  00:00:00
-------------------------------------------------------------------------------------------------------------
Total                                                                        2.2 MB/s | 1.1 MB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : unbound-libs-1.6.6-1.el7.x86_64                                                           1/2
  Installing : unbound-1.6.6-1.el7.x86_64                                                                2/2
  Verifying  : unbound-1.6.6-1.el7.x86_64                                                                1/2
  Verifying  : unbound-libs-1.6.6-1.el7.x86_64                                                           2/2

Installed:
  unbound.x86_64 0:1.6.6-1.el7

Dependency Installed:
  unbound-libs.x86_64 0:1.6.6-1.el7

Complete!
```

### Enable the service, and allow DNS through the firewall:

```
[root@masterhost06 ~]# systemctl enable unbound
Created symlink from /etc/systemd/system/multi-user.target.wants/unbound.service to /usr/lib/systemd/system/unbound.service.
[root@masterhost06 ~]# firewall-cmd --permanent --add-service=dns
success
[root@masterhost06 ~]# firewall-cmd --reload
success
```

### Next, we must edit some parameters in the DNS configuration file /etc/unbound/unbound.conf:

`vim /etc/unbound/unbound.conf` 

### Specifically, uncomment the following parameters and change to values below:

```
interface: 0.0.0.0
interface: 10.0.2.6

access-control: 0.0.0.0/0 allow

  forward-zone:
        name: "."
        forward-addr: 8.8.8.8
        forward-addr: 8.8.4.4

do-ip4: yes
do-upd: yes
do-tcp: yes

logfile: /var/log/unbound
```

### Start the service:

`[root@masterhost06 ~]# systemctl start unbound`

### We can check our configuration file for syntax-errors using unbound-checkconf:

```
[root@masterhost06 ~]# unbound-checkconf
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```

### Our Cache-only server appears to be set up correctly.
### We can try pinging Google, and check DNS records using dig:

```
[root@masterhost06 ~]# ping google.com
PING google.com (216.58.195.78) 56(84) bytes of data.
64 bytes from sfo07s16-in-f78.1e100.net (216.58.195.78): icmp_seq=1 ttl=55 time=5.31 ms
64 bytes from sfo07s16-in-f78.1e100.net (216.58.195.78): icmp_seq=2 ttl=55 time=7.03 ms
64 bytes from sfo07s16-in-f78.1e100.net (216.58.195.78): icmp_seq=3 ttl=55 time=7.09 ms
64 bytes from sfo07s16-in-f78.1e100.net (216.58.195.78): icmp_seq=4 ttl=55 time=7.44 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3060ms
rtt min/avg/max/mdev = 5.315/6.722/7.446/0.831 ms

[root@masterhost06 ~]# dig google.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7188
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		299	IN	A	216.58.195.78

;; Query time: 27 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Jul 02 23:29:08 EDT 2018
;; MSG SIZE  rcvd: 55

```

### We can check records using the host command as well:

```
[root@masterhost06 ~]# host -a -v google.com
Trying "google.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19638
;; flags: qr rd ra; QUERY: 1, ANSWER: 16, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.			IN	ANY

;; ANSWER SECTION:
google.com.		299	IN	A	216.58.195.78
google.com.		299	IN	AAAA	2607:f8b0:4005:807::200e
google.com.		599	IN	MX	50 alt4.aspmx.l.google.com.
google.com.		299	IN	TXT	"docusign=05958488-4752-4ef2-95eb-aa7ba8a3bd0e"
google.com.		21599	IN	NS	ns4.google.com.
google.com.		21599	IN	CAA	0 issue "pki.goog"
google.com.		3599	IN	TXT	"v=spf1 include:_spf.google.com ~all"
google.com.		599	IN	MX	10 aspmx.l.google.com.
google.com.		599	IN	MX	20 alt1.aspmx.l.google.com.
google.com.		59	IN	SOA	ns1.google.com. dns-admin.google.com. 203527018 900 900 1800 60
google.com.		599	IN	MX	40 alt3.aspmx.l.google.com.
google.com.		21599	IN	NS	ns3.google.com.
google.com.		21599	IN	NS	ns2.google.com.
google.com.		599	IN	MX	30 alt2.aspmx.l.google.com.
google.com.		3599	IN	TXT	"facebook-domain-verification=22rm551cu4k0ab0bxsw536tlds4h95"
google.com.		21599	IN	NS	ns1.google.com.

Received 503 bytes from 8.8.8.8#53 in 30 ms

```

### The whois command also has some useful functionality (but likely needs to be installed):

```
[root@masterhost06 ~]# yum -y install whois
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.sfo12.us.leaseweb.net
 * extras: centos.eecs.wsu.edu
 * updates: mirrors.usc.edu
Resolving Dependencies
--> Running transaction check
---> Package whois.x86_64 0:5.1.1-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================
 Package                 Arch                     Version                       Repository              Size
=============================================================================================================
Installing:
 whois                   x86_64                   5.1.1-2.el7                   base                    72 k

Transaction Summary
=============================================================================================================
Install  1 Package

Total download size: 72 k
Installed size: 222 k
Downloading packages:
whois-5.1.1-2.el7.x86_64.rpm                                                          |  72 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : whois-5.1.1-2.el7.x86_64                                                                  1/1
  Verifying  : whois-5.1.1-2.el7.x86_64                                                                  1/1

Installed:
  whois.x86_64 0:5.1.1-2.el7

Complete!
```

### Find the name server for Google using whois:

```
[root@masterhost06 ~]# whois google.com | grep -i "name server"
   Name Server: NS1.GOOGLE.COM
   Name Server: NS2.GOOGLE.COM
   Name Server: NS3.GOOGLE.COM
   Name Server: NS4.GOOGLE.COM
```

### We can check our current cache file:

```
[root@masterhost06 ~]# unbound-control dump_cache
START_RRSET_CACHE
END_RRSET_CACHE
START_MSG_CACHE
END_MSG_CACHE
EOF
```

### There is currently no cache at the moment.

### We can check the cuurent status of the cache server:

```
[root@masterhost06 ~]# firewall-cmd --list-services
ssh dns
[root@masterhost06 ~]# unbound-control status
version: 1.6.6
verbosity: 1
threads: 4
modules: 3 [ ipsecmod validator iterator ]
uptime: 145 seconds
options: reuseport control(ssl)
unbound (pid 9256) is running...
```

### The unbound-control command has some useful options and functionality:

```
[root@masterhost06 ~]# unbound-control lookup google.com
The following name servers are used for lookup of google.com.
forwarding request:
Delegation with 0 names, of which 0 can be examined to query further addresses.
It provides 2 IP addresses.
8.8.8.8         	rto 59 msec, ttl 623, ping 15 var 11 rtt 59, tA 0, tAAAA 0, tother 0, EDNS 0 probed.
8.8.4.4         	rto 64 msec, ttl 623, ping 16 var 12 rtt 64, tA 0, tAAAA 0, tother 0, EDNS 0 probed.
```

### We have now likely produced some records in our cache file, let's query the contents once more:

```
[root@masterhost06 ~]# unbound-control dump_cache
START_RRSET_CACHE
;rrset nsec_apex 85436 1 1 2 0
.	85436	IN	NSEC	aaa. NS SOA RRSIG NSEC DNSKEY
.	85436	IN	RRSIG	NSEC 8 0 86400 20180719170000 20180706160000 41656 . xmA4SX6ld86vv0vtoA0xegDZu3Z+lkq8T8M0poF5jSVfRcgexLOQxFoTcpuV95rXOGlZTZWkoqAyJ81DiNhkQPFmzVJ6srX9WbrfxDb92kaxweLtvJquFpsAHQbsv5GfYLqGBEDWALCJJ+/IMXf2cl/sTzhcSqF7ma304mDOWgfHxRYXYopwmp9EQY3ZNLArQKvHG3XelOknc6m2sJCJV9YQ94yEcx91QePkRURHg82pMm8CygNhxtkqxH2yByJkiY2iC4kwmrp6J1UEvXxAe3LsKHsc+CV/3uNGEsWd5yu82XwfIWhjWFZYh2bbhDKAjgeNztVwDcMetrEaLJA/+g== ;{id = 41656}
;rrset 85190 4 1 5 0
.	115615	IN	DNSKEY	256 3 8 AwEAAdU4aKlDgEpXWWpH5aXHJZI1Vm9Cm42mGAsqkz3akFctS6zsZHC3pNNMug99fKa7OW+tRHIwZEc//mX8Jt6bcw5bPgRHG6u2eT8vUpbXDPVs1ICGR6FhlwFWEOyxbIIiDfd7Eq6eALk5RNcauyE+/ZP+VdrhWZDeEWZRrPBLjByBWTHl+v/f+xvTJ3Stcq2tEqnzS2CCOr6RTJepprYhu+5Yl6aRZmEVBK27WCW1Zrk1LekJvJXfcyKSKk19C5M5JWX58px6nB1IS0pMs6aCIK2yaQQVNUEg9XyQzBSv/rMxVNNy3VAqOjvh+OASpLMm4GECbSSe8jtjwG0I78sfMZc= ;{id = 39570 (zsk), size = 2048b}
.	115615	IN	DNSKEY	256 3 8 AwEAAfaifSqh+9ItxYRCwuiY0FY2NkaEwd/zmyVvakixDgTOkgG/PUzlEauAiKzlxGwezjqbKFPSwrY3qHmbbsSTY6G8hZtna8k26eCwy59Chh573cu8qtBkmUIXMYG3fSdlUReP+uhBWBfKI2aGwhRmQYR0zSmg7PGOde34c/rOItK1ebJhjTAJ6TmnON7qMfk/lKvH4qOvYtzstLhr7Pn9ZOVLx/WUKQpU/nEyFyTduRbz1nZqkp6yMuHwWVsABK8lUYXSaUrDAsuMSldhafmR/A15BxNhv9M7mzJj7UH2RVME9JbYinBEzWwW9GpnY+ZmBWgZiRVTaDuemCTJ5ZJWLRs= ;{id = 41656 (zsk), size = 2048b}
.	115615	IN	DNSKEY	257 3 8 AwEAAagAIKlVZrpC6Ia7gEzahOR+9W29euxhJhVVLOyQbSEW0O8gcCjFFVQUTf6v58fLjwBd0YI0EzrAcQqBGCzh/RStIoO8g0NfnfL2MTJRkxoXbfDaUeVPQuYEhg37NZWAJQ9VnMVDxP/VHL496M/QZxkjf5/Efucp2gaDX6RS6CXpoY68LsvPVjR0ZSwzz1apAzvN9dlzEheX7ICJBBtuA6G3LQpzW5hOA2hzCTMjJPJ8LbqF6dsV6DoBQzgul0sGIcGOYl7OyQdXfZ57relSQageu+ipAdTTJ25AsRTAoub8ONGcLmqrAmRLKBP1dfwhYB4N7knNnulqQxA+Uk1ihz0= ;{id = 19036 (ksk), size = 2048b}
.	115615	IN	DNSKEY	257 3 8 AwEAAaz/tAm8yTn4Mfeh5eyI96WSVexTBAvkMgJzkKTOiW1vkIbzxeF3+/4RgWOq7HrxRixHlFlExOLAJr5emLvN7SWXgnLh4+B5xQlNVz8Og8kvArMtNROxVQuCaSnIDdD5LKyWbRd2n9WGe2R8PzgCmr3EgVLrjyBxWezF0jLHwVN8efS3rCj/EWgvIWgb9tarpVUDK/b58Da+sqqls3eNbuv7pr+eoZG+SrDK6nWeL3c6H5Apxz7LjVc1uTIdsIXxuOLYA4/ilBmSVIzuDWfdRUfhHdY6+cn8HFRm+2hM8AnXGXws9555KrUB5qihylGa8subX2Nn6UwNR1AkUTV74bU= ;{id = 20326 (ksk), size = 2048b}
.	115615	IN	RRSIG	DNSKEY 8 0 172800 20180722000000 20180701000000 19036 . I3gcUYC8GwZmq4TTEy1SFxvFyZuCvGCiosqlQ1MPBbsLz4GXUkw7jcbbtgBfsZXPTtxIuPnOLamt6yHHGadAUxcDdH6YX9ots2yS5n1Qy56maFyboEcFc77V9945b16lCKwyitks+7tGDanG1tb/XhNgpGFw0+rurdb3p6wKvQvMUbyQIu8u0WGbcR1SiFD6zLlJdD6upIYQyYDKMI6YgYP2UQqbGiEkbmdPWOKHvKDc85+BRvcaRng7+SYk1aCFRqzVg/m9MNSaeSgOP0Nwo3YvTFRDy/jMEyWAoujNopUCR8lFPqcV9DCLXh2tFSH7G8fdeB3234RyPPx6VK/xCQ== ;{id = 19036}
;rrset 84994 1 1 2 0
loans.	84994	IN	NSEC	locker. NS DS RRSIG NSEC
loans.	84994	IN	RRSIG	NSEC 8 1 86400 20180719170000 20180706160000 41656 . EMfZbjlN0Kq+1Ywu3bKFKTCBCvl3lX+z6Sg6ZDhQ+DTkSNZNIKebDZ3cw5qozjsm4Rj7JBO3jZ1GacR8agxodtXnvJPzra1Y9512hnWQLaV1SQu7/0Lh1OYw7z6pQCFrHPAiY6ISL2ZhZ3TZ28stikryV8ajATkUoDQ890SzTBDEeH7d9sfdAQdz02MXjQtSxqEkdujdqct/r6Hd2j/D9JXNFp3X52RZwyQ9JDwD664U3PbrvcunheMyx4JypRljkVB7VTc2upkRdYWmUeBqZSeUwuZsaJtRAWzVnMUy1VzljYA7d0hx44r9j10kdPe7lzAb6FQ8Z+G1kT8fUunbsA== ;{id = 41656}
END_RRSET_CACHE
START_MSG_CACHE
msg . IN DNSKEY 33152 1 85190 0 1 0 0
. IN DNSKEY 0
END_MSG_CACHE
EOF

```

## Client Configuration:

### We will be using vm07 (10.0.2.7) as our client.

### We will be configuring masterhost06 as our cache-only DNS server.  First we must edit our network interface file:

`[root@vm07 ~]# vim /etc/sysconfig/network-scripts/ifcfg-enp0s3`

### Original:

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=fad7c00c-cff2-43bd-b76e-1f01c409a8a7
DEVICE=enp0s3
ONBOOT=yes
IPADDR=10.0.2.7
GATEWAY=10.0.2.1
NETMASK=255.255.255.0
DOMAIN=jdillon.local
DNS1=8.8.8.8
```

### Edited (we are specifically changing the DNS1 parameter):

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=fad7c00c-cff2-43bd-b76e-1f01c409a8a7
DEVICE=enp0s3
ONBOOT=yes
IPADDR=10.0.2.7
GATEWAY=10.0.2.1
NETMASK=255.255.255.0
DOMAIN=jdillon.local
DNS1=10.0.2.6
```

### Restart the network:

```
[root@vm07 ~]# cat /etc/resolv.conf
search jdillon.local
nameserver 10.0.2.6
```

### We see that 10.0.2.6 has been updated as our current nameserver

### We should ensure that we are able to resolve names through 10.0.2.6:

```
[root@vm07 ~]# dig google.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 5455
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.			IN	A

;; Query time: 0 msec
;; SERVER: 10.0.2.6#53(10.0.2.6)
;; WHEN: Tue Jul 03 01:03:42 EDT 2018
;; MSG SIZE  rcvd: 39
```

### We see that Google.com is being resolved by our nameserver:

<hr><hr>
