# Architecture Linux et processus de démarrage

## 1. Introduction

Linux est un système d'exploitation de type Unix, utilisé aussi bien sur les serveurs que sur les postes de travail, les systèmes embarqués et les infrastructures cloud.

Pour comprendre son fonctionnement, il est important de connaître son architecture interne, les différents composants qui le constituent et les étapes nécessaires à son démarrage.

Ce chapitre présente :

- L'architecture générale d'un système Linux.
- Le rôle du noyau (Kernel).
- La séparation entre User Space et Kernel Space.
- Les appels système (System Calls).
- Le rôle du Shell.
- L'organisation du système de fichiers.
- Les différentes étapes du démarrage de Linux.
- Le fonctionnement de systemd et des targets.

**Objectif :** comprendre comment les applications interagissent avec le système d'exploitation et comment Linux initialise ses composants au démarrage.

---

## 2. Linux et les distributions

### 2.1. Qu'est-ce que Linux ?

Linux désigne principalement le noyau (Kernel) du système d'exploitation.

Le noyau constitue le composant central du système. Il assure la communication entre les logiciels et les ressources matérielles de la machine.

Il prend notamment en charge :

- La gestion des processus.
- La gestion de la mémoire.
- La gestion des systèmes de fichiers.
- La gestion des périphériques.
- La gestion des communications réseau.

Il est important de distinguer Linux du système complet que l'on utilise au quotidien.

Une distribution Linux regroupe généralement :

- Le noyau Linux.
- Les bibliothèques système.
- Les outils et commandes système.
- Un gestionnaire de paquets.
- Un système d'initialisation.
- Éventuellement un environnement graphique.

Exemples de distributions :

- Ubuntu.
- Debian.
- Fedora.
- Red Hat Enterprise Linux (RHEL).
- Linux Mint.

### 2.2. Identifier sa distribution

Pour connaître la distribution utilisée :

```bash
cat /etc/os-release
```

Pour connaître la version du noyau :

```bash
uname -r
```

Pour obtenir davantage d'informations sur le système :

```bash
uname -a
```

Attention : la version de la distribution et celle du noyau sont deux informations différentes.

---

## 3. Architecture générale de Linux

### 3.1. Les différentes couches

L'architecture Linux peut être représentée en plusieurs couches :

```text
+-------------------------------------------+
|             Applications                  |
|    Java, Docker, Git, PostgreSQL, Nginx   |
+-------------------------------------------+
|         Shell et outils système           |
|          Bash, ls, grep, systemctl        |
+-------------------------------------------+
|          Bibliothèques système            |
|              glibc, etc.                  |
+-------------------------------------------+
|                Kernel                     |
|  Processus, mémoire, réseau, fichiers     |
|              périphériques                |
+-------------------------------------------+
|                Matériel                   |
|       CPU, RAM, disque, réseau             |
+-------------------------------------------+
```

Chaque couche joue un rôle spécifique.

| Couche | Rôle |
|---|---|
| Applications | Exécution des programmes et services. |
| Shell et outils système | Interaction avec le système et exécution des commandes. |
| Bibliothèques système | Fournissent des fonctions utilisables par les applications pour accéder aux services du système. |
| Kernel | Gère les ressources matérielles et fournit les services fondamentaux du système. |
| Matériel | Fournit les ressources physiques de la machine. |

**À retenir :** le Shell n'est pas le noyau. Bash est un programme utilisateur qui permet d'interagir avec le système.

### 3.2. Le noyau Linux (Kernel)

Le Kernel est le cœur du système Linux.

Il assure l'accès contrôlé aux ressources matérielles et fournit les mécanismes nécessaires à l'exécution des applications.

Ses principales responsabilités sont les suivantes.

#### Gestion des processus

Le noyau est responsable de :

- La création et la terminaison des processus.
- L'ordonnancement des processus sur le processeur.
- La gestion des signaux.
- La gestion des changements de contexte.

#### Gestion de la mémoire

Le noyau assure :

- L'allocation et la libération de mémoire.
- La gestion de la mémoire virtuelle.
- La pagination.
- La protection des espaces mémoire des processus.

