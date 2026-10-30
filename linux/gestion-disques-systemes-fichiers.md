# Gestion des disques et des systèmes de fichiers Linux

## 1. Introduction

Linux dispose de plusieurs mécanismes permettant de gérer les disques, les partitions, les systèmes de fichiers et les points de montage.

La gestion du stockage est une compétence importante pour administrer un serveur Linux et maintenir un environnement de développement.

Elle permet notamment de :

- Identifier les disques et les partitions.
- Comprendre les systèmes de fichiers.
- Contrôler l'espace disque disponible.
- Identifier les répertoires qui occupent le plus d'espace.
- Monter et démonter des systèmes de fichiers.
- Configurer des montages permanents.
- Comprendre les volumes utilisés par Docker et Kubernetes.
- Diagnostiquer les problèmes de stockage.

**Objectif :** savoir identifier, inspecter et gérer les ressources de stockage d'un système Linux.

---

## 2. Comprendre l'organisation du stockage

### 2.1. Disque, partition et système de fichiers

Il est important de distinguer trois notions.

**Le disque**

Un disque est un périphérique de stockage physique ou virtuel.

Exemples :

- Disque dur HDD.
- Disque SSD.
- Disque virtuel dans une machine virtuelle.
- Volume de stockage présenté par une infrastructure cloud.

Sous Linux, les disques sont généralement représentés par des fichiers spéciaux dans `/dev`.

Exemples :

```text
/dev/sda
/dev/sdb
/dev/nvme0n1
```

**La partition**

Une partition est une division logique d'un disque.

Un disque peut contenir plusieurs partitions.

Exemple :

```text
/dev/sda
├── /dev/sda1
├── /dev/sda2
└── /dev/sda3
```

**Le système de fichiers**

Un système de fichiers organise les données sur une partition ou un autre périphérique de stockage.

Il permet notamment de :

- Créer des fichiers et des répertoires.
- Organiser les données.
- Gérer les métadonnées.
- Gérer les permissions.
- Retrouver les données stockées.

Exemples de systèmes de fichiers :

| Système | Caractéristiques |
|---|---|
| ext4 | Très répandu sous Linux. |
| XFS | Fréquent sur les serveurs et systèmes RHEL. |
| Btrfs | Système moderne proposant notamment snapshots et sous-volumes. |
| FAT32 | Compatible avec de nombreux systèmes, mais avec certaines limitations. |
| exFAT | Courant pour les supports amovibles. |
| NTFS | Principalement utilisé sous Windows. |

### 2.2. Schéma simplifié

```text
Disque physique ou virtuel
          |
          v
      Partition
          |
          v
   Système de fichiers
          |
          v
    Point de montage
          |
          v
 Arborescence Linux
```

Exemple :

```text
Disque : /dev/sdb
Partition : /dev/sdb1
Système de fichiers : ext4
Point de montage : /data
```

Les fichiers stockés sur cette partition seront accessibles depuis le répertoire `/data`.

Attention : tous les systèmes de stockage ne reposent pas nécessairement sur une partition classique. Linux peut également utiliser LVM, RAID, des volumes chiffrés ou des systèmes de fichiers réseau.

---

## 3. Identifier les disques et les partitions

### 3.1. La commande lsblk

`lsblk` permet d'afficher les périphériques de stockage sous forme d'arborescence.

```bash
lsblk
```

Pour obtenir davantage d'informations :

```bash
lsblk -f
```

Cette commande permet notamment d'afficher :

- Le nom du périphérique.
- Le type de périphérique.
- Le système de fichiers.
- L'UUID.
- Le point de montage.

Exemple de résultat :

```text
NAME   FSTYPE LABEL UUID                                 MOUNTPOINTS
sda
├─sda1 ext4         1234-5678                            /
└─sda2 swap         abcd-1234                            [SWAP]
sdb
└─sdb1 ext4         efgh-5678                            /data
```

Les valeurs ci-dessus sont illustratives.

### 3.2. La commande fdisk

`fdisk` permet d'examiner et de gérer les tables de partitions.

Pour afficher les partitions :

