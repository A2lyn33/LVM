# Introduction à LVM : Concepts, Terminologie et Opérations

**Mis à jour le 22 décembre 2022**  
*Catégories :* Bases de Linux, Outils système, Stockage, Concepts  

## Introduction

LVM, ou *Logical Volume Management*, est une technologie de gestion des périphériques de stockage qui permet de regrouper et d'abstraire la disposition physique des périphériques de stockage pour une administration flexible. En s'appuyant sur le framework *device mapper* du noyau Linux, la version actuelle, LVM2, permet de regrouper des périphériques de stockage existants en groupes et d'allouer des unités logiques à partir de l'espace combiné selon les besoins.

Les principaux avantages de LVM incluent une abstraction accrue, une flexibilité et un contrôle amélioré. Les volumes logiques peuvent avoir des noms significatifs comme "bases-de-données" ou "sauvegarde-racine". Les volumes peuvent également être redimensionnés dynamiquement en fonction des besoins en espace, migrés entre des périphériques physiques dans le groupe de volumes ou exportés. LVM propose également des fonctionnalités avancées comme les instantanés (*snapshotting*), le striping et la mise en miroir.

Dans ce guide, vous apprendrez comment fonctionne LVM et découvrirez les commandes de base pour configurer un système LVM sur une machine physique.

---

## Architecture et Terminologie LVM

Avant de plonger dans les commandes administratives, il est important de comprendre comment LVM organise les périphériques de stockage et les termes qu’il utilise.

### Structures de Gestion du Stockage LVM

LVM fonctionne en superposant des abstractions au-dessus des périphériques de stockage physiques. Les couches de base utilisées par LVM, de la plus primitive à la plus abstraite, sont :

- **Volumes physiques (Physical Volumes, PV)**  
  Les volumes physiques, préfixés par `pv...`, représentent des blocs de périphériques physiques ou d'autres dispositifs similaires (comme des tableaux RAID créés par *device mapper*). LVM écrit un en-tête sur le périphérique pour le préparer à la gestion.

- **Groupes de volumes (Volume Groups, VG)**  
  Les groupes de volumes, préfixés par `vg...`, regroupent les volumes physiques en une unité logique fonctionnant comme un périphérique unique avec la capacité combinée des volumes physiques.

- **Volumes logiques (Logical Volumes, LV)**  
  Les volumes logiques, préfixés par `lv...`, sont créés à partir des groupes de volumes. Ces volumes sont l'équivalent fonctionnel des partitions sur un disque physique, mais offrent beaucoup plus de flexibilité.

### Comprendre les Extents

Chaque volume d’un groupe de volumes est segmenté en petites unités de taille fixe appelées *extents*. La taille des extents est définie par le groupe de volumes, et tous les volumes au sein d’un même groupe respectent cette taille.

Les extents physiques sont ceux des volumes physiques, tandis que les extents logiques sont ceux des volumes logiques. Cette structure permet une flexibilité accrue, comme l'agrandissement ou la réduction des volumes logiques sans interruption.

---

## Cas d’Utilisation Courants

Vous allez maintenant explorer un scénario où deux disques physiques sont utilisés pour former quatre volumes logiques.

### Marquer les Dispositifs Physiques comme Volumes Physiques

Commencez par scanner le système pour détecter les périphériques accessibles par LVM :

```bash
sudo lvmdiskscan
```

Cela renverra une liste des périphériques disponibles. Par exemple :

```plaintext
/dev/sda  [ 200.00 GiB ]
/dev/sdb  [ 100.00 GiB ]
```

**Attention :** Assurez-vous que les périphériques à utiliser avec LVM ne contiennent pas de données importantes.

Pour marquer les disques comme volumes physiques, utilisez la commande suivante :

```bash
sudo pvcreate /dev/sda /dev/sdb
```

Vérifiez avec :

```bash
sudo pvs
```

### Ajouter les Volumes Physiques à un Groupe de Volumes

Créez un groupe de volumes (par exemple, `LVMVolGroup`) et ajoutez les volumes physiques :

```bash
sudo vgcreate LVMVolGroup /dev/sda /dev/sdb
```

Listez les groupes de volumes avec :

```bash
sudo vgs
```

### Créer des Volumes Logiques à partir du Groupe de Volumes

Créez des volumes logiques avec des tailles spécifiques :

```bash
sudo lvcreate -L 10G -n projets LVMVolGroup
sudo lvcreate -L 5G -n www LVMVolGroup
sudo lvcreate -L 20G -n db LVMVolGroup
sudo lvcreate -l 100%FREE -n workspace LVMVolGroup
```

Vérifiez la configuration :

```bash
sudo vgs -o +lv_size,lv_name
```

### Formater et Monter les Volumes Logiques

Formatez les volumes logiques avec le système de fichiers Ext4 :

```bash
sudo mkfs.ext4 /dev/LVMVolGroup/projets
sudo mkfs.ext4 /dev/LVMVolGroup/www
sudo mkfs.ext4 /dev/LVMVolGroup/db
sudo mkfs.ext4 /dev/LVMVolGroup/workspace
```

Créez des points de montage :

```bash
sudo mkdir -p /mnt/{projets,www,db,workspace}
```

Montez les volumes logiques :

```bash
sudo mount /dev/LVMVolGroup/projets /mnt/projets
sudo mount /dev/LVMVolGroup/www /mnt/www
sudo mount /dev/LVMVolGroup/db /mnt/db
sudo mount /dev/LVMVolGroup/workspace /mnt/workspace
```

Ajoutez les montages dans `/etc/fstab` pour qu'ils soient persistants :

```plaintext
/dev/LVMVolGroup/projets /mnt/projets ext4 defaults,nofail 0 0
/dev/LVMVolGroup/www /mnt/www ext4 defaults,nofail 0 0
/dev/LVMVolGroup/db /mnt/db ext4 defaults,nofail 0 0
/dev/LVMVolGroup/workspace /mnt/workspace ext4 defaults,nofail 0 0
```

---

## Conclusion

Vous comprenez maintenant les bases de la gestion des volumes avec LVM. Vous pouvez configurer des périphériques de stockage flexibles et dynamiques, adaptés à vos besoins spécifiques.

Pour approfondir, consultez des guides supplémentaires, comme celui sur l'utilisation de LVM avec Ubuntu 18.04.

source : DigitalOcean.com
