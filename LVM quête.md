# 🖥️ Exercice LVM sur Debian

Le challenge consiste, en partant d'une machine Debian telle que celle installée au début de cette quête, à suivre une succession d'étapes visant à effectuer des opérations classiques sur LVM.

À chaque étape, des commandes doivent être saisies pour afficher l'état du système avant les opérations et après les opérations pour pouvoir constater les modifications effectuées.

Les commandes et leurs affichages sont à recopier dans la solution de cette quête.

## 1️⃣ Étape 1: Ajouter un nouveau disque à la machine et l'ajouter au groupe de volume `debian-vg`

### Commande pour afficher l'état initial des disques :
```bash
lsblk
```
Cela montre les disques et partitions avant d'ajouter un nouveau disque.

### Ajouter le nouveau disque en tant que PV (Physical Volume) :
Supposons que le nouveau disque soit `/dev/sdb`. Nous allons le convertir en un PV et l'ajouter au groupe de volume `debian-vg`.

```bash
sudo pvcreate /dev/sdb
```

### Ajouter le PV au groupe de volume `debian-vg` :
```bash
sudo vgextend debian-vg /dev/sdb
```

### Commande pour vérifier l'extension du VG :
```bash
vgs
```

L'affichage doit maintenant montrer que la taille totale du groupe de volume `debian-vg` a doublé.

## 2️⃣ Étape 2: Créer un snapshot du LV `home`

### Commande pour créer un snapshot de `home` :
```bash
sudo lvcreate -L 10G -s -n home-snap /dev/debian-vg/home
```

Cette commande crée un snapshot de 10 Go du volume logique `home`.

### Commande pour afficher les LV (Logical Volumes) existants :
```bash
lvs
```

L'affichage doit inclure le snapshot `home-snap` créé.

## 3️⃣ Étape 3: Monter le snapshot sur `/home-snap`

### Commande pour créer le dossier de montage :
```bash
sudo mkdir /home-snap
```

### Commande pour monter le snapshot :
```bash
sudo mount /dev/debian-vg/home-snap /home-snap
```

### Vérification du montage avec `df` :
```bash
df -h
```

Cela doit afficher `/home-snap` comme un système de fichiers monté, à côté de `/home`.

## 4️⃣ Étape 4: Vérification que `/home-snap` est bien une copie de `/home`

### Commande pour vérifier le contenu de `/home-snap` :
```bash
ls /home-snap
```

Cela doit afficher le même contenu que le répertoire `/home`.

### Commande pour vérifier le contenu de `/home` :
```bash
ls /home
```

Les deux commandes doivent afficher un contenu identique, ce qui prouve que le snapshot est bien une copie de `/home`.

## 5️⃣ Étape 5: Travailler sur `/home-snap` et effectuer des modifications

À ce stade, vous pouvez apporter des modifications sur `/home-snap` sans affecter le contenu original de `/home`. Par exemple, vous pouvez créer des fichiers, les modifier ou les supprimer.

### Exemple de modification :
```bash
sudo touch /home-snap/fichier_test.txt
sudo ls /home-snap
```

Cela crée un fichier `fichier_test.txt` dans `/home-snap`, ce qui montre que vous pouvez modifier le snapshot indépendamment de `/home`.

## 6️⃣ Étape 6: Démonter `/home-snap` lorsque vous n'en avez plus besoin

### Commande pour démonter le snapshot :
```bash
sudo umount /home-snap
```

### Commande pour vérifier que `/home-snap` n'est plus monté :
```bash
df -h
```

L'affichage ne doit plus mentionner `/home-snap`.

## 7️⃣ Étape 7: Détruire le snapshot

### Commande pour supprimer le snapshot :
```bash
sudo lvremove /dev/debian-vg/home-snap
```

### Commande pour vérifier la suppression du snapshot :
```bash
lvs
```

L'affichage ne doit plus inclure `home-snap` dans la liste des volumes logiques, et le LV `home` ne doit plus avoir de snapshot associé.

---

## ✅ Critères d'acceptation

Les commandes et leurs affichages permettent bien de constater :

- L'ajout d'un PV à `debian-vg` et au moins le doublement des Total PE.
- La création d'un snapshot du LV `home`.
- La création d'un dossier `/home-snap` et le montage du snapshot dans ce dossier.
- L'affichage du contenu de `/home-snap` doit être identique à celui de `/home`.
- L'affichage des systèmes de fichiers actuellement montés ne doit plus afficher `/home-snap`.
- L'affichage des LV ne doit plus afficher le snapshot, et le LV `home` ne doit plus être la source d'aucun snapshot.