```bash
sudo fdisk -l
```

Cette commande affiche notamment :

- Les disques détectés.
- Leur capacité.
- Le type de table de partitions.
- Les partitions présentes.
- Les secteurs utilisés.

Attention : `fdisk` permet également de modifier les partitions. Une mauvaise manipulation peut entraîner une perte de données.

Pour consulter les informations uniquement, privilégier la commande avec l'option `-l`.

### 3.3. La commande blkid

`blkid` permet d'identifier les systèmes de fichiers et leurs UUID.

```bash
sudo blkid
```

Exemple :

```text
/dev/sda1: UUID="1234-5678" TYPE="ext4"
/dev/sda2: UUID="abcd-1234" TYPE="swap"
```

L'UUID est un identifiant unique du système de fichiers.

Il est particulièrement utile pour configurer les montages permanents dans `/etc/fstab`.

---

## 4. Comprendre l'espace disque

Il existe plusieurs commandes permettant de contrôler l'utilisation du stockage.

### 4.1. df : espace disque disponible

La commande `df` permet de connaître l'espace utilisé et disponible sur les systèmes de fichiers montés.

```bash
df
```

Pour obtenir un affichage lisible :

```bash
df -h
```

Pour afficher également le type de système de fichiers :

```bash
df -Th
```

Exemple :

```text
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sda1      ext4   50G   30G   18G  63% /
/dev/sdb1      ext4  100G   40G   55G  43% /data
```

Interprétation :

- `Size` : capacité totale.
- `Used` : espace utilisé.
- `Avail` : espace disponible.
- `Use%` : pourcentage d'utilisation.
- `Mounted on` : point de montage.

**Cas d'utilisation :**

Lorsqu'une application Spring Boot ne parvient plus à écrire ses fichiers de logs, une première vérification consiste à examiner l'espace disque disponible.

```bash
df -h
```

### 4.2. du : taille des fichiers et répertoires

Contrairement à `df`, qui examine l'utilisation des systèmes de fichiers, `du` permet d'estimer l'espace occupé par des fichiers et des répertoires.

Afficher la taille d'un répertoire :

```bash
du -sh /var/log
```

Afficher la taille des sous-répertoires :

```bash
du -h --max-depth=1 /var/log
```

Trier les résultats par taille :

```bash
du -h --max-depth=1 /var/log | sort -h
```

Afficher les dix plus gros fichiers d'un répertoire :

```bash
find /var/log -type f -printf '%s %p\n' | sort -nr | head -10
```

Attention : cette dernière commande utilise les extensions GNU de `find` et ne fonctionne pas nécessairement sur tous les systèmes Unix.

### 4.3. Différence entre df et du

| Commande | Rôle |
|---|---|
| `df -h` | Affiche l'espace utilisé et disponible sur les systèmes de fichiers. |
| `du -sh` | Affiche l'espace occupé par un fichier ou un répertoire. |

Une différence entre les résultats de `df` et `du` peut notamment provenir de fichiers supprimés mais encore ouverts par un processus, de fichiers cachés dans des points de montage ou de particularités du système de fichiers.

---

## 5. Comprendre les points de montage

### 5.1. Qu'est-ce qu'un point de montage ?

Sous Linux, un système de fichiers doit être monté pour que son contenu soit accessible dans l'arborescence.

Un point de montage est un répertoire dans lequel le système de fichiers est rendu accessible.

Exemple :

```text
/dev/sdb1
    |
    v
  /data
```

Une fois monté, le contenu de la partition est accessible depuis `/data`.

### 5.2. Afficher les systèmes de fichiers montés

```bash
findmnt
```

Afficher les informations d'un point de montage précis :

```bash
findmnt /
```

Afficher les systèmes de fichiers :

```bash
mount
```

Pour un affichage plus synthétique :

```bash
df -Th
```

### 5.3. Monter un système de fichiers

Exemple : monter une partition `/dev/sdb1` dans `/data`.

Créer le répertoire :

```bash
sudo mkdir -p /data
```

Monter la partition :

```bash
sudo mount /dev/sdb1 /data
```

