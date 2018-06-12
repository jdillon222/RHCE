# Masquerade Configuration:

<hr><hr>

### We can set up a masquerade rule for all, or specific network interfaces:

* All:

```
[root@masterhost06 ~]# firewall-cmd --permanent --add-masquerade
Warning: ALREADY_ENABLED: masquerade
success
[root@masterhost06 ~]# firewall-cmd --reload
success
```

### This will allow masquerading for all networkg interfaces.  We can select specific interfaces by working with rich-rules:

* Specific:

```
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

### We see that the masquerade option has been set to 'yes', with our specific masquerade rich-rule for network 10.0.2.0

<hr><hr>
