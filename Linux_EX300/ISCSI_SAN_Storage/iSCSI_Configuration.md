# Configuring iSCSI Target & Initiator:

<HR><HR>

### (Target will be on vm11 [10.0.2.11] & initiator will be on vm10 [10.0.2.10])

### On our target host, we will install required packages:

```
[root@vm11 ~]# yum -y install target*
Installed:
  targetd.noarch 0:0.8.6-1.el7

Dependency Installed:
  PyYAML.x86_64 0:3.10-11.el7                       lvm2-python-libs.x86_64 7:2.02.177-4.el7
  python-setproctitle.x86_64 0:1.1.6-5.el7

Updated:
  targetcli.noarch 0:2.1.fb46-6.el7_5

Dependency Updated:
  device-mapper.x86_64 7:1.02.146-4.el7                device-mapper-event.x86_64 7:1.02.146-4.el7
  device-mapper-event-libs.x86_64 7:1.02.146-4.el7     device-mapper-libs.x86_64 7:1.02.146-4.el7
  device-mapper-persistent-data.x86_64 0:0.7.3-3.el7   lvm2.x86_64 7:2.02.177-4.el7
  lvm2-libs.x86_64 7:2.02.177-4.el7
```

### Enable and start the target:

```
[root@vm11 ~]# systemctl enable target
Created symlink from /etc/systemd/system/multi-user.target.wants/target.service to /usr/lib/systemd/system/target.service.
[root@vm08 ~]# systemctl start target
```

### Allow iSCSI's default port (3260) through the firewall under tcp:

```
[root@vm08 ~]# firewall-cmd --permanent --add-port=3260/tcp
success
[root@vm08 ~]# firewall-cmd --reload
success
```

<hr>

### On the initiator (vm10), we will install necessary packages:

```
[root@vm10 ~]# yum search iscsi
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: centos-distro.1gservers.com
 * extras: repos.lax.quadranet.com
 * updates: mirrors.sonic.net
======================================== N/S matched: iscsi =========================================
iscsi-initiator-utils.x86_64 : iSCSI daemon and utility programs
iscsi-initiator-utils.i686 : iSCSI daemon and utility programs
iscsi-initiator-utils-devel.i686 : Development files for iscsi-initiator-utils
iscsi-initiator-utils-devel.x86_64 : Development files for iscsi-initiator-utils
iscsi-initiator-utils-iscsiuio.x86_64 : Userspace configuration daemon required for some iSCSI
                                      : hardware
libiscsi.i686 : iSCSI client library
libiscsi.x86_64 : iSCSI client library
libiscsi-devel.i686 : iSCSI client development libraries
libiscsi-devel.x86_64 : iSCSI client development libraries
libiscsi-utils.x86_64 : iSCSI Client Utilities
libvirt-daemon-driver-storage-iscsi.x86_64 : Storage driver plugin for iscsi
storaged-iscsi.x86_64 : Module for iSCSI
udisks2-iscsi.x86_64 : Module for iSCSI

  Name and summary matches only, use "search all" for everything.

```

### We will install the iscsi-initiator-utils package:

```
[root@vm10 ~]# yum -y install iscsi-initiator-utils.x86_64
Updated:
  iscsi-initiator-utils.x86_64 0:6.2.0.874-7.el7

Dependency Updated:
  iscsi-initiator-utils-iscsiuio.x86_64 0:6.2.0.874-7.el7

Complete!
```

### We will enable and start the iSCSI initiator service:

```
[root@vm10 ~]# systemctl enable iscsi
[root@vm10 ~]# systemctl start iscsi
```

### Allow port 3260 through the firewall:

```
[root@vm10 ~]# firewall-cmd --permanent --add-port=3260/tcp
success
[root@vm10 ~]# firewall-cmd --reload
success
```

### Now, we must change the IQN for the initiator in the iSCSI configuration file, this will follow a format of year-month, followed by our reverse fully qualified domain name, followed by a colon and the hostname:

`[root@vm10 ~]# vim /etc/iscsi/initiatorname.iscsi`

### Original:
```
InitiatorName=iqn.1994-05.com.redhat:ba15203feac3
```

### Edited:
```
InitiatorName=iqn.2018-07.local.jdillon:vm10
```

### Restart iSCSI:

`[root@vm10 ~]# systemctl restart iscsi`

<hr>

### On the target server, if we run the 'fdisk -l' command we can see our sd disk devices:

