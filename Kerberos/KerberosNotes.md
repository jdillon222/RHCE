# Kerberos Notes:
<hr><hr>

## Directory Contents (Please configure steps in this order):

* 1.) [Kerberos Server Configuration](KDC_Server_Config.md)
* 2.) [Kerberos_Client_Configuration](Kdc_Client_Config.md)

<hr><hr>

* Kerberos is a network authentication protocol created by MIT
* It uses symmetric-key cryptography to authenticate users to network services,
 so that passwords are never actually sent over the network.
* The authentication mechanism will be done through the use of 'tickets'
* The KDC server (Key Distribution Center) will be responsible for giving the 
 users their tickets, therefore it is technically an SSO (Single Sign On) system.
* The KDC server uses its own database to store the passwords of all users.
* However, it does not store user information (Shell, home dir...etc) like LDAP.
* Kerberos simply provides the authentication process.

---
### Terms:
------
* **Realm:**
  * The administrative Domain, written in upper-case (EXAMPLE.COM)
* **Principle:**
  * An entry in the authentication DB of Kerberos like (nfs/nfs.example.com)
* **KDC (Key Distribution Center):**
  * Consisting of three components:
     - 1). DB: to host all principles information
     - 2). Auth Server: to initialize the authentication process
     - 3). TGS: (Ticket Granting Ticket) To generate encrypted keys and to be 
         sent to the user.
* **Ticket:**
  * The client receives this ticket from the KDC and presents it to the network
   service to access.

---
### The Authentication Process:
---------------------------
* **User-->**(Sends principal identity and credentials to the KDC [TGT request])
* **KDC-->**(Queries the DB, checking for the principal)
* **KDC-->**(Receives validation from DB, creates TGT and wraps it in the 
         principal's user key)
* **TGT/principal** user key combination is sent to a credentials cache
* **TGT** is decrypted, and stored in this cache
* **Decrypted TGT** can be queried in the keytab, by Kerberos aware applications and services

---
### Process Summary:
---------------
* 1).  Client requests access to a service (e.g. NFS, SSH).
* 2).  Client will request TGS (ticket granting session) from the KDC server.
* 3).  KDC server will grant the user encrypted TGT (Ticket Granting Ticket).
* 4).  Client will present this TGT to the network service.
* 5).  Network service will verify the user's ticket.
* 6).  Now, the client can access the network service normally.

