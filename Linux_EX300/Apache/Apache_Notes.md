# Apache Server Notes:

<hr><hr>

Directory contents:

* 1.) [Simple Web-Site Creation](Creating_Website)
* 2.) [VirtualHost Configuration](VirtualHost_Configuration)
* 3.) [Private Directory Configuration](Private_Directory_Configuration)
* 4.) [Deploying CGI Applications](Deploying_CGI_Applications)
* 5.) [Managing Group Content](Group_Managed_Content)
* 6.) [Configuring TLS Security](TLS_Security)

<hr><hr>

## Understanding Apache Server:

* Apache configuration files are stored in /etc/httpd/
* The main configuration file is /etc/httpd/conf/httpd.conf
* You can create any supplemental configurations in the /etc/httpd/conf.d/ directory
* Apache is a modular application, meaning you can create a module to support a specific function.  All modules are stored in /etc/httpd/modules
* To enable module support, a line must be uncommented in httpd.conf:
  * Include conf.modules.d/*.conf
* And for supplemental configuration:
  * IncludeOptional conf.d/*.conf

## Apache Virtual Hosts:

* Several web sites can be created on the same server.
* There are two main types of Apache virtual hosts:
  * 1.) IP-Based Virtual Hosts:
    * Working with IP-based virtual hosts means you must have more than one network interface card in your server to handle high traffic.
    * You create a number of virtual hosts with different IP addresses, the amount listed in the server will specify the number of virtual hosts.
  * 2.) Name-Based Virtual Hosts:
    * Used to host multiple virtual sites on a single IP address.
    * This requires a DNS server, whose records must be updated for all virtual host names.
    * It is recommended to use network teaming for network redundancy and load-balancing.

<hr><hr>