Vérifier :

```bash
findmnt /data
```

Le contenu de la partition devient accessible dans `/data`.

Attention : monter un système de fichiers sur un répertoire déjà utilisé peut masquer temporairement son contenu existant. Les données ne sont pas supprimées, mais elles deviennent invisibles tant que le montage est actif.

### 5.4. Démonter un système de fichiers

Pour démonter une partition :

```bash
sudo umount /data
```

Attention : la commande s'appelle `umount`, et non `unmount`.

Si le système indique que le périphérique est occupé :

```text
target is busy
```

Cela signifie généralement qu'un processus utilise encore le système de fichiers.

Pour identifier les processus concernés :

```bash
sudo lsof +f -- /data
```

Ou :

```bash
sudo fuser -vm /data
```

Il faut ensuite fermer les fichiers ou arrêter proprement les processus concernés avant de démonter le système de fichiers.

---

## 6. Configurer les montages permanents avec /etc/fstab

### 6.1. Pourquoi utiliser /etc/fstab ?

Un montage effectué manuellement avec `mount` n'est généralement pas conservé après un redémarrage.

Le fichier `/etc/fstab` permet de définir les systèmes de fichiers qui doivent être montés automatiquement.

Il contient les informations nécessaires au montage des partitions et autres systèmes de fichiers.

Emplacement :

```text
/etc/fstab
```

### 6.2. Structure du fichier

Une entrée classique contient six champs :

```text
<device> <mount_point> <filesystem> <options> <dump> <pass>
```

Exemple :

```text
UUID=1234-5678 /data ext4 defaults 0 2
```

Explication :

| Champ | Rôle |
|---|---|
| Device | Périphérique ou UUID du système de fichiers. |
| Mount point | Répertoire de montage. |
| Filesystem | Type de système de fichiers. |
| Options | Options de montage. |
| Dump | Indication utilisée par l'ancien mécanisme de sauvegarde `dump`. |
| Pass | Ordre de vérification du système de fichiers avec `fsck` au démarrage. |

### 6.3. Pourquoi privilégier UUID ?

Il est préférable d'utiliser l'UUID plutôt que le nom du périphérique.

Exemple :

```text
UUID=1234-5678 /data ext4 defaults 0 2
```

Les noms comme `/dev/sdb1` peuvent changer selon l'ordre de détection des périphériques.

L'UUID permet d'identifier le système de fichiers de manière plus stable.

Pour récupérer l'UUID :

```bash
sudo blkid
```

### 6.4. Options de montage courantes

| Option | Rôle |
|---|---|
| `defaults` | Utilise les options de montage par défaut. |
| `ro` | Monte en lecture seule. |
| `rw` | Monte en lecture et écriture. |
| `noexec` | Empêche l'exécution de fichiers depuis ce système de fichiers. |
| `nosuid` | Ignore les bits SUID et SGID. |
| `nodev` | N'interprète pas les fichiers spéciaux de périphériques. |
| `nofail` | Permet notamment de ne pas bloquer le démarrage si le périphérique est absent, selon le contexte. |

Certaines options doivent être choisies avec attention selon l'usage du stockage.

### 6.5. Modifier /etc/fstab

Avant toute modification :

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Modifier le fichier :

```bash
sudo nano /etc/fstab
```

Après modification, il est recommandé de vérifier la syntaxe et les montages :

```bash
sudo findmnt --verify
```

Puis tester les entrées de montage :

```bash
sudo mount -a
```

Attention : une erreur dans `/etc/fstab` peut entraîner des problèmes au démarrage. Il faut toujours conserver une sauvegarde et tester la configuration avant de redémarrer.

---

## 7. Les systèmes de fichiers

### 7.1. ext4

ext4 est l'un des systèmes de fichiers les plus utilisés sous Linux.

Ses caractéristiques :

- Bonne stabilité.
- Bonnes performances générales.
- Journalisation des métadonnées.
- Large compatibilité avec les distributions Linux.

Il est adapté à de nombreux usages courants.

### 7.2. XFS

