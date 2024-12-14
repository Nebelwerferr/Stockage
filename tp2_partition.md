CREDENTIALS : rocky / aze+123
UFW INSTALLE SUR TOUTES LES VM

# II. SAN network

# 1. Storage machines

## A. Disks and RAID

### 🌞 Configurer des RAID

#### - Installer mdadm : 
```Bash
sudo dnf install mdadm
```

#### - Créer le raid : 

```Bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
sudo mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde
sudo mdadm --create /dev/md2 --level=1 --raid-devices=2 /dev/sdf /dev/sdg
```

 REPETER SUR LES DEUX VM


#### - Prouver : 

```Bash
[root@sto2 rocky]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0   20G  0 disk
└─md0         9:0    0   20G  0 raid1
sdc           8:32   0   20G  0 disk
└─md0         9:0    0   20G  0 raid1
sdd           8:48   0   20G  0 disk
└─md1         9:1    0   20G  0 raid1
sde           8:64   0   20G  0 disk
└─md1         9:1    0   20G  0 raid1
sdf           8:80   0   20G  0 disk
└─md2         9:2    0   20G  0 raid1
sdg           8:96   0   20G  0 disk
└─md2         9:2    0   20G  0 raid1
sr0          11:0    1 1024M  0 rom
```

## B. iSCSI target

### 🌞 Installer target :
```Bash
dnf install targetcli
```

### 🌞 Démarrer le service target :

```Bash
systemctl start target

#Démarrage:
systemctl enable target
```

### 🌞 Configurer les targets iSCSI :

#### - Commandes : 

```Bash
/backstores/fileio create name=data-chunk1 file_or_dev=/dev/md0
/backstores/fileio create name=data-chunk2 file_or_dev=/dev/md1
/backstores/fileio create name=data-chunk3 file_or_dev=/dev/md2


/iscsi create iqn.2024-12.tp2.b3:data-chunk1
/iscsi create iqn.2024-12.tp2.b3:data-chunk2
/iscsi create iqn.2024-12.tp2.b3:data-chunk3


/iscsi/iqn.2024-12.tp2.b3:data-chunk1/tpg1/acls create iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
/iscsi/iqn.2024-12.tp2.b3:data-chunk2/tpg1/acls create iqn.2024-12.tp2.b3:data-chunk1:chunk2-initiator
/iscsi/iqn.2024-12.tp2.b3:data-chunk3/tpg1/acls create iqn.2024-12.tp2.b3:data-chunk1:chunk3-initiator

/iscsi/iqn.2024-12.tp2.b3:data-chunk1/tpg1/luns/ create /backstores/fileio/data-chunk1
/iscsi/iqn.2024-12.tp2.b3:data-chunk2/tpg1/luns/ create /backstores/fileio/data-chunk2
/iscsi/iqn.2024-12.tp2.b3:data-chunk3/tpg1/luns/ create /backstores/fileio/data-chunk3


saveconfig
```

#### - Résultat : 

```Bash
/> ls
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 0]
  | o- fileio ................................................................................................. [Storage Objects: 3]
  | | o- data-chunk1 ..................................................................... [/dev/md0 (20.0GiB) write-back activated]
  | | | o- alua ................................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | | o- data-chunk2 ..................................................................... [/dev/md1 (20.0GiB) write-back activated]
  | | | o- alua ................................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | | o- data-chunk3 ..................................................................... [/dev/md2 (20.0GiB) write-back activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 3]
  | o- iqn.2024-12.tp2.b3:data-chunk1 .................................................................................... [TPGs: 1]
  | | o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  | |   o- acls .......................................................................................................... [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator ...................................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk1 (rw)]
  | |   o- luns .......................................................................................................... [LUNs: 1]
  | |   | o- lun0 ............................................................... [fileio/data-chunk1 (/dev/md0) (default_tg_pt_gp)]
  | |   o- portals .................................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ..................................................................................................... [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk2 .................................................................................... [TPGs: 1]
  | | o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  | |   o- acls .......................................................................................................... [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk1:chunk2-initiator ...................................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk2 (rw)]
  | |   o- luns .......................................................................................................... [LUNs: 1]
  | |   | o- lun0 ............................................................... [fileio/data-chunk2 (/dev/md1) (default_tg_pt_gp)]
  | |   o- portals .................................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ..................................................................................................... [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk3 .................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 1]
  |     | o- iqn.2024-12.tp2.b3:data-chunk1:chunk3-initiator ...................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk3 (rw)]
  |     o- luns .......................................................................................................... [LUNs: 1]
  |     | o- lun0 ............................................................... [fileio/data-chunk3 (/dev/md2) (default_tg_pt_gp)]
  |     o- portals .................................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ..................................................................................................... [OK]
  o- loopback ......................................................................................................... [Targets: 0]
/>
```