```
[root@vm11 ~]# fdisk -l

Disk /dev/sda: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000c583e

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048      616447      307200   83  Linux
/dev/sda2          616448    15296511     7340032   83  Linux
/dev/sda3        15296512    17393663     1048576   83  Linux
/dev/sda4        17393664    20971519     1788928    5  Extended
/dev/sda5        17395712    17981439      292864   83  Linux
/dev/sda6        17983488    19982335      999424   83  Linux
/dev/sda7        19982337    20969567      493615+  83  Linux
```

### We will be creating a logical volume group from physical partitions /dev/sda6 & 7

```
[root@vm11 ~]# pvcreate /dev/sda6 /dev/sda7
  Physical volume "/dev/sda6" successfully created.
  Physical volume "/dev/sda7" successfully created.
```

### Create a new logical volume group 'vg0' from our physical volumes:

```
[root@vm11 ~]# pvcreate /dev/sda6 /dev/sda7
  Physical volume "/dev/sda6" successfully created.
  Physical volume "/dev/sda7" successfully created.
[root@vm11 ~]# vgcreate vg0 /dev/sda6 /dev/sda7
  Volume group "vg0" successfully created
[root@vm11 ~]# vgs
  VG  #PV #LV #SN Attr   VSize  VFree
  vg0   2   0   0 wz--n- <1.42g <1.42g

```

### Create the actual logical volume 'lv0' from the group 'vg0':

```
[root@vm11 ~]# lvcreate -L .7G -n lv0 vg0
  Rounding up size to full physical extent 720.00 MiB
  Logical volume "lv0" created.
```

### Create another logical volume 'lv1' from the group 'vg0':

```
[root@vm11 ~]# lvcreate -L .7G -n lv1 vg0
  Rounding up size to full physical extent 720.00 MiB
  Logical volume "lv1" created.
[root@vm11 ~]# ls /dev/mapper
control  vg0-lv0  vg0-lv1
```

### We can initiate the target management console with the 'targetcli' command:

`[root@vm11 ~]# targetcli`

### Typing 'ls' within the management console shows us some configuration options:

```
Warning: Could not load preferences file /root/.targetcli/prefs.bin.
targetcli shell version 2.1.fb46
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
o- / ................................................................................................. [...]
  o- backstores ...................................................................................... [...]
  | o- block .......................................................................... [Storage Objects: 0]
  | o- fileio ......................................................................... [Storage Objects: 0]
  | o- pscsi .......................................................................... [Storage Objects: 0]
  | o- ramdisk ........................................................................ [Storage Objects: 0]
  o- iscsi .................................................................................... [Targets: 0]
  o- loopback ................................................................................. [Targets: 0]
/>
```

### We will be configuring 'block' device and 'fileio' in this example.  We will 'cd' to block:

```
/> cd ..
/> ls
o- / ................................................................................................. [...]
  o- backstores ...................................................................................... [...]
  | o- block .......................................................................... [Storage Objects: 0]
  | o- fileio ......................................................................... [Storage Objects: 0]
  | o- pscsi .......................................................................... [Storage Objects: 0]
  | o- ramdisk ........................................................................ [Storage Objects: 0]
  o- iscsi .................................................................................... [Targets: 0]
  o- loopback ................................................................................. [Targets: 0]
/> cd backstores
/backstores> cd block
/backstores/block>
```

### We will create two block devices from /dev/vg0/lv0 lv1:

```
/backstores/block> create block1 /dev/vg0/lv0
Created block storage object block1 using /dev/vg0/lv0.
/backstores/block> create block2 /dev/vg0/lv1
Created block storage object block2 using /dev/vg0/lv1.
```

### We should now be able to see our new block storage devices:

```
/backstores/block> ls
o- block .............................................................................. [Storage Objects: 2]
  o- block1 ............................................... [/dev/vg0/lv0 (720.0MiB) write-thru deactivated]
  | o- alua ............................................................................... [ALUA Groups: 1]
  |   o- default_tg_pt_gp ................................................... [ALUA state: Active/optimized]
  o- block2 ............................................... [/dev/vg0/lv1 (720.0MiB) write-thru deactivated]
    o- alua ............................................................................... [ALUA Groups: 1]
      o- default_tg_pt_gp ................................................... [ALUA state: Active/optimized]
```

### We move to the fileIO directory, and create fileIO of 5G by creating a file in the /root directory:

