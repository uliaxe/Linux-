# **TP4 : Real services**

## **Partie 1 : Partitionnement du serveur de stockage**

🌞 **Partitionner le disque à l'aide de LVM**

- créer un physical volume (PV) :

```bash
[root@storage ~]# sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```

verification :

```bash
[root@storage ~]# sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sdb
  VG Name               
  PV Size              2.03GiB
  Allocatable           NO
  PE Size               4.00 MiB
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               6TktfJ-HwH2-4eM9-uSVm-us5C-9kAQ-PVe9L6
```

- créer un volume group (VG) :

```bash
[root@storage ~]# sudo vgcreate storage /dev/sdb
  Volume group "vg1" successfully created
```

verification :

```bash
[root@storage ~]# sudo vgdisplay
  --- Volume group ---
  VG Name               storage
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               2.03 GiB
  PE Size               4.00 MiB
  Total PE              520
  Alloc PE / Size       0 / 0   
  Free  PE / Size       520 / 2.03 GiB
  VG UUID               L9tELa-gLes-1Rgk-5yFN-HJp2-iDMU-vEkimg
```

- créer un logical volume (LV) :

```bash
[root@storage ~]# sudo lvcreate -L 100%FREE -n storage data
  Logical volume "data" created.
```

verification :

```bash
[root@storage ~]# sudo lvs
    LV   VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    data storage -wi-a----- 2.03GiB

 ```

## 🌞 **Formater la partition**

- vous formaterez la partition en ext4 (avec une commande mkfs)

```bash
[root@storage ~]# sudo mkfs.ext4 /dev/storage/data
```

## 🌞 **Monter la partition**

- montage de la partition (avec la commande mount)
  
  ```bash
  [root@storage ~]# sudo mount /dev/storage/data /mnt/data1
  ```

verification :

```bash
[root@storage ~]# sudo df -h | grep data
/dev/mapper/storage-data  2.0G  24K  1.9G   1% /mnt/data1
```

- prouvez que vous pouvez lire et écrire des données sur cette partition
  
  ```bash
  [root@storage ~]# sudo touch /mnt/data1/test
  [root@storage ~]# sudo ls /mnt/data1
  lost+found test
  ```

- définir un montage automatique de la partition (fichier /etc/fstab)

```bash
[root@storage ~]# sudo vim /etc/fstab
```

ajouter la ligne suivante :

```bash
/dev/mapper/storage-data /mnt/data1 ext4 defaults 0 0
```

- vous vérifierez que votre fichier /etc/fstab fonctionne correctement

```bash
[root@storage ~]# sudo umount /mnt/data1
[root@storage ~]# sudo mount -av
```

## **Partie 2 : Serveur de partage de fichiers**

🌞 Donnez les commandes réalisées sur le serveur NFS storage.tp4.linux

- installer le serveur NFS

```bash
[root@storage ~]# sudo dnf install nfs-utils
```

- créer un répertoire partagé

```bash
[root@storage ~]# sudo mkdir /mnt/data1/share
```

- donner les droits d'accès au répertoire partagé

```bash
[root@storage ~]# sudo chmod 777 /mnt/data1/share
```

- ajouter une entrée dans le fichier /etc/exports

```bash
[root@storage ~]# sudo vim /etc/exports
```