## 2. Chunks machine

## A. Simple iSCSI 

### 🌞 Installer les tools iSCSI sur chunk1.tp2.b3 : 
SUR CHUNK1

```Bash
dnf install iscsi-initiator-utils
```

### 🌞 Configurer un iSCSI initiator : 

```Bash
#Sur la machine target 
firewall-cmd --add-port=3260/tcp --permanent
systemctl reload firewalld

#Sur Chunk1 
sudo iscsiadm -m discoverydb -t st -p 10.3.1.1:3260 --discover
sudo iscsiadm -m discoverydb -t st -p 10.3.2.1:3260 --discover
sudo iscsiadm -m discoverydb -t st -p 10.3.1.2:3260 --discover
sudo iscsiadm -m discoverydb -t st -p 10.3.2.2:3260 --discover

#Modifier initiator name
nano /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator

#Enable au démarrage
systemctl enable iscsi
systemctl enable iscsid

#Verifier présence des liens 
ls /var/lib/iscsi/send_targets/

#SUR STO MODIF AUTHENTIFICATION A 0 : 
cd /iscsi/iqn.2024-12.tp2.b3:data-chunk1/tpg1/ 
set attribute authentication=0

#Node : 
iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -l
```

#### - Résultat : 
```Bash
[root@chunk1 rocky]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0   20G  0 disk
sdc           8:32   0   20G  0 disk
sdd           8:48   0   20G  0 disk
sde           8:64   0   20G  0 disk
sr0          11:0    1 1024M  0 rom
```

### 🌞 Modifier la configuration du démon iSCSI : 

```Bash
nano /etc/iscsi/iscsid.conf
node.session.timeo.replacement_timeout = 0
systemctl restart iscsi
systemctl restart iscsid
```

### 🌞 Prouvez que la configuration est prête : 

```Bash
#LSBLK :
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0   20G  0 disk
sdc           8:32   0   20G  0 disk
sdd           8:48   0   20G  0 disk
sde           8:64   0   20G  0 disk
sr0          11:0    1 1024M  0 rom

#iscsiadm -m session -P 3
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk1 (non-flash)
        Current Portal: 10.3.2.2:3260,1
        Persistent Portal: 10.3.2.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.2.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 61
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 3  State: running
                scsi3 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdc          State: running
        Current Portal: 10.3.1.1:3260,1
        Persistent Portal: 10.3.1.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.1.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 62
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 4  State: running
                scsi4 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdb          State: running
        Current Portal: 10.3.1.2:3260,1
        Persistent Portal: 10.3.1.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.1.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 63
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 5  State: running
                scsi5 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdd          State: running
        Current Portal: 10.3.2.1:3260,1
        Persistent Portal: 10.3.2.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.2.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 64
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 120
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 6  State: running
                scsi6 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sde          State: running
```


## B. Multipathing : 

### 🌞 Installer les outils multipath sur chunk1.tp2.b3

```Bash
dnf install device-mapper-multipath
```

### 🌞 Configurer le fichier /etc/multipath.conf

```Bash
nano /etc/multipath.conf

defaults {
  user_friendly_names yes
  find_multipaths yes
  path_grouping_policy failover
  features "1 queue_if_no_path"
  no_path_retry 100
}
```

### 🌞 Démarrer le service multipathd

```Bash
systemctl start multipathd
systemctl status multipathd
```