```
/backstores> ls
o- backstores ........................................................................................ [...]
  o- block ............................................................................ [Storage Objects: 2]
  | o- block1 ............................................. [/dev/vg0/lv0 (720.0MiB) write-thru deactivated]
  | | o- alua ............................................................................. [ALUA Groups: 1]
  | |   o- default_tg_pt_gp ................................................. [ALUA state: Active/optimized]
  | o- block2 ............................................. [/dev/vg0/lv1 (720.0MiB) write-thru deactivated]
  |   o- alua ............................................................................. [ALUA Groups: 1]
  |     o- default_tg_pt_gp ................................................. [ALUA state: Active/optimized]
  o- fileio ........................................................................... [Storage Objects: 1]
  | o- fileIO ............................................... [/root/fileIO (5.0GiB) write-back deactivated]
  |   o- alua ............................................................................. [ALUA Groups: 1]
  |     o- default_tg_pt_gp ................................................. [ALUA state: Active/optimized]
  o- pscsi ............................................................................ [Storage Objects: 0]
  o- ramdisk .......................................................................... [Storage Objects: 0]
/backstores> cd fileio
/backstores/fileio> create fileIO /root/fileIO 5G
Created fileio fileIO with size 5368709120
```

### Navigate back to the root directory, and then to the iscsi directory:

```
/backstores/fileio> cd ..
/backstores> cd ..
/> cd iscsi
/iscsi> ls
o- iscsi ...................................................................................... [Targets: 0]
```

### We will create an IQN for the target server:

```
/iscsi> create iqn.2018-07.local.jdillon:vm11
Created target iqn.2018-07.local.jdillon:vm11.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
```

### We must now create an ACL for the initiator:

```
/iscsi> ls
o- iscsi ...................................................................................... [Targets: 1]
  o- iqn.2018-07.local.jdillon:vm11 .............................................................. [TPGs: 1]
    o- tpg1 ......................................................................... [no-gen-acls, no-auth]
      o- acls .................................................................................... [ACLs: 0]
      o- luns .................................................................................... [LUNs: 0]
      o- portals .............................................................................. [Portals: 1]
        o- 0.0.0.0:3260 ............................................................................... [OK]
/iscsi> cd iqn*
No such path /iscsi/iqn
/iscsi> cd iqn.2018-07.local.jdillon:vm11
/iscsi/iqn.20....jdillon:vm11> cd tpg1
/iscsi/iqn.20...lon:vm11/tpg1> cd acls
/iscsi/iqn.20...m11/tpg1/acls> ls
o- acls .......................................................................................... [ACLs: 0]
```

### We must create a file matching the name of the IQN on the initiator:

```
/iscsi/iqn.20...m11/tpg1/acls> create iqn.2018-07.local.jdillon:vm10
Created Node ACL for iqn.2018-07.local.jdillon:vm10
```

### Now we must create our luns, and assign block1 and block2 for the initiator IQN:

```
/iscsi/iqn.20...m11/tpg1/acls> cd /
/> ls
o- / ................................................................................................. [...]
  o- backstores ...................................................................................... [...]
  | o- block .......................................................................... [Storage Objects: 2]
  | | o- block1 ........................................... [/dev/vg0/lv0 (720.0MiB) write-thru deactivated]
  | | | o- alua ........................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | | o- block2 ........................................... [/dev/vg0/lv1 (720.0MiB) write-thru deactivated]
  | |   o- alua ........................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | o- fileio ......................................................................... [Storage Objects: 1]
  | | o- fileIO ............................................. [/root/fileIO (5.0GiB) write-back deactivated]
  | |   o- alua ........................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | o- pscsi .......................................................................... [Storage Objects: 0]
  | o- ramdisk ........................................................................ [Storage Objects: 0]
  o- iscsi .................................................................................... [Targets: 1]
  | o- iqn.2018-07.local.jdillon:vm11 ............................................................ [TPGs: 1]
  |   o- tpg1 ....................................................................... [no-gen-acls, no-auth]
  |     o- acls .................................................................................. [ACLs: 1]
  |     | o- iqn.2018-07.local.jdillon:vm10 ............................................... [Mapped LUNs: 0]
  |     o- luns .................................................................................. [LUNs: 0]
  |     o- portals ............................................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ............................................................................. [OK]
  o- loopback ................................................................................. [Targets: 0]
/> cd iscsi/iqn.2018-07.local.jdillon:vm11/tpg1/luns
/iscsi/iqn.20...m11/tpg1/luns>
```

