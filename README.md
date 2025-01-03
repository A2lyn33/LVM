# 🖥️ Installation d'un système avec LVM

## 📚 Introduction

LVM (Logical Volume Manager) est une méthode flexible de gestion des volumes de stockage sous Linux. Cette technologie permet de créer des partitions dynamiques, facilement redimensionnables et gérables, offrant une grande souplesse dans la gestion des disques.

---

## 📖 Rappel sur le Boot

Lors du démarrage d'un système, le BIOS ou UEFI ne comprend pas LVM. De ce fait :
- **GNU Grub**, le chargeur d’amorçage Linux, peut gérer LVM mais la plupart des distributions préfèrent créer une partition `/boot` distincte hors du LVM.
- Cette partition `/boot` est utilisée pour contenir les fichiers nécessaires au démarrage avant que le noyau Linux ne prenne le relais pour initialiser les volumes logiques (LV).

---

## 🔬 Exercice : Installation de Debian avec LVM

### 🛠️ Étapes d'installation :

1. **Préparation :**
   - Téléchargez l’image ISO de Debian depuis le site officiel.
   - Créez une machine virtuelle (ou utilisez une machine physique de test).

2. **Partitionnement :**
   - Lors de l'installation, choisissez :  
     **Assisté - utiliser tout un disque avec LVM**.
   - Optez pour une partition `/home` séparée.  
     Cela crée deux volumes logiques distincts : `/` et `/home`.

3. **Vérification après installation :**
   - Utilisez les commandes suivantes pour inspecter la configuration LVM :

   ```bash
   lsblk  # Liste des périphériques blocs et volumes logiques
   pvs    # Liste des volumes physiques (PV)
   vgs    # Liste des groupes de volumes (VG)
   lvs    # Liste des volumes logiques (LV)
   ```

   Exemple de sortie avec `lsblk` :

   ```plaintext
   NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda                     8:0    0    8G  0 disk
   ├─sda1                  8:1    0  487M  0 part /boot
   ├─sda2                  8:2    0    1K  0 part
   └─sda5                  8:5    0  7.5G  0 part
     ├─debian--vg-root   254:0    0  2.8G  0 lvm  /
     ├─debian--vg-swap_1 254:1    0  976M  0 lvm  [SWAP]
     └─debian--vg-home   254:2    0  3.8G  0 lvm  /home
   ```

---

## 🔍 Exploration du Partitionnement

- **Schéma de partitionnement (MBR) :**
  - Une partition primaire `/boot`.
  - Une partition logique pour le volume physique (PV) LVM.
  
  Exemple de sortie avec `fdisk` :

  ```plaintext
  /dev/sda1    *           2048   999423   997376   487M 83 Linux
  /dev/sda2             1001470 16775167 15773698   7.5G  5 Étendue
  /dev/sda5             1001472 16775167 15773696   7.5G 8e LVM Linux
  ```

- **Analyse de LVM :**
  - Volume group : `debian-vg`
  - Physical volume : `/dev/sda5`
  - Logical volumes :
    - `root` monté sur `/`
    - `home` monté sur `/home`
    - `swap_1` pour le swap

---

## 🛠️ Commandes LVM : Manipulations

LVM propose une variété de commandes organisées par cible (`pv`, `vg`, `lv`) et par action (`create`, `display`, `remove`, etc.).

### 📋 Commandes courantes :

1. **Afficher les informations :**
   ```bash
   pvs          # Vue d'ensemble des volumes physiques
   vgs          # Vue d'ensemble des groupes de volumes
   lvs          # Vue d'ensemble des volumes logiques
   ```

2. **Ajouter un nouveau volume logique :**
   ```bash
   lvcreate -L 5G -n lvdata debian-vg
   mkfs.ext4 /dev/debian-vg/lvdata
   mount /dev/debian-vg/lvdata /mnt
   ```

3. **Faire un snapshot d’un volume logique :**
   ```bash
   lvcreate -L 1G -s -n snapshot /dev/debian-vg/root
   ```

4. **Supprimer un volume logique :**
   ```bash
   lvremove /dev/debian-vg/lvdata
   ```

---

## 🧪 Expérimentations

1. **Ajouter un nouveau disque au VG :**
   - Ajouter un disque à la VM.
   - Initialiser le disque comme PV.
   - Ajouter au VG existant :
     ```bash
     pvcreate /dev/sdb
     vgextend debian-vg /dev/sdb
     ```

2. **Conversion d’un LV en RAID :**
   - RAID 1 (mirroring) :
     ```bash
     lvconvert --type raid1 -m1 /dev/debian-vg/lvdata
     ```

   - RAID 0 (striping) :
     ```bash
     lvconvert --type raid0 /dev/debian-vg/lvdata
     ```

3. **Snapshot et restauration :**
   - Créer un snapshot :
     ```bash
     lvcreate -s -L 1G -n snapshot /dev/debian-vg/lvdata
     mount /dev/debian-vg/snapshot /mnt/snapshot
     ```

---

## ☝️ Résumé

- **LVM** offre une gestion flexible des partitions en utilisant des concepts comme PV, VG et LV.
- Les partitions peuvent être redimensionnées, ajoutées ou supprimées dynamiquement sans redémarrage.
- Expérimenter avec LVM sur une machine virtuelle est un excellent moyen de comprendre ses capacités.

Pour approfondir, consultez la [documentation Ubuntu-fr sur LVM](https://doc.ubuntu-fr.org/lvm).

---
