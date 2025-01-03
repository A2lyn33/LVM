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

Voici le texte avec des icônes ajoutées pour améliorer la lisibilité et l'attractivité dans un fichier `.md` :

```markdown
# 📂 Système de fichiers

Avec les partitions, on avait des noms ressemblant à /dev/sda3, etc. Avec LVM, on utilise aussi des périphériques dans /dev, mais le chemin est de la forme `/dev/nom_du_vg/nom_du_lv`. Autrement dit, puisqu'on a décidé d'appeler nos volumes logiques "Vol1" et "Vol2", les noms de ces périphériques de ce volume logique sont "/dev/mvg/Vol1" et "/dev/mvg/Vol2". À partir de maintenant, `/dev/mvg/Volx` peut être utilisé dans toutes les situations et avec toutes les commandes qui attendent quelque chose de la forme `/dev/...`. Par exemple :

```bash
mkfs -t ext4 /dev/mvg/Vol1
mkfs -t ext4 /dev/mvg/Vol2
mkdir /Essai1
mount /dev/mvg/Vol1 /Essai1
df -h
```

Et normalement, `/dev/mvg/Vol1` devrait être monté sur `/Essai`. Regardez bien la ligne correspondante. Si on avait choisi un nom de VG ou de LV plus long, la sortie de `df` aurait été modifiée, car le nom aurait « touché » les valeurs… On aurait été obligé de passer des lignes et l'affichage aurait été plus difficile à lire. Techniquement, choisir des noms « longs » pour les VG et les LV ne pose aucun problème, mais c'est l'affichage qui sera parfois délicat. Pour cette raison uniquement, je préconise de se limiter à 7 caractères au total (donc par exemple 3 pour le VG et 4 pour le LV, ou 2 et 5, comme vous voulez).

### ❓ Pourquoi est-il écrit `/dev/mapper/mvg-Vol1` et non `/dev/mvg/Vol1` ?

Avec LVM en version 1, c'est bien `/dev/mvg/Vol1` qui aurait été affiché. Depuis la version 2, LVM utilise le périphérique mapper, ce qui permet pas mal de choses (comme chiffrer les volumes logiques, etc.). Pour simplifier, disons que ces deux notations « `/dev/mvg/Vol1` » et « `/dev/mappfer/mvg-Vol1` » sont synonymes. Dans la pratique, il est conseillé quand même d'utiliser plutôt la forme « `/dev/mvg/Vol1` », certaines commandes ne passeront pas autrement.

---

# 🗑️ Suppression

Rien de plus simple :

```bash
umount /Essai1  # si le volume Vol1 est monté en /Essai1
lvremove /dev/mvg/Vol1
```

⚠️ Attention, une fois un volume logique effacé, il est totalement impossible de récupérer les données qu'il contenait.

---

# ↗️ Redimensionnement

## 💾 Volume physique

### ➕ Agrandissement

Imaginons maintenant que notre groupe de volume (mvg) n'ait plus suffisamment d'espace libre. On souhaite donc lui rajouter un volume physique afin de rajouter de l'espace libre. Ça tombe bien, on dispose d'un volume physique `sdc2` que l'on va pouvoir ajouter à `mvg` :

On initialise le volume en vue de son utilisation dans LVM :

```bash
pvcreate /dev/sdc2
```

On rajoute le volume `sdc2` au groupe de volume `mvg` :

```bash
vgextend mvg /dev/sdc2
```

### ➖ Rétrécissement

Imaginons maintenant que la partition Boot soit devenue trop petite et que tout le reste du disque qui contient cette partition soit défini comme une partition utilisée en LVM (`/dev/sda2` dans l'exemple). Il sera alors nécessaire de diminuer l'espace physique de ce groupe de volume (`mvg`).

En premier, il est nécessaire de rétrécir le ou les LV qui sont définis dans ce volume-groupe. Voir ci-après.

Puis il faut rétrécir l'enveloppe physique. Normalement, c'est sans risque : les commandes sont refusées si la demande est trop agressive.

Commençons par vérifier l'implantation physique des données dans la partition. Le but est de vérifier que la fin de la partition n'est pas utilisée.

```bash
sudo pvs -v --segments /dev/sda2
```

La dernière ligne de la réponse indique si la fin de la partition est libre ou non. Si la fin de la partition n'a pas suffisamment d'espace libre, il est possible de procéder à une réorganisation physique.

```bash
sudo pvmove --alloc anywhere /dev/sda2:88888888-9999999999 /dev/sda2:0-88888887
```

(Les valeurs `88888887` et `88888888` sont à adapter en fonction de la réponse précédente, notamment la quantité d'espace libre en début de partition. Lancer alors l'éditeur de partitions. Indiquer alors la nouvelle taille de la partition. Le minimum possible est exprimé en MiO et obtenu par la formule `88888888*4`).

---

## 🛠️ Volume logique

Il est très facile d'augmenter ou de diminuer la taille d'un volume logique. Mais attention, la taille d'un LV n'a pas de lien direct avec la taille de ce qu'il contient (swap ou système de fichier). Le LV est une boîte, le système de fichier est le contenu de la boîte. Augmenter la taille de la boîte sans augmenter la taille du contenu ne pose pas de problème, mais l'inverse…

### ➕ Agrandissement

Bien qu'il soit évidemment moins risqué d'agrandir ou de diminuer la taille d'un système de fichiers après l'avoir démonté, la plupart des formats (ext3, reiserfs, ext4…) supportent désormais cette modification "à chaud" (avec des données qui restent donc accessibles en lecture/écriture durant toute l'opération).

Pour agrandir un volume, il est nécessaire de démonter le système de fichier, ici, prenons celui du volume logique `Vol2` :

```bash
umount /Essai2
```

Maintenant, nous allons ajouter 5Gio au volume et agrandir son système de fichier :

```bash
lvresize --resizefs --size +5G /dev/mvg/Vol2
```

Le paramètre `--resizefs` ne fonctionne pas avec tous les systèmes de fichiers (voir ici). Une fois l'opération terminée, le volume, une fois monté, a gagné 5Gio.

Il est également possible d'augmenter la taille du volume logique à l'ensemble de l'espace libre disponible sur le support en utilisant :

```bash
lvextend -l +100%FREE /dev/mvg/Vol2
```

### ➖ Rétrécissement

Diminuer la taille d'un système de fichier est un peu plus délicat. En effet, il faut dans un premier temps s'assurer de pouvoir réduire d'autant qu'on le souhaite.

Tous les systèmes de fichiers ne supportent pas d'être redimensionnés (voir ici). Voyons d'abord l'espace du système de fichier :

```bash
df -h -BM | grep ca
```

```bash
/dev/mapper/svg-ca    512M  230M  283M  45% /home/ca
```

Les valeurs qui nous intéressent sont la deuxième et la quatrième, à savoir :

- 512Mio d'espace total
- 283Mio d'espace libre

L'espace disque étant de 283Mio, nous pouvons réduire l'espace de 256Mio et c'est ce que nous allons faire.

Commençons par démonter le volume :

```bash
umount /dev/mapper/svg-ca
```

Maintenant, nous allons retirer 256Mio :

```bash
lvresize --resizefs --size -256M /dev/mapper/svg-ca
```

Si la partition n'est pas démontée, la commande propose de la démonter et s'occupera de la remonter une fois le redimensionnement terminé. Le volume peut maintenant être monté :

```bash
mount /dev/mapper/svg-ca /home/ca
```

Et on peut alors afficher sa nouvelle taille :

```bash
df -h -BM | grep ca
```

```bash
/dev/mapper/svg-ca    256M  230M   27M  90% /home/ca
```
# ⚠️ Attention
Il est possible que le rétrécissement soit refusé suite à une dé-organisation qui se fait pendant la vie du LVM car la demande est trop importante. Dans ce cas, voici la démarche à effectuer :

## 🔒 Démonter le volume
```bash
sudo umount /dev/mapper/svg-ca
```

## 🛠️ Contrôler la qualité du volume
```bash
sudo e2fsck -f /dev/mapper/svg-ca
```

## 📏 Demander l'espace réel minimum nécessaire
```bash
sudo resize2fs -PM /dev/mapper/svg-ca
```

## 📉 Rétrécir à la taille minimum indiquée
Mettre la valeur retournée par la commande précédente (ou une valeur plus importante) à la place de la valeur `123456789` de cette commande :
```bash
sudo lvresize --resizefs --size $((123456789/256+1))M /dev/mapper/svg-ca
```

## 🔄 Remonter le volume
```bash
mount /dev/mapper/svg-ca /home/ca
```

De même, il est possible de rétrécir une partition logique chiffrée. La procédure est un peu plus longue. Un exemple est disponible [ici](#).

# 💡 Snapshot
## 🧙‍♂️ Comprendre la magie du Snapshot LVM

Pourquoi donner une taille au snapshot ? Tout simplement parce que celui-ci est intelligent, il ne va pas copier l'intégralité du LV original. Au contraire, il ne va stocker que les différences. C'est pourquoi il est instantané et commence avec une occupation taille nulle. La commande `lvdisplay` permet de voir l'évolution de la taille.

### ⚖️ Fonctionnement de LVM 1
Avec LVM 1, les instantanés sont en lecture seule. Ils fonctionnent par l'utilisation d'une table d'exception qui trace les blocs modifiés : lorsqu'un bloc est modifié sur la source, il est d'abord copié dans l'instantané, marqué comme modifié dans la table d'exceptions et ensuite modifié sur le volume source avec les nouvelles données.

### 🔓 Fonctionnement de LVM 2
Avec LVM 2, les instantanés sont par défaut en lecture/écriture. Le fonctionnement est similaire aux instantanés en lecture seule avec la possibilité supplémentaire d'écrire sur l'instantané : le bloc est alors marqué comme utilisé dans la table d'exceptions et ne sera plus récupéré du volume source. Cela ouvre de nouvelles perspectives par rapport au fonctionnement en lecture seule de LVM 1. Par exemple, on peut faire l'instantané d'un volume, le monter et tester un programme expérimental qui modifie les fichiers dessus. Si le résultat n'est pas satisfaisant, on peut le démonter, le supprimer et remonter le système de fichiers originel à la place. C'est aussi utile pour créer des volumes utilisés avec Xen.

Vous pouvez créer une image disque et en faire un instantané que vous pourrez modifier avec une instance spécifique de domU. Vous pourrez ensuite créer un autre instantané de l'image originale et le modifier avec une autre instance de domU. Comme les instantanés ne stockent que les blocs modifiés, la majeure partie du volume sera partagée entre les domUs.

Voir ici pour [sauvegarder son système à chaud avec LVM](#).

## 🛠️ Création d'un snapshot LVM
```bash
lvcreate -L 10g -s -n lv_test_20110617 /dev/vg_data/lv_test
```
Va créer un snapshot du LV `lv_test` à la taille de 10Go qui va avoir comme nom `lv_test_20110617`. Attention, la taille d'utilisation du snapshot évolue avec l'utilisation. Si ce snapshot se retrouve rempli à 100%, il devient alors inutilisable (état "INACTIVE") mais pas d’inquiétude car il n'y a pas d'impact pour le LV d'origine.

## ↔️ Redimensionnement du snapshot
La taille du snapshot est trop petite et elle arrive bientôt à 100%, pourtant vous avez encore besoin d'utiliser ce snap ? Il faut donc redimensionner ! Vérifiez avec `vgdisplay` que le VG dispose encore d'assez d'espace libre (Free PE / Size) puis effectuez :
```bash
lvresize -L +3GB /dev/vg_data/lv_test_20110617
```
Va ajouter 3Go au snap `lv_test_20110617` qui est présent dans le VG `vg_data`.

## 🔄 Fusionner un snapshot
Le but ici est de fusionner un snapshot modifié vers le LV d'origine. Pour ainsi dire, "faire que les modifications apportées sur le snapshot se retrouvent sur le LV d'origine".

```bash
lvconvert --merge /path/to/dev/snap
```
Attention : vous avez besoin d'un kernel (>=2.6.33).

# 🔧 Changement d'un disque défectueux 

Votre disque `/dev/sda` présente des signes de faiblesse (signalés par SMART, par la présence de nombreux fichiers dans les dossiers "lost + found" de vos partitions). Vous désirez le remplacer par un disque neuf, de taille plus importante (surtout pas plus petite !), que vous avez installé dans la machine (ou sur un port USB) et qui est reconnu par le système comme étant `/dev/sdb`.

## 📝 Principe

Supposons que votre disque initial (/dev/sda) ait été formaté ainsi :  
- `/dev/sda1` est une partition primaire, de type bootable, montée sur `/boot`.
- `/dev/sda2` est une partition étendue, contenant la partition logique `/dev/sda5` de type lvm2.

Vous avez besoin de copier `/dev/sda1` sur une partition `/dev/sdb1`, et `/dev/sda5` sur une partition `/dev/sdb5`.

Vous allez utiliser l'outil GParted pour préparer le disque `/dev/sdb` et copier la partition de boot. Gparted ne gérant pas lvm2, nous utiliserons la ligne de commande pour la copie de `/dev/sda5`.

## 💻 Avec GParted

Lancez Gparted (Système → Administration → Editeur de partitions GParted). Les partitions de votre disque `/dev/sda` s'affichent. Notez la taille de `/dev/sda1`, ainsi que son filesystem (ext2/ext3/ext4).

Passez au disque `/dev/sdb`. Créez-y une nouvelle partition primaire `/dev/sdb1`, de taille légèrement supérieure à celle de `/dev/sda1`. "Appliquez" pour que la création soit effective, puis modifiez (par clic droit) les drapeaux de `/dev/sdb1` pour rendre cette partition bootable. Créez une partition étendue `/dev/sdb2`, occupant le reste du disque. Sur cette partition, créez une partition logique `/dev/sdb5` non formatée. "Appliquez" pour que vos créations soient effectives.

Repassez au disque `/dev/sda`. Cliquez-droit sur `/dev/sda1` et choisissez "Démonter" puis "Copier". Repassez au disque `/dev/sdb`. Cliquez-droit sur `/dev/sdb1` et choisissez "Coller" (ou "Paste"). "Appliquez" à nouveau. Fermez GParted.

## 📂 En ligne de commande

Remontez votre partition de boot :  
```bash
sudo mount /boot
```

Faites un scan des volumes physiques de LVM :  
```bash
sudo pvscan
```
Cela signifie que le volume physique (PV) `/dev/sda5` est inclus dans le groupe de volumes (VG) nommé ici `delphy` (bien sûr le vôtre porte un autre nom).

Déclarez `/dev/sdb5` comme volume physique :  
```bash
sudo pvcreate /dev/sdb5
```
Vérifiez qu'il existe bien, mais n'est pas encore attribué à un groupe de volumes :  
```bash
sudo pvscan
```

Attribuez `/dev/sdb5` à votre groupe de volumes (ici `delphy`). Ce groupe de volumes est "étendu" à `/dev/sdb5` :  
```bash
sudo vgextend delphy /dev/sdb5
```

Vérification :  
```bash
sudo pvscan
```

Lancez enfin le déplacement des données, du volume physique `/dev/sda5` vers le volume physique `/dev/sdb5` :  
```bash
sudo pvmove /dev/sda5 /dev/sdb5
```
Attention, l'opération peut prendre du temps (plusieurs heures pour les grosses partitions), suivant la taille des données à transférer, la rapidité des disques, etc.

Vérifiez que le contenu de `/dev/sda5` a bien été transféré sur `/dev/sdb5` :  
```bash
sudo pvscan
```

Supprimez `/dev/sda5` du groupe de volumes `delphy` :  
```bash
sudo vgreduce delphy /dev/sda5
```

Vérifiez :  
```bash
sudo pvscan
```

Enlevez le disque des volumes physiques :  
```bash
sudo pvremove /dev/sda5
```
Vous pouvez désormais enlever le disque.

## ⚙️ Finalisation

Réinstallez GRUB sur le MBR de votre disque dur :  
```bash
sudo grub-install /dev/sdb
```

Éteignez votre ordinateur, enlevez l'ancien disque et remplacez-le par le nouveau, au niveau des branchements.

## 📘 Mieux comprendre ou aller plus loin

### Notion d'« extent »

Un extent, ou « physical extent » aussi appelé « PE », est un tout petit morceau d'un groupe de volumes. En fait, au moment de la création d'un groupe de volumes, le ou les disques sont découpés en morceaux de quelques Mio (4 Mio par défaut). Lorsqu'on crée un volume logique, LVM va utiliser autant de PE que nécessaires. La taille d'un volume logique sera donc toujours un multiple de la taille d'un PE.

### 📚 Glossaire

| Abbreviation | Anglais          | Français             | Description                                  |
|--------------|------------------|----------------------|----------------------------------------------|
| VG           | Volume Group     | Groupe de Volumes     |                                              |
| LV           | Logical Volume   | Volume Logique       | Une "partition" dans un groupe de volumes    |
| PV           | Physical Volume  | Volume Physique      |                                              |
| PE           | Physical Extent  | Etendue Physique     | Un tout petit morceau d'un groupe de volumes |

### ⚡ LVM et RAID

Il est tout à fait possible d'utiliser LVM sur un volume en RAID logiciel. Une fois que le RAID a été créé (`/dev/md0` par exemple), il suffit de le donner à LVM, avec la commande habituelle :  
```bash
pvcreate /dev/md0
```

Bien qu'il soit possible de partitionner le RAID `/dev/md0` comme n'importe quel disque ordinaire (ce qui permet d'obtenir des devices de la forme `/dev/md0p1`, `/dev/md0p2` etc), je vous le déconseille vivement.

### 💡 LVM miroir

Convertir un LVM en miroir :  
```bash
lvconvert -m 1 Volume_Group/Logical_Volume /dev/sdx1 /dev/sdy1
```

Voir l'état du miroir :  
```bash
lvs -a -o +devices
```

### 🖥️ Interface graphique pour LVM

Il existe une interface graphique pour LVM, qui permet de manipuler les volumes logiques : `system-config-lvm`. Cette interface n'est pas disponible avec la version 18.04.

### 🛠️ Monter une partition

Obtenez la liste des groupes logiques :  
```bash
lvm vgscan
```

Obtenez la liste des partitions :  
```bash
lvm lvs
```

Rendre la partition disponible :  
```bash
lvm lvchange -ay /dev/VolGroup01/LogVol00
```

Monter la partition :  
```bash
mount /dev/VolGroup01/LogVol00 /media/user/point_de_montage
```

### 🔍 Vérifier une partition

Rendre la partition disponible :  
```bash
lvm lvchange -ay /dev/VolGroup01/LogVol00
```

Lancer fsck :  
```bash
sudo fsck -f -y /dev/VolGroup01/LogVol00
```

## 📖 Références

- [L'origine de cette page par Hoper – lien mort]()
- Article de Léa Linux un peu vieux (LVM 1) mais plus complet que le mien…
- Article de developpez.com excellent aussi (attention, sauf la partie réduction !)
- [LVM HOW TO (en)](https://www.example.com) un how to assez complet en anglais
- [Guide pratique de LVM (fr)](https://www.example.com) un guide assez complet en français
- Comment chiffrer une partition système Linux
- Gestion des LVM sous Linux sur IT-Connect.fr
- Mise en place LVM tout simplement
- [Lien page Ubuntu](https://doc.ubuntu-fr.org/lvm#introduction)

Contributeurs : Koshie-2.0, claudiux (remplacement disque défectueux), Alexandre LG ; merci à Ner0lph et à tous les autres correcteurs :)