### If we type 'create' and tab complete, we can see options for creating new elements:

```
/iscsi/iqn.20...m11/tpg1/luns> create
anaconda-ks.cfg            /backstores/block/block1   /backstores/block/block2   /backstores/fileio/fileIO
Desktop/                   Documents/                 Downloads/                 fileIO
initial-setup-ks.cfg       Music/                     Pictures/                  Public/
Templates/                 Videos/                    add_mapped_luns=           lun=
storage_object=
```

### We will create block1 and block2:

```
/iscsi/iqn.20...m11/tpg1/luns> create /backstores/block/block1
Created LUN 0.
Created LUN 0->0 mapping in node ACL iqn.2018-07.local.jdillon:vm10
/iscsi/iqn.20...m11/tpg1/luns> create /backstores/block/block2
Created LUN 1.
Created LUN 1->1 mapping in node ACL iqn.2018-07.local.jdillon:vm10
```

### We will also create a lun for our fileIO:

```
/iscsi/iqn.20...m11/tpg1/luns> create /backstores/fileio/fileIO
Created LUN 2.
Created LUN 2->2 mapping in node ACL iqn.2018-07.local.jdillon:vm10
```

### If we go back to our root directory, we should now see three luns created under the initiator server:

```
/iscsi/iqn.20...m11/tpg1/luns> cd /
/> ls
o- / ................................................................................................. [...]
  o- backstores ...................................................................................... [...]
  | o- block .......................................................................... [Storage Objects: 2]
  | | o- block1 ............................................. [/dev/vg0/lv0 (720.0MiB) write-thru activated]
  | | | o- alua ........................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | | o- block2 ............................................. [/dev/vg0/lv1 (720.0MiB) write-thru activated]
  | |   o- alua ........................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | o- fileio ......................................................................... [Storage Objects: 1]
  | | o- fileIO ............................................... [/root/fileIO (5.0GiB) write-back activated]
  | |   o- alua ........................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | o- pscsi .......................................................................... [Storage Objects: 0]
  | o- ramdisk ........................................................................ [Storage Objects: 0]
  o- iscsi .................................................................................... [Targets: 1]
  | o- iqn.2018-07.local.jdillon:vm11 ............................................................ [TPGs: 1]
  |   o- tpg1 ....................................................................... [no-gen-acls, no-auth]
  |     o- acls .................................................................................. [ACLs: 1]
  |     | o- iqn.2018-07.local.jdillon:vm10 ............................................... [Mapped LUNs: 3]
  |     |   o- mapped_lun0 ........................................................ [lun0 block/block1 (rw)]
  |     |   o- mapped_lun1 ........................................................ [lun1 block/block2 (rw)]
  |     |   o- mapped_lun2 ....................................................... [lun2 fileio/fileIO (rw)]
  |     o- luns .................................................................................. [LUNs: 3]
  |     | o- lun0 ......................................... [block/block1 (/dev/vg0/lv0) (default_tg_pt_gp)]
  |     | o- lun1 ......................................... [block/block2 (/dev/vg0/lv1) (default_tg_pt_gp)]
  |     | o- lun2 ........................................ [fileio/fileIO (/root/fileIO) (default_tg_pt_gp)]
  |     o- portals ............................................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ............................................................................. [OK]
  o- loopback ................................................................................. [Targets: 0]
```

### Finally, we must create a portal to connect to the target server:

