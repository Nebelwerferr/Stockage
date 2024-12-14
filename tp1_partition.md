# Compte Rendu TP Partition

## I. Fundamentals :


### 🌞 Lister tous les périphériques de stockage branchés à la VM && Lister toutes les partitions des périphériques de stockage :

```Bash
lsblk
```

### 🌞 Effectuer un test SMART sur le disque : 

```Bash
apt-get install sarmentons
sudo smartctl -i /dev/sda
```

### 🌞 Espace disque... : 

```Bash
df -k
```

### 🌞 Inodes : 

```Bash
df -i
ioping -c 10 .
```
### 🌞 Déterminer la taille du cache filesystem : 

## II. Partitioning : 

### 🌞 Ajouter sdb comme Physical Volume LVM :

- Pour formater le disque : 
```Bash
sudo fdisk /dev/sdb

Bienvenue dans fdisk (util-linux 2.38.1).
Les modifications resteront en mémoire jusqu'à écriture.
Soyez prudent avant d'utiliser la commande d'écriture.

Le périphérique ne contient pas de table de partitions reconnue.
Created a new DOS (MBR) disklabel with disk identifier 0xd3ec0ada.

Commande (m pour l'aide) : n
Type de partition
   p   primaire (0 primaire, 0 étendue, 4 libre)
   e   étendue (conteneur pour partitions logiques)
Sélectionnez (p par défaut) : p
Numéro de partition (1-4, 1 par défaut) :
Premier secteur (2048-20971519, 2048 par défaut) :
Dernier secteur, +/-secteurs ou +/-taille{K,M,G,T,P} (2048-20971519, 20971519 par défaut) :

Une nouvelle partition 1 de type « Linux » et de taille 10 GiB a été créée.

Commande (m pour l'aide) : p
Disque /dev/sdb : 10 GiB, 10737418240 octets, 20971520 secteurs
Modèle de disque : VBOX HARDDISK
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets
Type d'étiquette de disque : dos
Identifiant de disque : 0xd3ec0ada

Périphérique Amorçage Début      Fin Secteurs Taille Id Type
/dev/sdb1              2048 20971519 20969472    10G 83 Linux

Commande (m pour l'aide) : t
Partition 1 sélectionnée
Code Hexa ou synonyme (taper L pour afficher tous les codes) :8e
Type de partition « Linux » modifié en « Linux LVM ».

Commande (m pour l'aide) : p
Disque /dev/sdb : 10 GiB, 10737418240 octets, 20971520 secteurs
Modèle de disque : VBOX HARDDISK
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets
Type d'étiquette de disque : dos
Identifiant de disque : 0xd3ec0ada

Périphérique Amorçage Début      Fin Secteurs Taille Id Type
/dev/sdb1              2048 20971519 20969472    10G 8e LVM Linux

Commande (m pour l'aide) : w
La table de partitions a été altérée.
Appel d'ioctl() pour relire la table de partitions.
Synchronisation des disques.
```

- Pour créer le volume physique : 
```Bash
sudo pvcreate /dev/sdb
```

- Pour visualiser les volumes physiques : 
```Bash
sudo pvdisplay
```

### 🌞 Créer un Volume Group LVM nommé storage :

- Créer le volume : 
```Bash
root@storage:~# sudo vgcreate storage /dev/sdb1
 Volume group "storage" successfully created
```

- Visualiser les volumes existants :
```Bash
sudo vgs
```

### 🌞 Créer un Logical Volume LVM : 

```Bash
root@storage:~# sudo lvcreate -L 2G -n smol_data storage
  Logical volume "smol_data" created.
```

### 🌞 Créer un deuxième Logical Volume LVM

```Bash
root@storage:~# sudo lvcreate --name big_data -l 100%FREE storage
  Logical volume "big_data" created.
```
### 🌞 Créez un système de fichiers sur les deux LVs

```Bash
root@storage:~# sudo mkfs.ext4 /dev/storage/smol_data
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: fc7c2b7a-4216-455e-8bb2-b2686d1d3fd3
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

root@storage:~# sudo mkfs.ext4 /dev/storage/big_data
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2096128 4k blocks and 524288 inodes
Filesystem UUID: 300987a7-a622-44f6-ab42-531bb26b9886
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

### 🌞 Montez la partition smol_data : 

```Bash
mkdir /smol
sudo mount /dev/storage/smol_data /smol/
```

### 🌞 Montez la partition big_data : 

```Bash
mkdir /big
sudo mount /dev/storage/big_data /big/
```

### 🌞 Configurer un automount A FINIR : 

Désolé, je n'ai pas réussi après avoir cherché pas mal de temps :/
Je suis en interface graphique Debian 12 (Bien que je fais le tp via putty).
Pour tout ce qui touche à l'automount j'ai cherché de chercher a passer par une autre commande, mais je n'ai pas trouvé.

### 🌞 Prouvez que l'automount est effectif A FINIR :

""

## III. RAID

### 🌞 Mettre en place un RAID 5

```Bash
root@storage:~# sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdc /dev/sdd /dev/sde
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 10476544K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