#### Gestion des systèmes de fichiers

Le noyau fournit les mécanismes permettant :

- L'accès aux fichiers et répertoires.
- La lecture et l'écriture des données.
- La gestion des permissions.
- L'utilisation de différents systèmes de fichiers.

#### Gestion du réseau

Le noyau implémente notamment :

- La pile réseau TCP/IP.
- La gestion des interfaces réseau.
- Les mécanismes de communication entre applications.
- Le traitement des paquets réseau.

#### Gestion des périphériques

Le noyau communique avec les périphériques grâce aux pilotes (drivers).

Il peut ainsi gérer les disques, les cartes réseau, les périphériques USB et de nombreux autres composants.

---

## 4. User Space et Kernel Space

### 4.1. Comprendre la séparation

Linux sépare l'exécution des programmes en deux espaces principaux :

- **User Space :** espace dans lequel s'exécutent les applications et les programmes utilisateur.
- **Kernel Space :** espace privilégié dans lequel s'exécute le noyau.

Cette séparation permet notamment d'améliorer la sécurité et la stabilité du système.

Un programme utilisateur ne peut pas accéder librement aux ressources internes du noyau.

Lorsqu'une application a besoin d'un service système, elle doit passer par une interface prévue à cet effet.

### 4.2. User Space

L'espace utilisateur contient notamment :

- Les applications Java.
- Les commandes Bash.
- Les outils Git.
- Les serveurs PostgreSQL et Nginx.
- Les programmes Docker.

Chaque processus dispose de son propre espace mémoire virtuel.

Une application ne peut pas librement lire ou modifier la mémoire d'un autre processus.

### 4.3. Kernel Space

L'espace noyau contient le code exécuté avec les privilèges du noyau.

Il permet notamment :

- L'accès aux périphériques.
- La gestion des processus.
- La gestion de la mémoire.
- La gestion des fichiers.
- La gestion des communications réseau.

### 4.4. Exemple concret

Imaginons qu'une application Java souhaite lire un fichier.

Le déroulement simplifié est le suivant :

1. L'application Java demande l'ouverture du fichier.
2. Une bibliothèque système permet d'effectuer l'appel système correspondant.
3. Le noyau vérifie les permissions et traite la demande.
4. Les données sont récupérées.
5. Le résultat est retourné à l'application.

L'application n'accède donc pas directement au disque : elle utilise les mécanismes fournis par le système.

---

## 5. Les appels système (System Calls)

### 5.1. Définition

Un appel système est une interface permettant à un programme utilisateur de demander un service au noyau Linux.

Il constitue le mécanisme de communication entre User Space et Kernel Space.

Les appels système sont utilisés pour de nombreuses opérations : lecture de fichiers, création de processus, communication réseau ou gestion de la mémoire.

### 5.2. Les principaux appels système

| Appel système | Rôle |
|---|---|
| `openat()` | Ouvrir un fichier ou un répertoire. |
| `read()` | Lire des données. |
| `write()` | Écrire des données. |
| `close()` | Fermer un descripteur de fichier. |
| `fork()` | Créer un nouveau processus. |
| `execve()` | Exécuter un programme. |
| `socket()` | Créer un socket réseau. |
| `connect()` | Établir une connexion réseau. |

### 5.3. Différence entre commande et appel système

Il ne faut pas confondre une commande Linux avec un appel système.

Exemple :

```bash
cat fichier.txt
```

`cat` est un programme utilisateur.

Pour lire et afficher le fichier, il utilise notamment des appels système comme :

- `openat()` pour ouvrir le fichier.
- `read()` pour lire les données.
- `write()` pour afficher le contenu.
- `close()` pour fermer le fichier.

### 5.4. Observer les appels système avec strace

L'outil `strace` permet d'observer les appels système effectués par un programme.

Installation sur Debian/Ubuntu :

```bash
sudo apt install strace
```

Observer les appels système d'une commande :

```bash
strace cat /etc/hostname
```

Filtrer les appels liés aux fichiers :