### 🌞 Et euh c'est tout, il est smart enough

```Bash
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0   20G  0 disk
└─mpathb    253:3    0   20G  0 mpath
sdc           8:32   0   20G  0 disk
└─mpatha    253:2    0   20G  0 mpath
sdd           8:48   0   20G  0 disk
└─mpatha    253:2    0   20G  0 mpath
sde           8:64   0   20G  0 disk
└─mpathb    253:3    0   20G  0 mpath
sr0          11:0    1 1024M  0 rom


mpatha (360014050c61babf015c46029afb5e6a5) dm-2 LIO-ORG,data-chunk1
size=20G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 3:0:0:0 sdc 8:32 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 5:0:0:0 sdd 8:48 active ready running
mpathb (3600140582676305f0534a8f83eb10388) dm-3 LIO-ORG,data-chunk1
size=20G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 4:0:0:0 sdb 8:16 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
  ```

## 3. Formatage et montage :

### 🌞 Créez une partition sur les devices mpatha et mpathb :
```Bash
fdisk /dev/mapper/mpatha
fdisk /dev/mapper/mpathb
```

### 🌞 Formatez en xfs les partitions :
```Bash
mkfs.ext4 /dev/mapper/mpatha
mkfs.ext4 /dev/mapper/mpathb
```

### 🌞 Point de montage /mnt/data_chunk1 :
```Bash
mkdir /mnt/data_chunk1
mount /dev/mapper/mpatha /mnt/data_chunk1/

#Resultat:
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0   20G  0 disk
└─mpathb    253:3    0   20G  0 mpath /mnt/data_chunk2
sdc           8:32   0   20G  0 disk
└─mpatha    253:2    0   20G  0 mpath /mnt/data_chunk1
sdd           8:48   0   20G  0 disk
└─mpatha    253:2    0   20G  0 mpath /mnt/data_chunk1
sde           8:64   0   20G  0 disk
└─mpathb    253:3    0   20G  0 mpath /mnt/data_chunk2
sr0          11:0    1 1024M  0 rom

#Automount : 
nano /etc/systemd/system/data_chunk1.mount
[Unit]
Description=mpunt chunk1
After=local-fs.target iscsid.service

[Mount]
What=/dev/mapper/mpatha
Where=/mnt/data_chunk1
Type=ext4

[Install]
WantedBy=multi-user.target
After=network-online.target
```

### 🌞 Point de montage /mnt/data_chunk2 :

```Bash
mkdir /mnt/data_chunk1
mount /dev/mapper/mpatha /mnt/data_chunk1/

#Mount : 
nano /etc/systemd/system/data_chunk2.mount
[Unit]
Description=chunk2
After=local-fs.target iscsid.service

[Mount]
What=/dev/mapper/mpathb
Where=/mnt/data_chunk2
Type=ext4

[Install]
WantedBy=multi-user.target
```

## 4. Tests :

## A. Simulation de panne :

### 🌞 Simuler une coupure réseau :

```bash
ifdown enp0s9

watch -n 0.5 multipath -ll

#Résultat :

mpatha (360014050c61babf015c46029afb5e6a5) dm-3 LIO-ORG,data-chunk1
size=20G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 10:0:0:0 sde 8:64 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 9:0:0:0  sdd 8:48 active ready running
mpathb (3600140582676305f0534a8f83eb10388) dm-2 LIO-ORG,data-chunk1
size=20G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 7:0:0:0  sdb 8:16 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 8:0:0:0  sdc 8:32 active ready running
```

## B. Jouer avec les paramètres : 

### 🌞 Resimuler une panne :

```Bash
#Conf a modifier pour modifier la réaction : 

defaults {
  user_friendly_names yes
  find_multipaths yes
  path_grouping_policy failover
  features "1 queue_if_no_path"
  no_path_retry 100
}
```

## 5. Replicate :

### 🌞 Preuve du setup :