root@storage:~# sudo mkfs.ext4 /dev/md0
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 5238272 4k blocks and 1310720 inodes
Filesystem UUID: d2f42f2f-5baf-4308-acb3-9388ee0dd4f8
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

### 🌞 Prouvez que le RAID5 est en place, on doit voir :

```Bash
mdadm -D /dev/md0
```

### 🌞 Rendre la configuration automatique au boot de la machine A FINIR :

""

### 🌞 Créez un système de fichiers sur la partition proposé par le RAID :

```Bash
sudo mkfs.ext4 /dev/md0
```

### 🌞 Monter la partition sur /mnt/raid_storage :

```Bash
root@storage:~# mkdir /mnt/raid5
root@storage:~# sudo mount /dev/md0 /mnt/raid5
```

### 🌞 Prouvez que… la partition est bien montée il y a bien l'espace disponible attendue sur la partition vous pouvez lire et écrire sur la partition

```Bash
20GO Disponible
sdc                     8:32   0   10G  0 disk
└─md0                   9:0    0   20G  0 raid5 /mnt/raid5
sdd                     8:48   0   10G  0 disk
└─md0                   9:0    0   20G  0 raid5 /mnt/raid5
sde                     8:64   0   10G  0 disk
└─md0                   9:0    0   20G  0 raid5 /mnt/raid5
```

### 🌞 Mini benchmark : 

```Bash
Teste de vitesse partition : sudo dd if=/dev/zero of=/mnt/raid_storage bs=1G count=1 oflag=direct

1+0 enregistrements lus
1+0 enregistrements écrits
1073741824 octets (1,1 GB, 1,0 GiB) copiés, 2,13335 s, 503 MB/s

Teste de vitesse partition LVM : sudo dd if=/dev/zero of=/mnt/lvm_storage bs=1G count=1 oflag=direct

1+0 enregistrements lus
1+0 enregistrements écrits
1073741824 octets (1,1 GB, 1,0 GiB) copiés, 0,803255 s, 1,3 GB/s
```

### 🌞 Simule une panne

- Supprimer un disque (sde) : 
```Bash
	- METTRE EN ERREUR : sudo mdadm --manage /dev/md127 --fail /dev/sde
	- UNMOUNT LE DISQUE DU RAID : sudo mdadm --manage /dev/md127 --remove /dev/sde
```

### 🌞 Montre l'état du RAID dégradé : 

- Verifier que le raid est en failure : 
```Bash
cat /proc/mdstat
```

- Résultat : 
```Bash
Personalities : [raid6] [raid5] [raid4] [linear] [multipath] [raid0] [raid1] [raid10]
md127 : active (auto-read-only) raid5 sdc[0] sdd[1]
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
```

Il y a bien seulement 2 disques sur 3.

### 🌞 Remonte le disque dur : 

- Ajouter à nouveau le disque sde : 
```Bash
sudo mdadm --manage /dev/md127 --add /dev/sde
```

- Recovery en cours : 
```Bash
Personalities : [raid6] [raid5] [raid4] [linear] [multipath] [raid0] [raid1] [raid10]
md127 : active raid5 sde[3] sdc[0] sdd[1]
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
      [>....................]  recovery =  3.7% (397276/10476544) finish=0.8min speed=198638K/sec

unused devices: <none>
```

- Prouver que le raid est fonctionnel : 
```Bash
Personalities : [raid6] [raid5] [raid4] [linear] [multipath] [raid0] [raid1] [raid10]
md127 : active raid5 sde[3] sdc[0] sdd[1]
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
```

Il y a bien 3/3 disque et pas de F pour signaler une failure sur un disque en particulier

### 🌞 Ajoutez un disque encore inutilisé au RAID5 comme disque de spare : 

- Ajout de sdf comme disque spare :
```Bash
sudo mdadm --manage /dev/md127 --add-spare /dev/sdf
```

- Vérifier qu'il y a bien un disque spare :
```Bash
sudo mdadm --detail /dev/md127
```

