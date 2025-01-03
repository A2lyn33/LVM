# 🛠️ Création 

Bien, passons à la pratique !

Puisque nous allons entrer toutes les commandes à venir en tant que root, je vous conseille de passer root « une bonne fois pour toutes » :

```bash
sudo -i
```

Voilà : le prompt est maintenant en mode root « # », cela nous évitera d'avoir à préfixer chaque commande par `sudo`.

---

## 🔨 Commandes LVM

Bonne nouvelle, les commandes LVM sont extrêmement simples à utiliser, et elles intègrent toutes une aide en ligne très bien conçue, claire, courte, mais suffisante. De plus, leurs noms se « devinent » assez facilement :

- Toutes les commandes agissant sur les **volumes physiques** commencent par `pv` (pour **physical volume**).
- Toutes les commandes agissant sur les **groupes de volumes** commencent par `vg` (pour **volume group**).
- Toutes les commandes agissant sur les **volumes logiques** commencent par `lv` (pour **logical volume**).

La première chose à faire est de créer un volume physique, en attribuant une partition à LVM.

### 🔍 Liste des commandes disponibles pour les volumes physiques :

Essayez la commande suivante pour connaître la liste des commandes disponibles pour les volumes physiques :

```bash
man -k pv
```

Parmi toutes les commandes renvoyées, on remarque une `pvcreate`, ça doit être celle-là… ;-)

### 🚀 Commande pour créer un volume physique

```bash
pvcreate /dev/sdc1
```

Cela nous permettra de l'utiliser dans notre groupe de volume.

> **Note** : La création d'un volume physique avec un disque complet comme `/dev/sdc` n'est pas recommandée. Voir la documentation pour plus d'informations : [LVM-HOWTO](https://tldp.org/HOWTO/LVM-HOWTO/initdisks.html)

---

## 📦 Groupe de volumes

Il existe de nombreuses options lors de la création d'un groupe de volumes… Mais continuons de faire au plus simple. Le seul paramètre indispensable sera de lui donner un nom, nous utiliserons les valeurs par défaut pour tout le reste. Pour une raison que j'expliquerai par la suite, donnons-lui un nom très court (2 ou 3 caractères). Par exemple : « mvg » pour « mon vg ».

Pour connaître la syntaxe de la commande `vgcreate` (comme pour toutes les autres commandes LVM), tapez simplement son nom :

```bash
vgcreate
```

### 📋 Syntaxe de la commande :

```bash
vgcreate VolumeGroupName PhysicalVolume [optionnellement d'autres PhysicalVolume]
```

### 🚀 Commande pour créer un groupe de volumes :

```bash
vgcreate mvg /dev/sdc1
```

Si tout se passe bien, vous avez maintenant un groupe de volumes, contenant un disque physique. Vous pouvez obtenir d'autres informations sur ce groupe de volumes en tapant :

```bash
vgdisplay
```

---

## 📂 Volume logique

Nous y voilà… Cette fois-ci, nous allons vraiment créer deux espaces que l'on pourra ensuite « formater » en ext4 par exemple.

### 🔍 Vérification de la syntaxe de la commande `lvcreate` :

```bash
lvcreate
```

Les deux options vraiment importantes sont `-n` pour son nom, et `-L` pour sa taille. Le paramètre principal est « **OriginalLogicalVolume** ». Il s'agit peut-être d'une erreur dans le manuel (man). En fait, ce qu'il faut indiquer, c'est bien le groupe de volumes dans lequel nous allons créer le volume logique.

Pour l'exemple présent, je fais ici deux volumes, 10 Gio et 50 Gio :

```bash
lvcreate -n Vol1 -L 10g mvg
lvcreate -n Vol2 -L 50g mvg
```

Idem, on peut vérifier avec la commande `lvdisplay` :

```bash
lvdisplay
```

---

## 📁 Système de fichiers

Avec les partitions, on avait des noms ressemblant à `/dev/sda3`, etc. Avec LVM, on utilise aussi des périphériques dans `/dev`, mais le chemin est de la forme `/dev/nom_du_vg/nom_du_lv`. Autrement dit, puisqu'on a décidé d'appeler nos volumes logiques "Vol1" et "Vol2", les noms de ces périphériques de ce volume logique sont `/dev/mvg/Vol1` et `/dev/mvg/Vol2`.

### 🚀 Exemple de commande pour formater et monter :

```bash
mkfs -t ext4 /dev/mvg/Vol1
mkfs -t ext4 /dev/mvg/Vol2
mkdir /Essai1
mount /dev/mvg/Vol1 /Essai1
df -h
```

Et normalement, `/dev/mvg/Vol1` devrait être monté sur `/Essai1`. Vérifiez la sortie de la commande `df`.

> **Note** : Si vous choisissez un nom de VG ou de LV plus long, la sortie de `df` serait modifiée, car le nom serait « long » et toucherait d'autres valeurs, rendant l'affichage plus difficile à lire. C'est pour cela qu'il est conseillé de se limiter à 7 caractères au total pour une meilleure lisibilité.

---

## 🧐 Pourquoi utiliser `/dev/mapper/mvg-Vol1` et non `/dev/mvg/Vol1` ?

Avec LVM en version 1, c'est bien `/dev/mvg/Vol1` qui aurait été affiché. Depuis la version 2, LVM utilise le périphérique **mapper**, ce qui permet de faire plus de choses (comme chiffrer les volumes logiques, etc.).

En résumé, ces deux notations :

```bash
/dev/mvg/Vol1
```

et

```bash
/dev/mapper/mvg-Vol1
```

sont synonymes, mais il est conseillé d'utiliser la première forme (`/dev/mvg/Vol1`), car certaines commandes ne fonctionneront pas avec l'autre notation.