```Bash
#Chunk 1 : 
[root@chunk1 rocky]# ls /var/lib/iscsi/send_targets/
10.3.1.1,3260  10.3.1.2,3260  10.3.2.1,3260  10.3.2.2,3260

#Chunk 2 :
[root@chunk2 rocky]# ls /var/lib/iscsi/send_targets/
10.3.1.1,3260  10.3.1.2,3260  10.3.2.1,3260  10.3.2.2,3260

#Chunk 3 : 
[root@chunk3 rocky]# ls /var/lib/iscsi/send_targets/
10.3.1.1,3260  10.3.1.2,3260  10.3.2.1,3260  10.3.2.2,3260
```

# 2. Distribued filesystem


## 1. Master server

### 🌞 Installer les paquets nécessaires pour avoir un Moose Master

```Bash
curl "https://repository.moosefs.com/RPM-GPG-KEY-MooseFS" > /etc/pki/rpm-gpg/RPM-GPG-KEY-MooseFS
curl "http://repository.moosefs.com/MooseFS-3-el9.repo" > /etc/yum.repos.d/MooseFS.repo
yum install moosefs-master moosefs-cgi moosefs-cgiserv moosefs-cli

#Note pour la suite : 

#Master 
yum install moosefs-master moosefs-cgi moosefs-cgiserv moosefs-cli
#Chunk 
yum install moosefs-chunkserver
#Metalogger
yum install moosefs-metalogger
#Clients
yum install moosefs-client
```

### 🌞 Démarrez les services du Moose Master

```Bash
systemctl start moosefs-master
systemctl start moosefs-cgi
```

### 🌞 Ouvrez les ports firewall

```Bash
ufw allow 80/tcp
ufw allow 9425/tcp
ufw allow 9419/tcp
ufw allow 9420/tcp
ufw allow 9421/tcp
```

## 2. Chunk servers (Toutes les commandes de ces parties sont répétés sur les 3 chunk)

### 🌞 Installer les paquets nécessaires pour avoir un Moose Chunk Server

```Bash
#Répété sur les 3 serveurs.
curl "https://repository.moosefs.com/RPM-GPG-KEY-MooseFS" > /etc/pki/rpm-gpg/RPM-GPG-KEY-MooseFS
curl "http://repository.moosefs.com/MooseFS-3-el9.repo" > /etc/yum.repos.d/MooseFS.repo
yum install moosefs-chunkserver
```

### 🌞 Modifier la conf du Chunk Server

```Bash
nano /etc/mfs/mfschunkserver.cfg
MooseFS master host, IP is allowed only in single-master installations (default is mfsmaster)
MASTER_HOST = master

#Le fichier hosts :
10.3.250.11 chunk1.tp2.b3 chunk1
10.3.1.101  chunk1.tp2.b3
10.3.2.101  chunk1.tp2.b3
10.3.250.12 chunk2.tp2.b3 chunk2
10.3.1.102  chunk2.tp2.b3
10.3.2.102  chunk2.tp2.b3
10.3.250.13 chunk3.tp2.b3 chunk3
10.3.1.103  chunk3.tp2.b3
10.3.2.103  chunk3.tp2.b3
10.3.250.1  master.tp2.b3 master
10.3.250.101 web.tp2.b3 web
10.3.1.1    sto1.tp2.b3 sto1
10.3.2.1    sto1.tp2.b3
10.3.1.2    sto2.tp2.b3 sto2
10.3.2.2    sto2.tp2.b3
```

### 🌞 Faire appartenir les partitions à partager à l'utilisateur 
```Bash
chown mfs:mfs /mnt/data_chunk1
chown mfs:mfs /mnt/data_chunk2
```

### 🌞 Modifier la conf des disques du Chunk Server

```Bash
nano /etc/mfs/mfshdd.cfg

use hard drive '/mnt/data_chunk1' with default options:
/mnt/data_chunk1

use hard drive '/mnt/data_chunk2' with default options:
/mnt/data_chunk2
```

### 🌞 Démarrez les services du Moose Chunk Server

```Bash
sudo systemctl enable moosefs-chunkserver
sudo systemctl start moosefs-chunkserver
```

# 3. Consume

## 1. Monter la partition Moose