XFS est un système de fichiers fréquemment utilisé dans les environnements serveurs, notamment sur les distributions Red Hat.

Il est conçu pour gérer efficacement de grands volumes de données et de gros fichiers.

### 7.3. Btrfs

Btrfs propose des fonctionnalités avancées :

- Snapshots.
- Sous-volumes.
- Compression.
- Contrôle d'intégrité des données.
- Gestion de plusieurs périphériques.

Il peut être intéressant pour des environnements nécessitant des mécanismes avancés de gestion du stockage.

### 7.4. Identifier le système de fichiers utilisé

```bash
lsblk -f
```

Ou :

```bash
df -Th
```

### 7.5. Formater une partition

La commande `mkfs` permet de créer un système de fichiers.

Exemple pour ext4 :

```bash
sudo mkfs.ext4 /dev/sdb1
```

**Attention : cette opération détruit les données existantes présentes sur le système de fichiers ciblé.**

Ne jamais exécuter une commande de formatage sans avoir vérifié précisément le périphérique concerné et sauvegardé les données importantes.

---

## 8. LVM : Logical Volume Manager

### 8.1. Pourquoi utiliser LVM ?

LVM est un mécanisme permettant de gérer le stockage de manière plus flexible que les partitions classiques.

Il permet notamment de :

- Regrouper plusieurs volumes physiques.
- Créer des volumes logiques.
- Agrandir certains volumes.
- Réorganiser le stockage plus facilement.
- Créer des snapshots.

Il est particulièrement utilisé dans les environnements serveurs.

### 8.2. Les composants de LVM

LVM repose sur trois notions principales.

**Physical Volume (PV)**

Un volume physique correspond à un périphérique ou une partition initialisée pour être utilisée par LVM.

Exemple :

```text
/dev/sdb1
```

**Volume Group (VG)**

Un groupe de volumes regroupe un ou plusieurs volumes physiques.

Exemple :

```text
VG : vg_data
```

**Logical Volume (LV)**

Un volume logique est créé à partir de l'espace disponible dans un groupe de volumes.

Exemple :

```text
VG : vg_data
 |
 +---- LV : lv_database
 |
 +---- LV : lv_logs
```

Un système de fichiers peut ensuite être créé sur un volume logique et monté dans l'arborescence.

### 8.3. Schéma simplifié

```text
+---------------------+
|     Disque /dev/sdb |
+---------------------+
          |
          v
+---------------------+
|  Physical Volume    |
|       /dev/sdb1     |
+---------------------+
          |
          v
+---------------------+
|    Volume Group     |
|      vg_data        |
+---------------------+
          |
          +------------------+
          |                  |
          v                  v
+----------------+  +----------------+
| Logical Volume |  | Logical Volume |
|  lv_database   |  |    lv_logs     |
+----------------+  +----------------+
```

### 8.4. Commandes LVM utiles

Afficher les volumes physiques :

```bash
sudo pvs
```

Afficher les groupes de volumes :

```bash
sudo vgs
```

Afficher les volumes logiques :

```bash
sudo lvs
```

Afficher une vue détaillée :

```bash
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay
```

Ces commandes permettent d'inspecter une infrastructure LVM existante.

La création et l'extension de volumes LVM nécessitent une bonne compréhension du stockage sous-jacent. Elles ne sont pas détaillées ici afin d'éviter les manipulations risquées sur un système réel.

---

## 9. Le stockage dans Docker et Kubernetes

La gestion des disques est directement liée à l'utilisation des conteneurs.

### 9.1. Stockage Docker

Par défaut, les données écrites dans le système de fichiers interne d'un conteneur sont liées au cycle de vie de ce conteneur.

Pour conserver les données indépendamment de la suppression d'un conteneur, Docker propose notamment :

- Les volumes Docker.
- Les bind mounts.
- Les systèmes de stockage fournis par des plugins.

**Volume Docker**

Un volume est géré par Docker.

Exemple :

```bash
docker volume create postgres_data
```

Lister les volumes :

```bash
docker volume ls
```

Afficher les informations d'un volume :

