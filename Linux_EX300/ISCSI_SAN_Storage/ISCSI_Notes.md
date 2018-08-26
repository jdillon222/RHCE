# iSCSI SAN Storage Notes:

<hr><hr>

Directory Contents:

* 1.) [Configuring iSCSI Target & Initiator](iSCSI_Configuration)

<hr><hr>

## Understanding iSCSI:

* iSCSI (Internet Small Computer Systems Interface) Terms:
  * <strong>iSCSI Target</strong>: The iSCSI server that shares LUNs or disks.
  * <strong>iSCSI Initiator</strong>: The iSCSI client that accesses the shared LUNs or disks.
  * <strong>IQN</strong>: The iSCSI qualified name.  It is a unique name to identify target and initiator.
  * <strong>ACL</strong>: The access control list based on iSCSI IQN to grant access to the initiator IQNs.
  * <strong>LUN</strong>: The logical unit number. This is the block device that will be shared through the iSCSI target server.
  * <strong>Portal</strong>: The IP address and port number that the initiator will access the shared LUNs through.
  * <strong>TPG</strong>: The target portal group.  This is a list of IP addresses and port numbers that the iSCSI target will will listen for.

<hr>

* The shared LUN will be seen as a local device in the iSCSI initiator as /dev/sdb
* By default iSCSI listens onf port 3260/tcp
* iSCSI iniator package name: iscsi-initiator-utils
* iSCSI target package: targetcli
* The IQN configuration file on iSCSI initiator is located in /etc/iscsi/initiatorname.iscsi

<hr><hr>