### 🌞 Installer les paquets nécessaires pour avoir un Moose Client

```Bash
curl "https://repository.moosefs.com/RPM-GPG-KEY-MooseFS" > /etc/pki/rpm-gpg/RPM-GPG-KEY-MooseFS
curl "http://repository.moosefs.com/MooseFS-3-el9.repo" > /etc/yum.repos.d/MooseFS.repo
yum install moosefs-client
```

### 🌞 Monter la partition Moose

```Bash
sudo mkdir /mnt/www
sudo mfsmount /mnt/www -H master
```

### 🌞 Preuve et test d'écriture

```Bash
#Permission RW
mfsmount on /mnt/www type fuse.mfsmount (rw,allow_other)
```

## 2. NGINX

### 🌞 Installer et configurer NGINX sur la machine web.tp2.b3

```Bash
dnf install nginx
```

### 🌞 Démarrer le service NGINX

```Bash
[root@web ~]# systemctl start nginx
[root@web ~]# systemctl enable nginx

[root@web ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
     Active: active (running) since Sat 2024-12-14 17:43:56 CET; 38s ago
   Main PID: 11614 (nginx)
      Tasks: 2 (limit: 11097)
     Memory: 2.0M
        CPU: 82ms
     CGroup: /system.slice/nginx.service
             ├─11614 "nginx: master process /usr/sbin/nginx"
             └─11615 "nginx: worker process"

Dec 14 17:43:56 web systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 14 17:43:56 web nginx[11612]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 14 17:43:56 web nginx[11612]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 14 17:43:56 web systemd[1]: Started The nginx HTTP and reverse proxy server.
```

### 🌞 Prouvez avec un curl que le site est actif