```bash
docker volume inspect postgres_data
```

Utiliser un volume avec PostgreSQL :

```bash
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v postgres_data:/var/lib/postgresql/data \
  postgres
```

Dans cet exemple, les données PostgreSQL sont stockées dans un volume Docker.

Attention : cet exemple est destiné à illustrer le stockage. Dans un environnement réel, il faut utiliser une gestion adaptée des secrets et des mots de passe.

**Bind mount**

Un bind mount permet de partager un répertoire de l'hôte avec un conteneur.

Exemple :

```bash
docker run -d \
  --name nginx \
  -v /opt/nginx-data:/usr/share/nginx/html \
  nginx
```

Le répertoire `/opt/nginx-data` de la machine hôte est accessible depuis le conteneur.

### 9.2. Stockage Kubernetes

Kubernetes propose plusieurs abstractions pour gérer le stockage :

| Objet | Rôle |
|---|---|
| Volume | Espace de stockage monté dans un Pod. |
| PersistentVolume (PV) | Ressource de stockage disponible dans le cluster. |
| PersistentVolumeClaim (PVC) | Demande de stockage formulée par une application. |
| StorageClass | Définit une classe de stockage et permet notamment le provisionnement dynamique. |

Le PVC permet à une application de demander un espace de stockage sans devoir connaître tous les détails de son infrastructure physique.

Exemple de principe :

```text
Application
    |
    v
   Pod
    |
    v
   PVC
    |
    v
   PV
    |
    v
Stockage physique ou distant
```

Cette abstraction permet de séparer les besoins de stockage de l'application de leur implémentation technique.

Documentation officielle :

https://kubernetes.io/docs/concepts/storage/

---

## 10. Diagnostic des problèmes de stockage

### 10.1. Un disque est plein

Symptôme :

```text
No space left on device
```

Première vérification :

```bash
df -h
```

Identifier les répertoires les plus volumineux :

```bash
sudo du -h --max-depth=1 /var | sort -h
```

Puis approfondir l'analyse dans le répertoire concerné.

### 10.2. L'espace disque semble plein alors que les fichiers ne sont pas volumineux

Il est possible qu'un processus conserve ouvert un fichier qui a été supprimé.

Dans ce cas, le fichier n'apparaît plus dans l'arborescence, mais son espace disque n'est pas encore libéré.

Identifier les fichiers supprimés encore ouverts :

```bash
sudo lsof +L1
```

La libération de l'espace intervient généralement lorsque le processus ferme le fichier ou s'arrête.

Il faut identifier le processus concerné et le traiter proprement.

### 10.3. Un système de fichiers n'est pas monté

Vérifier :

```bash
lsblk -f
findmnt
```

Examiner la configuration :

```bash
cat /etc/fstab
```

Vérifier les erreurs de montage :

```bash
sudo mount -a
```

Consulter les journaux du système :

```bash
sudo journalctl -b
```

### 10.4. Les inodes sont épuisés

Un système de fichiers peut manquer d'inodes même si de l'espace disque est encore disponible.

Les inodes permettent notamment de stocker les métadonnées des fichiers.

Cela peut arriver lorsqu'un répertoire contient un très grand nombre de petits fichiers.

Vérifier leur utilisation :

```bash
df -i
```

Si le nombre d'inodes disponibles est nul, il faut identifier les répertoires contenant trop de petits fichiers.

---

## 11. Bonnes pratiques

- Toujours vérifier le périphérique avant une opération de partitionnement ou de formatage.
- Sauvegarder les données avant toute modification importante.
- Utiliser les UUID dans `/etc/fstab`.
- Vérifier la configuration de `/etc/fstab` avant de redémarrer.
- Contrôler régulièrement l'espace disque avec `df -h`.
- Utiliser `du` pour identifier les répertoires volumineux.
- Surveiller également les inodes avec `df -i`.
- Ne pas supprimer manuellement des fichiers système sans comprendre leur rôle.
- Utiliser des volumes persistants pour les bases de données conteneurisées.
- Ne pas confondre stockage temporaire et stockage persistant.
- Prévoir des sauvegardes indépendantes des volumes utilisés par les applications.