```
/iscsi/iqn.20...m11/tpg1/luns> cd /
/> ls
o- / ................................................................................................. [...]
  o- backstores ...................................................................................... [...]
  | o- block .......................................................................... [Storage Objects: 2]
  | | o- block1 ............................................. [/dev/vg0/lv0 (720.0MiB) write-thru activated]
  | | | o- alua ........................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | | o- block2 ............................................. [/dev/vg0/lv1 (720.0MiB) write-thru activated]
  | |   o- alua ........................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | o- fileio ......................................................................... [Storage Objects: 1]
  | | o- fileIO ............................................... [/root/fileIO (5.0GiB) write-back activated]
  | |   o- alua ........................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | o- pscsi .......................................................................... [Storage Objects: 0]
  | o- ramdisk ........................................................................ [Storage Objects: 0]
  o- iscsi .................................................................................... [Targets: 1]
  | o- iqn.2018-07.local.jdillon:vm11 ............................................................ [TPGs: 1]
  |   o- tpg1 ....................................................................... [no-gen-acls, no-auth]
  |     o- acls .................................................................................. [ACLs: 1]
  |     | o- iqn.2018-07.local.jdillon:vm10 ............................................... [Mapped LUNs: 3]
  |     |   o- mapped_lun0 ........................................................ [lun0 block/block1 (rw)]
  |     |   o- mapped_lun1 ........................................................ [lun1 block/block2 (rw)]
  |     |   o- mapped_lun2 ....................................................... [lun2 fileio/fileIO (rw)]
  |     o- luns .................................................................................. [LUNs: 3]
  |     | o- lun0 ......................................... [block/block1 (/dev/vg0/lv0) (default_tg_pt_gp)]
  |     | o- lun1 ......................................... [block/block2 (/dev/vg0/lv1) (default_tg_pt_gp)]
  |     | o- lun2 ........................................ [fileio/fileIO (/root/fileIO) (default_tg_pt_gp)]
  |     o- portals ............................................................................ [Portals: 1]
  |       o- 0.0.0.0:3260 ............................................................................. [OK]
  o- loopback ................................................................................. [Targets: 0]
/> cd iscsi/iqn.2018-07.local.jdillon:vm11/tpg1/portals
/iscsi/iqn.20.../tpg1/portals> ls
o- portals .................................................................................... [Portals: 1]
  o- 0.0.0.0:3260 ..................................................................................... [OK]
```

### We will delete this default IP and port:

```
/iscsi/iqn.20.../tpg1/portals> delete ip_address=0.0.0.0 ip_port=3260
Deleted network portal 0.0.0.0:3260
```

### Create an IP for the initiator:

```
/iscsi/iqn.20.../tpg1/portals> create 10.0.2.10
Using default IP port 3260
Created network portal 10.0.2.10:3260.
```

### We can now view our completed configuration:

```
/iscsi/iqn.20.../tpg1/portals> cd /
/> ls
o- / ................................................................................................. [...]
  o- backstores ...................................................................................... [...]
  | o- block .......................................................................... [Storage Objects: 2]
  | | o- block1 ............................................. [/dev/vg0/lv0 (720.0MiB) write-thru activated]
  | | | o- alua ........................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | | o- block2 ............................................. [/dev/vg0/lv1 (720.0MiB) write-thru activated]
  | |   o- alua ........................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | o- fileio ......................................................................... [Storage Objects: 1]
  | | o- fileIO ............................................... [/root/fileIO (5.0GiB) write-back activated]
  | |   o- alua ........................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ............................................... [ALUA state: Active/optimized]
  | o- pscsi .......................................................................... [Storage Objects: 0]
  | o- ramdisk ........................................................................ [Storage Objects: 0]
  o- iscsi .................................................................................... [Targets: 1]
  | o- iqn.2018-07.local.jdillon:vm11 ............................................................ [TPGs: 1]
  |   o- tpg1 ....................................................................... [no-gen-acls, no-auth]
  |     o- acls .................................................................................. [ACLs: 1]
  |     | o- iqn.2018-07.local.jdillon:vm10 ............................................... [Mapped LUNs: 3]
  |     |   o- mapped_lun0 ........................................................ [lun0 block/block1 (rw)]
  |     |   o- mapped_lun1 ........................................................ [lun1 block/block2 (rw)]
  |     |   o- mapped_lun2 ....................................................... [lun2 fileio/fileIO (rw)]
  |     o- luns .................................................................................. [LUNs: 3]
  |     | o- lun0 ......................................... [block/block1 (/dev/vg0/lv0) (default_tg_pt_gp)]
  |     | o- lun1 ......................................... [block/block2 (/dev/vg0/lv1) (default_tg_pt_gp)]
  |     | o- lun2 ........................................ [fileio/fileIO (/root/fileIO) (default_tg_pt_gp)]
  |     o- portals ............................................................................ [Portals: 1]
  |       o- 10.0.2.10:3260 ........................................................................... [OK]
  o- loopback ................................................................................. [Targets: 0]
```

### We can see that both of our block devices are currently activated.

### We can now exit the interface and restart the target:

```
/> exit
Global pref auto_save_on_exit=true
Last 10 configs saved in /etc/target/backup/.
Configuration saved to /etc/target/saveconfig.json
[root@vm11 ~]# systemctl restart target
```

### We should now be able to connect the initiator to the target, in order to access the block devices.

<hr>