```Bash
[root@web ~]# curl web
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1);
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;

      }


  h1 {
    text-align: center;
    margin: 0;
    padding: 0.6em 2em 0.4em;
    color: #fff;
    font-weight: bold;
    font-family: 'Montserrat', sans-serif;
    font-size: 2em;
  }
  h1 strong {
    font-weight: bolder;
    font-family: 'Montserrat', sans-serif;
  }
  h2 {
    font-size: 1.5em;
    font-weight:bold;
  }

  .title {
    border: 1px solid black;
    font-weight: bold;
    position: relative;
    float: right;
    width: 150px;
    text-align: center;
    padding: 10px 0 10px 0;
    margin-top: 0;
  }

  .description {
    padding: 45px 10px 5px 10px;
    clear: right;
    padding: 15px;
  }

  .section {
    padding-left: 3%;
   margin-bottom: 10px;
  }

  img {

    padding: 2px;
    margin: 2px;
  }
  a:hover img {
    padding: 2px;
    margin: 2px;
  }

  :link {
    color: rgb(199, 252, 77);
    text-shadow:
  }
  :visited {
    color: rgb(122, 206, 255);
  }
  a:hover {
    color: rgb(16, 44, 122);
  }
  .row {
    width: 100%;
    padding: 0 10px 0 10px;
  }

  footer {
    padding-top: 6em;
    margin-bottom: 6em;
    text-align: center;
    font-size: xx-small;
    overflow:hidden;
    clear: both;
  }

  .summary {
    font-size: 140%;
    text-align: center;
  }

  #rocky-poweredby img {
    margin-left: -10px;
  }

  #logos img {
    vertical-align: top;
  }

  /* Desktop  View Options */

  @media (min-width: 768px)  {

    body {
      padding: 10em 20% !important;
    }

    .col-md-1, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6,
    .col-md-7, .col-md-8, .col-md-9, .col-md-10, .col-md-11, .col-md-12 {
      float: left;
    }

    .col-md-1 {
      width: 8.33%;
    }
    .col-md-2 {
      width: 16.66%;
    }
    .col-md-3 {
      width: 25%;
    }
    .col-md-4 {
      width: 33%;
    }
    .col-md-5 {
      width: 41.66%;
    }
    .col-md-6 {
      border-left:3px ;
      width: 50%;


    }
    .col-md-7 {
      width: 58.33%;
    }
    .col-md-8 {
      width: 66.66%;
    }
    .col-md-9 {
      width: 74.99%;
    }
    .col-md-10 {
      width: 83.33%;
    }
    .col-md-11 {
      width: 91.66%;
    }
    .col-md-12 {
      width: 100%;
    }
  }

  /* Mobile View Options */
  @media (max-width: 767px) {
    .col-sm-1, .col-sm-2, .col-sm-3, .col-sm-4, .col-sm-5, .col-sm-6,
    .col-sm-7, .col-sm-8, .col-sm-9, .col-sm-10, .col-sm-11, .col-sm-12 {
      float: left;
    }

    .col-sm-1 {
      width: 8.33%;
    }
    .col-sm-2 {
      width: 16.66%;
    }
    .col-sm-3 {
      width: 25%;
    }
    .col-sm-4 {
      width: 33%;
    }
    .col-sm-5 {
      width: 41.66%;
    }
    .col-sm-6 {
      width: 50%;
    }
    .col-sm-7 {
      width: 58.33%;
    }
    .col-sm-8 {
      width: 66.66%;
    }
    .col-sm-9 {
      width: 74.99%;
    }
    .col-sm-10 {
      width: 83.33%;
    }
    .col-sm-11 {
      width: 91.66%;
    }
    .col-sm-12 {
      width: 100%;
    }
    h1 {
      padding: 0 !important;
    }
  }


  </style>
  </head>
  <body>
    <h1>HTTP Server <strong>Test Page</strong></h1>

    <div class='row'>

      <div class='col-sm-12 col-md-6 col-md-6 '></div>
          <p class="summary">This page is used to test the proper operation of
            an HTTP server after it has been installed on a Rocky Linux system.
            If you can read this page, it means that the software is working
            correctly.</p>
      </div>

      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>


        <div class='section'>
          <h2>Just visiting?</h2>

          <p>This website you are visiting is either experiencing problems or
          could be going through maintenance.</p>

          <p>If you would like the let the administrators of this website know
          that you've seen this page instead of the page you've expected, you
          should send them an email. In general, mail sent to the name
          "webmaster" and directed to the website's domain should reach the
          appropriate person.</p>

          <p>The most common email address to send to is:
          <strong>"webmaster@example.com"</strong></p>

          <h2>Note:</h2>
          <p>The Rocky Linux distribution is a stable and reproduceable platform
          based on the sources of Red Hat Enterprise Linux (RHEL). With this in
          mind, please understand that:

        <ul>
          <li>Neither the <strong>Rocky Linux Project</strong> nor the
          <strong>Rocky Enterprise Software Foundation</strong> have anything to
          do with this website or its content.</li>
          <li>The Rocky Linux Project nor the <strong>RESF</strong> have
          "hacked" this webserver: This test page is included with the
          distribution.</li>
        </ul>
        <p>For more information about Rocky Linux, please visit the
          <a href="https://rockylinux.org/"><strong>Rocky Linux
          website</strong></a>.
        </p>
        </div>
      </div>
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
        <div class='section'>

          <h2>I am the admin, what do I do?</h2>

        <p>You may now add content to the webroot directory for your
        software.</p>

        <p><strong>For systems using the
        <a href="https://httpd.apache.org/">Apache Webserver</strong></a>:
        You can add content to the directory <code>/var/www/html/</code>.
        Until you do so, people visiting your website will see this page. If
        you would like this page to not be shown, follow the instructions in:
        <code>/etc/httpd/conf.d/welcome.conf</code>.</p>

        <p><strong>For systems using
        <a href="https://nginx.org">Nginx</strong></a>:
        You can add your content in a location of your
        choice and edit the <code>root</code> configuration directive
        in <code>/etc/nginx/nginx.conf</code>.</p>

        <div id="logos">
          <a href="https://rockylinux.org/" id="rocky-poweredby"><img src="icons/poweredby.png" alt="[ Powered by Rocky Linux ]" /></a> <!-- Rocky -->
          <img src="poweredby.png" /> <!-- webserver -->
        </div>
      </div>
      </div>

      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>

  </body>
</html>
```