```bash
strace -e trace=%file cat /etc/hostname
```

Filtrer les appels liés au réseau :

```bash
strace -e trace=%network curl https://example.com
```

**Cas d'utilisation :**

`strace` est particulièrement utile pour diagnostiquer un programme qui n'arrive pas à accéder à un fichier, à ouvrir un socket ou à utiliser une ressource système.

Attention : certains appels peuvent être masqués ou indisponibles selon les permissions et l'environnement d'exécution.

---

## 6. Le Shell

### 6.1. Définition

Le Shell est un interpréteur de commandes permettant à l'utilisateur d'interagir avec le système.

Il reçoit les commandes, les analyse et déclenche leur exécution.

Il existe plusieurs Shells :

- Bash.
- Zsh.
- Dash.
- Fish.
- Ksh.

Bash est particulièrement répandu dans les distributions Linux.

### 6.2. Fonctionnement

Lorsqu'un utilisateur exécute :

```bash
ls -l /var/log
```

Le déroulement simplifié est le suivant :

1. Le Shell analyse la commande et ses arguments.
2. Il recherche le programme `ls` dans les répertoires définis dans la variable `PATH`.
3. Il lance le programme.
4. Le programme utilise les interfaces système nécessaires pour accéder aux informations du répertoire.
5. Le résultat est affiché dans le terminal.

### 6.3. Identifier le Shell utilisé

Afficher le Shell configuré pour l'utilisateur :

```bash
echo "$SHELL"
```

Afficher le processus Shell courant :

```bash
ps -p $$ -o pid,comm,args
```

Attention : la variable `$SHELL` indique généralement le Shell de connexion configuré pour l'utilisateur. Elle ne garantit pas qu'il s'agit du processus Shell actuellement exécuté.

---

## 7. Organisation du système de fichiers Linux

### 7.1. Arborescence générale

Linux organise les fichiers selon une arborescence unique dont le point de départ est le répertoire racine `/`.

Contrairement à Windows, les différents périphériques et partitions sont généralement intégrés dans cette arborescence grâce à des points de montage.

### 7.2. Les principaux répertoires

| Répertoire | Rôle |
|---|---|
| `/` | Racine de toute l'arborescence. |
| `/bin` | Commandes essentielles, souvent un lien vers `/usr/bin`. |
| `/sbin` | Outils système, souvent un lien vers `/usr/sbin`. |
| `/etc` | Fichiers de configuration du système et des services. |
| `/home` | Répertoires personnels des utilisateurs. |
| `/root` | Répertoire personnel de l'administrateur root. |
| `/var` | Données variables : journaux, caches, files d'attente, etc. |
| `/usr` | Applications, bibliothèques et ressources système. |
| `/opt` | Applications additionnelles. |
| `/tmp` | Fichiers temporaires. |
| `/boot` | Fichiers nécessaires au démarrage du système. |
| `/dev` | Fichiers représentant les périphériques. |
| `/proc` | Informations virtuelles sur les processus et le noyau. |
| `/sys` | Interfaces et informations liées au noyau et aux périphériques. |
| `/run` | Données d'exécution temporaires depuis le démarrage. |

### 7.3. Quelques répertoires importants en détail

#### /etc

Contient les fichiers de configuration de nombreux composants du système.

Exemples :

```text
/etc/hostname
/etc/hosts
/etc/passwd
/etc/fstab
/etc/ssh/
/etc/systemd/
```

#### /var

Contient les données qui évoluent pendant l'exécution du système.

Exemples :

```text
/var/log
/var/cache
/var/lib
/var/tmp
```

Le répertoire `/var/log` contient de nombreux journaux système et applicatifs.

Certains journaux peuvent également être gérés directement par `systemd-journald`.

#### /proc

`/proc` est un système de fichiers virtuel qui expose des informations sur le noyau et les processus.

Exemples :

```bash
cat /proc/cpuinfo
cat /proc/meminfo
```

On peut également consulter les informations d'un processus :

```bash
ls /proc/1
```

#### /sys