- Résultat : 
```Bash
/dev/md127:
           Version : 1.2
     Creation Time : Thu Nov 14 16:14:43 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Mon Dec  2 21:57:45 2024
             State : clean
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 5342b153:efdc3cc3:8899a5b2:194f3307
            Events : 41

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde

       4       8       80        -      spare   /dev/sdf
```

### 🌞 Simuler une panne : 

- Mettre sde en failure :

```Bash
sudo mdadm --manage /dev/md127 --fail /dev/sde
```

Recovery directement relancé avec le disque sparee, entièrement remis en 3/3 au bout de 22.4 secondes.

### 🌞 Remonter le disque débranché :

- Rajouter a nouveau sde qui prend le rôle de spare :
```Bash
sudo mdadm --manage /dev/md127 --remove /dev/sde
```

### 🌞 Ajoutez un disque encore inutilisé au RAID5 comme disque de spare

- Ajout de sdg comme disque spare :
```Bash
sudo mdadm --manage /dev/md127 --add-spare /dev/sdg
```


- Vérifier :
```Bash
sudo mdadm --detail /dev/md127
```

- Résultat : 
```Bash
/dev/md127:
           Version : 1.2
     Creation Time : Thu Nov 14 16:14:43 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Mon Dec  2 22:06:45 2024
             State : clean
    Active Devices : 3
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 2

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 5342b153:efdc3cc3:8899a5b2:194f3307
            Events : 63

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       4       8       80        2      active sync   /dev/sdf

       3       8       64        -      spare   /dev/sde
       5       8       96        -      spare   /dev/sdg
```

### 🌞 Grow !

- Redimenssioner la taille du raid5 :
```Bash
mdadm --grow /dev/md127 --raid-devices=4
```

### 🌞 Prouvez que le RAID5 propose désormais 4 disques actifs : 

- Vérifier :
```Bash
sudo mdadm --detail /dev/md127
```

- Résultat : 
```Bash
/dev/md127:
           Version : 1.2
     Creation Time : Thu Nov 14 16:14:43 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 4
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Mon Dec  2 22:13:54 2024
             State : clean, reshaping
    Active Devices : 4
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

    Reshape Status : 29% complete
     Delta Devices : 1, (3->4)

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 5342b153:efdc3cc3:8899a5b2:194f3307
            Events : 96

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       4       8       80        2      active sync   /dev/sdf
       5       8       96        3      active sync   /dev/sdg

       6       8       64        -      spare   /dev/sde
```

Il y a bien 4 disques actifs : sdc sdd sdf sdg & sde en spare

### 🌞 Euuuh wait a sec... /mnt/raid_storage ???

VOIR ANCIENNE CONF VERIFIER MAIS MEME STOCKAGE 

- Commande pour actualiser le stockage :
```Bash
sudo resize2fs /dev/md127
```

- Vérifier : 
```Bash
/dev/md127:
           Version : 1.2
     Creation Time : Thu Nov 14 16:14:43 2024
        Raid Level : raid5
        Array Size : 31429632 (29.97 GiB 32.18 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 4
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Mon Dec  2 22:16:03 2024
             State : clean
    Active Devices : 4
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 5342b153:efdc3cc3:8899a5b2:194f3307
            Events : 111

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       4       8       80        2      active sync   /dev/sdf
       5       8       96        3      active sync   /dev/sdg

       6       8       64        -      spare   /dev/sde
```

### 🌞 Installer un serveur NFS : 

- Installer les paquets : 
```Bash
sudo apt install nfs-kernel-server
```

- Ajouter aux exports NFS, modifier le fichier de conf /etc/exports, ici donné permission RW et X :
```Bash
/mnt/raid_storage 10.2.70.53/24(rw,sync,no_subtree_check)
/mnt/lvm_storage 10.2.70.53/24(rw,sync,no_subtree_check)
```

- Monter les partages : 

```Bash
sudo mount 192.168.248.10:/mnt/raid_storage /mnt/raid_storage
sudo mount 192.168.248.10:/mnt/raid_storage /mnt/lvm_storage
```

- Vérifier : 

```Bash
df -h
ls /mnt/raid_storage
ls /mnt/lvm_storage
```

- Benchmark / test de vitesse :

```Bash
dd if=/dev/md127 of=/mnt/raid_storage/testfile bs=1G count=1 oflag=direct
dd if=/dev/md127 of=/mnt/lvm_storage/testfile bs=1G count=1 oflag=direct
```