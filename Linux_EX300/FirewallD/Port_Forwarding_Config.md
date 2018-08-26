# Port Forwarding Configuration:

<hr><hr>

### In order to configure port forwarding, masquerading must be added to the firewall:

```
[root@masterhost06 ~]# firewall-cmd --permanent --add-masquerade
Warning: ALREADY_ENABLED: masquerade
success
```

### We are now ready to add a forwarded port (https port 80 for example):

```
[root@masterhost06 ~]# firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=192.168.56.1
success
[root@masterhost06 ~]# firewall-cmd --reload
success
[root@masterhost06 ~]# firewall-cmd --list-all
external (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s9
  sources:
  services: ssh
  ports:
  protocols:
  masquerade: yes
  forward-ports: port=80:proto=tcp:toport=8080:toaddr=192.168.56.1
	port=80:proto=tcp:toport=8080:toaddr=192.168.56.1
  source-ports:
  icmp-blocks:
  rich rules:
	rule family="ipv4" source address="10.0.2.7" accept
	rule family="ipv4" source address="10.0.2.0/24" masquerade
```

### In this example, 192.168.56.1 is our external/forward-facing network.  We are forwarding our https port to this network.  Therefore, an internet user will be able to access our forwarded port 80 from outside the private 10.0.2.0 network.

### We can remove our forwarded port simply:

```
[root@masterhost06 ~]# firewall-cmd --permanent --remove-forward-port=port=80:proto=tcp:toport=8080:toaddr=192.168.56.1
success
[root@masterhost06 ~]# firewall-cmd --reload
success
[root@masterhost06 ~]# firewall-cmd --list-all
external (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s9
  sources:
  services: ssh
  ports:
  protocols:
  masquerade: yes
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
	rule family="ipv4" source address="10.0.2.7" accept
	rule family="ipv4" source address="10.0.2.0/24" masquerade
```

<hr><hr>