`/sys` est un système de fichiers virtuel permettant d'exposer des informations et certaines interfaces liées aux périphériques et au noyau.

#### /dev

Contient les fichiers spéciaux représentant les périphériques.

Exemples :

```text
/dev/null
/dev/zero
/dev/random
/dev/sda
```

Les noms et périphériques disponibles dépendent de la machine.

### 7.4. Explorer l'arborescence

Quelques commandes utiles :

```bash
ls /
ls /etc
ls /var/log
ls /proc
ls /sys
```

Pour identifier le type d'un système de fichiers :

```bash
df -Th
```

---

## 8. Le processus de démarrage de Linux

Le démarrage de Linux correspond à une succession d'étapes permettant d'initialiser le matériel, de charger le noyau et de démarrer les services nécessaires.

Le déroulement exact dépend de la distribution, du firmware et de la configuration de la machine.

### 8.1. Vue d'ensemble

```text
+-------------------------+
|       BIOS / UEFI       |
+-------------------------+
             |
             v
+-------------------------+
|      Bootloader         |
|         GRUB            |
+-------------------------+
             |
             v
+-------------------------+
|      Kernel Linux       |
+-------------------------+
             |
             v
+-------------------------+
|        Initramfs        |
+-------------------------+
             |
             v
+-------------------------+
|     Montage de /        |
+-------------------------+
             |
             v
+-------------------------+
|       systemd           |
|         PID 1           |
+-------------------------+
             |
             v
+-------------------------+
|   Services et targets   |
+-------------------------+
             |
             v
+-------------------------+
|   Système opérationnel  |
+-------------------------+
```

### 8.2. BIOS / UEFI

BIOS et UEFI sont des firmwares responsables de l'initialisation de la machine.

Au démarrage, ils effectuent notamment :

- L'initialisation du matériel.
- Des vérifications de base.
- La recherche d'un périphérique de démarrage.
- Le lancement du chargeur de démarrage.

UEFI est le successeur moderne du BIOS traditionnel.

### 8.3. Bootloader : GRUB

GRUB (GRand Unified Bootloader) est un chargeur de démarrage très utilisé dans les distributions Linux.

Ses principales responsabilités :

- Charger le noyau Linux en mémoire.
- Charger l'image initramfs.
- Permettre de sélectionner une version du noyau.
- Transmettre des paramètres au noyau.

Pour consulter les fichiers présents dans le répertoire de démarrage :

```bash
ls /boot
```

On peut notamment trouver :

```text
vmlinuz-...
initrd.img-...
```

- `vmlinuz` : image du noyau Linux.
- `initrd.img` : image de l'initramfs.

### 8.4. Initialisation du Kernel

Après son chargement, le noyau initialise les composants fondamentaux du système :

- Gestion de la mémoire.
- Gestion des processeurs.
- Initialisation des pilotes nécessaires.
- Initialisation des sous-systèmes réseau et stockage.
- Préparation du lancement du premier processus utilisateur.

### 8.5. Initramfs

L'initramfs (*initial RAM filesystem*) est un environnement temporaire chargé en mémoire pendant le démarrage.

Il permet notamment de préparer l'accès au véritable système de fichiers racine `/`.

Pourquoi est-il nécessaire ?

Imaginons que le système soit installé sur un disque utilisant un contrôleur ou un système de fichiers nécessitant des pilotes spécifiques.

Le noyau doit pouvoir accéder au disque avant de monter le véritable système de fichiers racine.

L'initramfs permet notamment de :

- Charger les pilotes nécessaires.
- Initialiser certains périphériques de stockage.
- Déverrouiller éventuellement des volumes chiffrés.
- Préparer le montage du système de fichiers racine.

Une fois cette étape terminée, le démarrage se poursuit avec le véritable système de fichiers racine.

### 8.6. Montage du système de fichiers racine

Une fois le système de stockage accessible, le système monte le véritable système de fichiers racine `/`.

Les répertoires et fichiers nécessaires au fonctionnement du système deviennent alors accessibles.

Le démarrage peut se poursuivre vers l'initialisation de l'environnement utilisateur.

