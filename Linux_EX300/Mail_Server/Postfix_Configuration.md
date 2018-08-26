# Postfix Configuration:
<hr><hr>

### (Postfix mail server will live on vm09 [10.0.2.09] & client will live on vm10 [10.0.2.10])

### We will be configuring Postfix, to forward all e-mails to a central mail server.

### Enable Postfix service on vm09:

`[root@vm09 ~]# systemctl enable postfix`

### Add a firewall rule for smtp:

```
[root@vm09 ~]# firewall-cmd --permanent --add-service=smtp
success
[root@vm09 ~]# firewall-cmd --reload
success
```

### We need to change a few parameters in the configuration file /etc/postfix/main.cf:

`[root@vm09 ~]# vim /etc/postfix/main.cf`

```
inet_interfaces = all
#inet_interfaces = localhost
inet_protocols = ipv4
myorigin = $mydomain
#mydestination = $myhostname, localhost.$mydomain, localhost
```

### To summarize config changes above:
* Uncomment the <strong>inet_interfaces = all</strong> parameter
* Comment out the <strong>inet_interfaces = localhost</strong> parameter
* Change the <strong>inet_protocols</strong> parameter to <strong>ipv4</strong>
* Uncomment the <strong>myorigin = $mydomain</strong> parameter
* Comment out the <strong>mydestination</strong> parameter

### Save the file, and restart Postfix:

`[root@vm09 ~]# systemctl restart postfix`

### We can check syntax errors using 'postfix check':

```
[root@vm09 ~]# postfix check
[root@vm09 ~]#
``` 

### We are now ready to configure our client vm10:

<hr>

### Open the configuration file /etc/postfix/main.cf and change the following parameters:

`[root@vm10 ~]# vim /etc/postfix/main.cf`

```
inet_interfaces = loopback-only
inet_protocols = ipv4
myorigin = $mydomain
mynetworks = 127.0.0.1/24
relayhost = 10.0.2.9
```

### Start and enable Postfix:

```
[root@vm10 ~]# systemctl start postfix
[root@vm10 ~]# systemctl enable postfix
```

### We can check the last log-lines pertaining to Postfix for any errors:

```
[root@vm10 ~]# tail /var/log/maillog
Jul 16 16:33:14 vm10 postfix/postfix-script[3585]: stopping the Postfix mail system
Jul 16 16:33:14 vm10 postfix/master[1190]: terminating on signal 15
Jul 16 16:33:14 vm10 postfix/postfix-script[3662]: starting the Postfix mail system
Jul 16 16:33:14 vm10 postfix/master[3664]: daemon started -- version 2.10.1, configuration /etc/postfix
```

### A quick check for configuration errors on the client:

```
[root@vm10 ~]# postfix check
[root@vm10 ~]#
```

### We are now ready to test our mail system by sending a message to the server:

```
[root@vm10 ~]# mail -s "test mail" root@vm09.jdillon.local < .
Null message body; hope that's ok
```

### We can now move back to our mail server vm09, to test receipt of the message.

<hr>

### On vm09:

```
[root@vm09 ~]# mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 1 message 1 new
>N  1 root                  Mon Jul 16 15:38  21/783   "test mail"
& 1
Message  1:
From root@jdillon.local  Mon Jul 16 15:38:15 2018
Return-Path: <root@jdillon.local>
X-Original-To: root@vm09.jdillon.local
Delivered-To: root@vm09.jdillon.local
Date: Mon, 16 Jul 2018 16:40:44 -0400
To: root@vm09.jdillon.local
Subject: test mail
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
From: root@jdillon.local (root)
Status: R


New mail has arrived.
Loaded 1 new message
 N  2 root                  Mon Jul 16 15:42  21/783   "test mail"
& 2
Message  2:
From root@jdillon.local  Mon Jul 16 15:42:45 2018
Return-Path: <root@jdillon.local>
X-Original-To: root@vm09.jdillon.local
Delivered-To: root@vm09.jdillon.local
Date: Mon, 16 Jul 2018 16:37:01 -0400
To: root@vm09.jdillon.local
Subject: test mail
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
From: root@jdillon.local (root)
Status: R
```

### We can check the current mail que via commands:

```
[root@vm10 ~]# mailq
Mail queue is empty
[root@vm10 ~]# posqueue -p
-bash: posqueue: command not found
[root@vm10 ~]# postqueue -p
Mail queue is empty
```

### If our queue was full, we could flush it with the following command:

```
[root@vm10 ~]# postqueue -f
[root@vm10 ~]#
```
<hr><hr>
