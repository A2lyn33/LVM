# 💪 Challenge : Opérations classiques sur LVM

---

## 🔧 Étape 1 : Ajout d'un nouveau disque au VG `debian-vg`

1. **Ajoute un nouveau disque à la machine.**
   - Exemple : Un disque de 8 Go ajouté en tant que `/dev/sdb`.

2. **Initialise le disque comme un PV (Physical Volume).**
   ```bash
   sudo pvcreate /dev/sdb
   ```

3. **Ajoute le nouveau PV au groupe de volumes `debian-vg`.**
   ```bash
   sudo vgextend debian-vg /dev/sdb
   ```

4. **Vérifie l'état du VG avant et après l'opération.**
   ```bash
   sudo vgs
   ```

5. **Constat attendu :**
   - Le total des PE (Physical Extents) dans le VG `debian-vg` est au moins doublé.

---

## 📸 Étape 2 : Création d’un snapshot du LV `home`

1. **Crée un snapshot de `home` dans le VG `debian-vg`.**
   ```bash
   sudo lvcreate -L 1G -s -n home-snap /dev/debian-vg/home
   ```

2. **Vérifie l’existence du snapshot.**
   ```bash
   sudo lvs
   ```

3. **Constat attendu :**
   - Le snapshot `home-snap` est visible dans la liste des volumes logiques.

---

## 📂 Étape 3 : Montage du snapshot sur `/home-snap`

1. **Crée un dossier pour monter le snapshot.**
   ```bash
   sudo mkdir /home-snap
   ```

2. **Monte le snapshot.**
   ```bash
   sudo mount /dev/debian-vg/home-snap /home-snap
   ```

3. **Affiche le contenu du dossier monté.**
   ```bash
   ls /home-snap
   ```

4. **Constat attendu :**
   - Le contenu de `/home-snap` est identique à celui de `/home`.

---

## 🧹 Étape 4 : Travail sur `/home-snap` et suppression du snapshot

1. **Effectue des modifications dans `/home-snap` (optionnel).**
   Exemple :
   ```bash
   echo "Modification temporaire" | sudo tee /home-snap/testfile.txt
   ```

2. **Démonte le snapshot.**
   ```bash
   sudo umount /home-snap
   ```

3. **Supprime le snapshot.**
   ```bash
   sudo lvremove /dev/debian-vg/home-snap
   ```

4. **Vérifie que le snapshot est supprimé.**
   ```bash
   sudo lvs
   ```

5. **Constat attendu :**
   - `/home-snap` n’est plus monté.
   - Le LV `home` n’est plus la source d’aucun snapshot.

---

## 🧐 Critères d'acceptation

- L’ajout du PV à `debian-vg` est confirmé par :
  - Le doublement des Total PE dans la commande `vgs`.

- La création et le montage du snapshot sont confirmés par :
  - La présence du LV `home-snap` dans `lvs`.
  - La liste des systèmes de fichiers montés montrant `/home-snap`.
  - Le contenu de `/home-snap` identique à `/home`.

- La suppression du snapshot est confirmée par :
  - L’absence de `/home-snap` dans la liste des systèmes de fichiers montés.
  - L’absence du LV `home-snap` dans `lvs`.

---

🎉 **Challenge terminé !**