---

## 12. Exercices pratiques

### Exercice 1 : identifier les disques

Exécuter :

```bash
lsblk
lsblk -f
sudo blkid
```

Objectifs :

- Identifier les disques.
- Identifier les partitions.
- Connaître les systèmes de fichiers.
- Comprendre le rôle des UUID.

### Exercice 2 : analyser l'espace disque

Exécuter :

```bash
df -h
df -Th
df -i
```

Puis :

```bash
du -sh /var/log
du -h --max-depth=1 /var | sort -h
```

Objectifs :

- Comprendre l'utilisation des disques.
- Identifier les répertoires volumineux.
- Comprendre la différence entre espace disque et inodes.

### Exercice 3 : explorer les montages

Exécuter :

```bash
findmnt
mount
cat /etc/fstab
```

Objectifs :

- Identifier les systèmes de fichiers montés.
- Comprendre les points de montage.
- Examiner les montages configurés au démarrage.

### Exercice 4 : manipuler un volume Docker

Si Docker est installé :

```bash
docker volume create test_volume
docker volume ls
docker volume inspect test_volume
```

Puis supprimer le volume :

```bash
docker volume rm test_volume
```

Objectifs :

- Comprendre le fonctionnement des volumes Docker.
- Différencier volume Docker et répertoire classique de l'hôte.

### Exercice 5 : analyser un problème de stockage

Scénario :

Une API Spring Boot ne parvient plus à écrire dans ses fichiers de logs et retourne une erreur `No space left on device`.

Démarche :

1. Exécuter `df -h`.
2. Identifier le système de fichiers concerné.
3. Examiner les répertoires avec `du`.
4. Vérifier les inodes avec `df -i`.
5. Rechercher les fichiers supprimés mais encore ouverts avec `lsof +L1`.
6. Identifier la cause avant toute suppression.

Objectif : adopter une démarche de diagnostic plutôt que supprimer des fichiers au hasard.

---

## 13. Résumé des commandes

| Commande | Utilité |
|---|---|
| `lsblk` | Lister les disques et partitions. |
| `lsblk -f` | Afficher les systèmes de fichiers et UUID. |
| `fdisk -l` | Examiner les tables de partitions. |
| `blkid` | Identifier les UUID et systèmes de fichiers. |
| `df -h` | Afficher l'espace disque disponible. |
| `df -Th` | Afficher les systèmes de fichiers et leurs types. |
| `df -i` | Contrôler les inodes. |
| `du -sh` | Afficher la taille d'un répertoire. |
| `findmnt` | Afficher les systèmes de fichiers montés. |
| `mount` | Monter ou afficher les systèmes de fichiers. |
| `umount` | Démonter un système de fichiers. |
| `mkfs` | Créer un système de fichiers. |
| `pvs` | Afficher les volumes physiques LVM. |
| `vgs` | Afficher les groupes de volumes LVM. |
| `lvs` | Afficher les volumes logiques LVM. |
| `lsof +L1` | Identifier les fichiers supprimés encore ouverts. |

---

## 14. Ressources complémentaires

- [Linux man-pages](https://man7.org/linux/man-pages/)
- [GNU Coreutils](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [Documentation ext4](https://docs.kernel.org/filesystems/ext4/)
- [Documentation XFS](https://docs.kernel.org/filesystems/xfs/)
- [LVM - Red Hat Documentation](https://docs.redhat.com/)
- [Docker - Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Kubernetes - Storage](https://kubernetes.io/docs/concepts/storage/)

---

## Conclusion

La gestion des disques et des systèmes de fichiers est essentielle pour comprendre et administrer un environnement Linux.

La maîtrise de `lsblk`, `df`, `du`, `mount`, `findmnt` et `/etc/fstab` permet d'identifier les ressources de stockage et de diagnostiquer de nombreux problèmes.

Ces connaissances seront particulièrement utiles pour administrer les serveurs, gérer les données des applications Spring Boot et comprendre le fonctionnement du stockage dans Docker et Kubernetes.