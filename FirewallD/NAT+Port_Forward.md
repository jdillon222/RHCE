# NAT & Port-Forwarding:

<hr><hr>

## Masquerading:

* In order to configure Network Address Translation (NAT), we will need to have two network interfaces on our server in order to forward traffic from one interface to another.
* One interface will need to have public, while the other should be configured with private IP configuration
* Within the /etc/sysctl.conf file, the following line needs to be added (this is altering the kernel parameter):
  `net.ipv4.ip_forward=1`
* Masquerading must then be activated within the firewall
  `firewall-cmd --permanent --add-masquerade`
* This will allow for a host configured with a private IP to be reachable from the external network, serving as the forward-facing IP address.  The external network will access the private IP via a specific port, set up for access
* This works for Ipv4 only
* Rich-rules can be used to configure masquerading for a specific network in our server:
  `firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=10.0.2.0/24 masquerade'`

## Port Forwarding:

* We can use port forwarding in order to make specific services available on the internet
* It is done using masquerading
* It allows a host on the private interface to send packets to the public interface based on port number
* The configuration can be fulfilled through two steps:
  * 1). Allow the masquerade option
  * 2). Add `--add-forward-port` in our FirewallD rich-rule