### 8.7. systemd : premier processus utilisateur

Sur la majorité des distributions Linux modernes utilisant systemd, celui-ci est lancé comme premier processus utilisateur.

Il possède le PID 1.

Un PID (*Process Identifier*) est un identifiant attribué à un processus.

Pour vérifier :

```bash
ps -p 1 -o pid,comm,args
```

Systemd est responsable notamment de :

- Démarrer les services.
- Gérer les dépendances entre services.
- Superviser les processus.
- Gérer les points de montage.
- Gérer certains événements système.
- Participer à la gestion des journaux via systemd-journald.

Pour consulter l'état général du système :

```bash
systemctl status
```

Pour afficher les services :

```bash
systemctl list-units --type=service
```

**À retenir :** systemd n'est pas le noyau Linux. C'est un système d'initialisation et de gestion des services qui s'exécute en espace utilisateur.

---

## 9. Les targets systemd

### 9.1. Définition

Systemd utilise des targets pour regrouper des unités et représenter des états ou objectifs de fonctionnement du système.

Une target peut regrouper des services et d'autres unités qui doivent être activés pour atteindre un état donné.

### 9.2. Les principales targets

| Target | Rôle |
|---|---|
| `poweroff.target` | Arrêt du système. |
| `rescue.target` | Mode de secours. |
| `multi-user.target` | Mode multi-utilisateur sans interface graphique. |
| `graphical.target` | Mode graphique. |
| `reboot.target` | Redémarrage du système. |

### 9.3. Identifier la target par défaut

```bash
systemctl get-default
```

### 9.4. Lister les targets actives

```bash
systemctl list-units --type=target
```

### 9.5. Modifier la target par défaut

Exemple :

```bash
sudo systemctl set-default multi-user.target
```

Cette commande modifie la cible utilisée par défaut lors des prochains démarrages.

Attention : sur une machine disposant d'une interface graphique, cette modification peut empêcher son lancement automatique.

Pour revenir à une cible graphique :

```bash
sudo systemctl set-default graphical.target
```

Ne modifie pas la cible par défaut sur une machine de production sans comprendre les conséquences.

---

## 10. Diagnostic et commandes utiles

Cette section rassemble quelques commandes permettant d'obtenir des informations sur l'architecture et le fonctionnement du système.

| Commande | Utilité |
|---|---|
| `uname -a` | Informations sur le noyau. |
| `uname -r` | Version du noyau. |
| `cat /etc/os-release` | Distribution et version. |
| `ps -p 1 -o pid,comm,args` | Identifier le processus PID 1. |
| `systemctl status` | État général de systemd. |
| `systemctl get-default` | Target par défaut. |
| `ls /boot` | Fichiers de démarrage. |
| `df -Th` | Systèmes de fichiers montés. |
| `cat /proc/meminfo` | Informations mémoire. |
| `cat /proc/cpuinfo` | Informations processeur. |

---

## 11. Liens avec le développement et le DevOps

Comprendre l'architecture Linux est particulièrement utile dans un environnement de développement et d'exploitation.

### 11.1. Exécution d'une application Java

Lorsqu'une application Spring Boot est lancée :

```bash
java -jar application.jar
```

Elle s'exécute en espace utilisateur sous forme de processus.

Elle utilise les services du noyau pour :

- Accéder aux fichiers.
- Allouer de la mémoire.
- Créer des threads.
- Établir des connexions réseau.
- Écrire dans les journaux.

### 11.2. Exécution avec systemd

Une application Spring Boot peut être configurée comme service systemd.

Exemple de fichier :

