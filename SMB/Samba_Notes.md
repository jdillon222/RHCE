# Samba Server Notes:
<hr><hr>

## Directory Contents (Please configure steps in this order):

* 1.) [Samba Server Configuration](Samba_Server_Config.md)
* 2.) [Samba Client Configuration](Samba_Client_Config.md)
<hr><hr>

## Overview:

* Samba is an open source project, using the Common Internet File System (CIFS)
* It allows users to share files and directories between different operating systems (Windows & Linux)
* Main configuration file: /etc/samba/smb.conf
* Client is able to mount a Samba share using the 'cifs-utils' and 'samba-client' packages
* Samba uses nmb service for NetBIOS name or naming service
* We can use the 'smbpasswd -a' command to create a password the a Samba user
* Sambe can be connected to from the client via the 'smbclient -L' command
* Default listening ports are 139 and 445
* 'smbstatus' queries general information about the Samba server
* 'testparm' allows a user or admin to check parameters pertaining to the smb.conf config file