### Back on the initiator (10.0.2.10), we will connect to the target and access the block devices.

### iSCSI has a useful utility, named 'iscsiadm'.  The examples section of the man page is very thorough.

### We must first discover the available connection:

```
[root@vm10 ~]# iscsiadm --mode discovery --type sendtargets --portal 10.0.2.11
10.0.2.11:3260,1 iqn.2018-07.local.jdillon:vm11
```

### We should now be able to connect to the target:

```
[root@vm10 ~]# iscsiadm --mode discovery --type sendtargets --portal 10.0.2.11
10.0.2.11:3260,1 iqn.2018-07.local.jdillon:vm11
[root@vm10 ~]# iscsiadm -m node -T iqn.2018-07.local.jdillon:vm11 -p 10.0.2.11:3260 --login
Logging in to [iface: default, target: iqn.2018-07.local.jdillon:vm11, portal: 10.0.2.11,3260] (multiple)
Login to [iface: default, target: iqn.2018-07.local.jdillon:vm11, portal: 10.0.2.11,3260] successful.
```

### We can verify that the block devices from the target, are now available as disk devices:

```
[root@vm10 ~]# fdisk -l

Disk /dev/sda: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b9c90

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    11511807     5242880   83  Linux
/dev/sda3        11511808    17803263     3145728   83  Linux
/dev/sda4        17803264    20971519     1584128    5  Extended
/dev/sda5        17805312    18829311      512000   82  Linux swap / Solaris
/dev/sda6        18831360    20969471     1069056   83  Linux

Disk /dev/sdb: 754 MB, 754974720 bytes, 1474560 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 33550336 bytes


Disk /dev/sdc: 5368 MB, 5368709120 bytes, 10485760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 4194304 bytes


Disk /dev/sdd: 754 MB, 754974720 bytes, 1474560 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 33550336 bytes
```

### We see that we now have a /dev/sdb & sdd device at 754Mib, as well as a 5G /dev/sdc device created by our fileIO on the target

### Install 'lsscsi':

```
[root@vm11 ~]# yum -y install lsscsi
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.scalabledns.com
 * extras: mirror.web-ster.com
 * updates: mirrors.cat.pdx.edu
Package lsscsi-0.27-6.el7.x86_64 already installed and latest version
Nothing to do
```

### We can use the 'lsscsi' command to view our connected block devices and files from the server:

```
[root@vm10 ~]# lsscsi
[1:0:0:0]    cd/dvd  VBOX     CD-ROM           1.0   /dev/sr0
[2:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sda
[3:0:0:0]    disk    LIO-ORG  block1           4.0   /dev/sdb
[3:0:0:1]    disk    LIO-ORG  block2           4.0   /dev/sdd
[3:0:0:2]    disk    LIO-ORG  fileIO           4.0   /dev/sdc
```

<hr>

### Back on the target (10.0.2.11), we can view our current connections from the management cli:

```
[root@vm11 ~]# targetcli
targetcli shell version 2.1.fb46
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/iscsi/iqn.20.../tpg1/portals> cd /
/> sessions
alias: vm10.jdillon.local	sid: 1 type: Normal session-state: LOGGED_IN
/>
```

### This shows the current open session that the target has with initiator (10.0.2.10)

```
/> exit
Global pref auto_save_on_exit=true
Last 10 configs saved in /etc/target/backup/.
Configuration saved to /etc/target/saveconfig.json
```

<hr>

### Back on our initiator (10.0.2.10), we can logout of the iscsi connection:

```
[root@vm10 ~]# iscsiadm -m node -T iqn.2018-07.local.jdillon:vm11 -p 10.0.2.11:3260 --logout
Logging out of session [sid: 1, target: iqn.2018-07.local.jdillon:vm11, portal: 10.0.2.11,3260]
Logout of [sid: 1, target: iqn.2018-07.local.jdillon:vm11, portal: 10.0.2.11,3260] successful.
```

### Using fdisk, we can see that we no longer have our block and file devices:

```
[root@vm10 ~]# fdisk -l

Disk /dev/sda: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b9c90

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    11511807     5242880   83  Linux
/dev/sda3        11511808    17803263     3145728   83  Linux
/dev/sda4        17803264    20971519     1584128    5  Extended
/dev/sda5        17805312    18829311      512000   82  Linux swap / Solaris
/dev/sda6        18831360    20969471     1069056   83  Linux
```

<hr><hr>