```ini
[Unit]
Description=Spring Boot API
After=network.target

[Service]
User=app
WorkingDirectory=/opt/my-api
ExecStart=/usr/bin/java -jar /opt/my-api/application.jar
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Systemd démarre et supervise le processus Java.

L'application reste un processus utilisateur : elle ne s'exécute pas directement dans le Kernel Space.

### 11.3. Lien avec Docker et Kubernetes

Docker et Kubernetes s'appuient sur plusieurs mécanismes du noyau Linux.

Parmi les plus importants :

- **Namespaces :** isolation de ressources telles que les processus, le réseau et les points de montage.
- **Cgroups :** contrôle et limitation de l'utilisation des ressources.
- **Capabilities :** gestion plus fine des privilèges accordés aux processus.
- **Netfilter :** mécanismes utilisés notamment pour le filtrage réseau.

Ces mécanismes permettent notamment d'isoler les conteneurs et de contrôler leurs ressources.

**À retenir :** un conteneur Linux n'est pas une machine virtuelle classique. Il partage le noyau de la machine hôte, tout en bénéficiant de mécanismes d'isolation.

---

## 12. Bonnes pratiques

- Comprendre la différence entre noyau, Shell et applications.
- Ne pas confondre User Space et Kernel Space.
- Identifier le noyau et la distribution avant un diagnostic.
- Comprendre le rôle de systemd et de PID 1.
- Savoir consulter les informations exposées dans `/proc` et `/sys`.
- Éviter de modifier les fichiers de démarrage sans comprendre leurs conséquences.
- Ne pas modifier les targets systemd par défaut sur un environnement de production sans validation.
- Utiliser les outils de diagnostic avant de modifier la configuration du système.

---

## 13. Exercices pratiques

### Exercice 1 : identifier son système

Exécuter :

```bash
cat /etc/os-release
uname -a
uname -r
```

Objectifs :

- Identifier la distribution.
- Identifier la version du noyau.
- Comprendre la différence entre distribution et noyau.

### Exercice 2 : explorer le système de fichiers

Exécuter :

```bash
ls /
ls /etc
ls /var/log
ls /proc
ls /sys
```

Objectifs :

- Identifier les principaux répertoires.
- Comprendre la différence entre les répertoires classiques et les systèmes de fichiers virtuels.

### Exercice 3 : comprendre le processus PID 1

Exécuter :

```bash
ps -p 1 -o pid,comm,args
```

Puis :

```bash
systemctl status
```

Objectifs :

- Identifier le premier processus utilisateur.
- Comprendre le rôle de systemd.

### Exercice 4 : observer les appels système

Installer `strace`, puis exécuter :

```bash
strace cat /etc/hostname
```

Objectifs :

- Identifier les appels système liés à la lecture d'un fichier.
- Comprendre comment une application communique avec le noyau.

### Exercice 5 : comprendre les targets

Exécuter :

```bash
systemctl get-default
systemctl list-units --type=target
```

Objectifs :

- Identifier la target par défaut.
- Comprendre le rôle des targets systemd.

---

## 14. Résumé

| Concept | À retenir |
|---|---|
| Kernel | Cœur du système Linux. |
| Distribution | Ensemble comprenant le noyau et les outils nécessaires à un système complet. |
| User Space | Espace d'exécution des applications utilisateur. |
| Kernel Space | Espace privilégié d'exécution du noyau. |
| System Calls | Interfaces permettant aux applications de demander des services au noyau. |
| Shell | Interpréteur de commandes. |
| `/etc` | Configuration système. |
| `/var` | Données variables et journaux. |
| `/proc` | Informations virtuelles sur les processus et le noyau. |
| GRUB | Chargeur de démarrage. |
| Initramfs | Environnement temporaire préparant l'accès au système de fichiers racine. |
| systemd | Système d'initialisation et gestionnaire de services. |
| PID 1 | Premier processus utilisateur. |
| Targets | États ou ensembles d'unités systemd. |

---

## 15. Ressources complémentaires

- [Linux Kernel Documentation](https://docs.kernel.org/)
- [The Linux man-pages project](https://man7.org/linux/man-pages/)
- [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)
- [systemd Documentation](https://systemd.io/)
- [GNU GRUB Manual](https://www.gnu.org/software/grub/manual/)

---

**Conclusion**

Comprendre l'architecture Linux permet de mieux appréhender le fonctionnement du système, l'exécution des applications et les mécanismes d'administration.

Ces notions constituent une base importante pour aborder ensuite la gestion des paquets, les disques, SSH, les scripts Bash et le diagnostic système.