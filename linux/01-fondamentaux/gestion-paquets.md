# Gestion des paquets sous Linux

## 1. Introduction

Sous Linux, l'installation et la gestion des logiciels reposent généralement sur un système de gestion de paquets (*Package Management*).

Contrairement à une installation manuelle où il faut télécharger un programme, gérer ses fichiers et installer soi-même ses dépendances, un gestionnaire de paquets automatise une grande partie de ces opérations.

Il permet notamment de :

- Installer des logiciels.
- Désinstaller des logiciels.
- Mettre à jour les logiciels installés.
- Gérer les dépendances.
- Rechercher des logiciels disponibles.
- Consulter les informations d'un paquet.
- Vérifier l'état des paquets installés.

Exemples de logiciels pouvant être installés :

- Java et OpenJDK.
- Maven.
- Git.
- Curl.
- Nginx.
- PostgreSQL.
- Docker (selon le dépôt utilisé).

**Objectif :** comprendre le fonctionnement des gestionnaires de paquets Linux et savoir installer, maintenir et supprimer proprement les logiciels.

---

## 2. Comprendre le fonctionnement d'un gestionnaire de paquets

### 2.1. Qu'est-ce qu'un paquet ?

Un paquet (*Package*) est une archive contenant les éléments nécessaires à l'installation d'un logiciel.

Il peut contenir :

- Les fichiers exécutables.
- Les bibliothèques.
- Les fichiers de configuration.
- Les scripts exécutés pendant l'installation ou la désinstallation.
- Des métadonnées décrivant le logiciel.
- Les informations sur ses dépendances.

Exemples de formats :

| Format | Distribution principalement concernée |
|---|---|
| `.deb` | Debian, Ubuntu |
| `.rpm` | Fedora, RHEL, Rocky Linux, AlmaLinux |
| `.apk` | Alpine Linux |

Ces formats ne sont pas directement interchangeables. Chaque famille de distributions possède ses propres outils de gestion.

### 2.2. Qu'est-ce qu'un gestionnaire de paquets ?

Un gestionnaire de paquets est un outil permettant d'installer, de mettre à jour et de supprimer des logiciels.

Il s'occupe notamment de :

1. Rechercher le paquet dans les dépôts configurés.
2. Télécharger le paquet.
3. Vérifier les informations nécessaires à son installation.
4. Identifier les dépendances.
5. Installer le logiciel et ses dépendances.
6. Enregistrer les informations du paquet dans la base locale du système.

Par exemple, lorsqu'on installe Git, le gestionnaire de paquets vérifie si les bibliothèques nécessaires sont déjà présentes et installe celles qui manquent.

### 2.3. Qu'est-ce qu'une dépendance ?

Une dépendance est un logiciel ou une bibliothèque nécessaire au fonctionnement d'un autre logiciel.

Exemple simplifié :

```text
Application A
    |
    +---- Bibliothèque B
    |
    +---- Bibliothèque C
               |
               +---- Bibliothèque D
```

L'application A dépend de B et C, tandis que C dépend de D.

Le gestionnaire de paquets résout automatiquement cette chaîne de dépendances dans la mesure où les paquets nécessaires sont disponibles et compatibles.

C'est l'un des principaux avantages de la gestion centralisée des logiciels.

### 2.4. Qu'est-ce qu'un dépôt logiciel ?

Un dépôt (*Repository*) est une source contenant des paquets et des métadonnées permettant de les retrouver.

Il peut être :

- Un dépôt officiel de la distribution.
- Un dépôt fourni par un éditeur.
- Un dépôt interne à une entreprise.
- Un dépôt local.

Les gestionnaires de paquets utilisent ces dépôts pour rechercher et télécharger les logiciels.

**Exemple :** Ubuntu propose des dépôts officiels contenant de nombreux logiciels. Il est également possible d'ajouter certains dépôts externes pour installer des versions spécifiques de logiciels.

---

## 3. Les principales familles de gestionnaires de paquets

Les commandes varient selon la distribution Linux.

### 3.1. Debian et Ubuntu : APT et DPKG

Les distributions Debian et Ubuntu utilisent le format `.deb`.

Deux outils principaux sont disponibles :

- `dpkg` : outil de bas niveau permettant de manipuler directement les paquets Debian.
- `apt` : outil de plus haut niveau qui facilite l'installation, la suppression et la résolution des dépendances.

**APT est généralement l'outil à privilégier pour les opérations courantes.**

### 3.2. Fedora, RHEL et distributions compatibles : DNF et RPM

Ces distributions utilisent principalement le format `.rpm`.

Deux outils principaux :

- `rpm` : outil de bas niveau permettant de manipuler directement les paquets RPM.
- `dnf` : gestionnaire de paquets de haut niveau, qui gère notamment les dépendances et les dépôts.

Sur les versions modernes de Fedora et RHEL, DNF est l'outil courant.

### 3.3. Tableau comparatif

| Fonction | Debian / Ubuntu | Fedora / RHEL |
|---|---|---|
| Gestionnaire principal | `apt` | `dnf` |
| Outil bas niveau | `dpkg` | `rpm` |
| Format | `.deb` | `.rpm` |
| Mettre à jour les métadonnées | `apt update` | `dnf makecache` |
| Installer | `apt install` | `dnf install` |
| Supprimer | `apt remove` | `dnf remove` |
| Rechercher | `apt search` | `dnf search` |
| Afficher les paquets installés | `dpkg -l` | `rpm -qa` |

Les concepts sont similaires, même si les commandes et certains comportements diffèrent.

---

## 4. Gestion des paquets avec APT

APT (*Advanced Package Tool*) est le gestionnaire de paquets utilisé sur Debian et Ubuntu.

### 4.1. Mettre à jour les informations des dépôts

```bash
sudo apt update
```

Cette commande actualise les informations locales concernant les paquets disponibles dans les dépôts configurés.

Elle ne met pas à jour les logiciels installés.

Il faut bien distinguer :

- `apt update` : actualise les métadonnées des dépôts.
- `apt upgrade` : met à jour les paquets installés.

C'est une distinction fondamentale.

### 4.2. Mettre à jour les logiciels installés

```bash
sudo apt upgrade
```

Cette commande installe les nouvelles versions disponibles des paquets installés, dans le respect des contraintes de mise à jour définies par APT.

Pour mettre à jour le système avec une résolution plus large des changements de dépendances :

```bash
sudo apt full-upgrade
```

Attention : `full-upgrade` peut supprimer certains paquets si cela est nécessaire pour effectuer la mise à jour.

Il faut toujours examiner les changements proposés avant de confirmer.

### 4.3. Installer un logiciel

Exemple avec Git :

```bash
sudo apt install git
```

Installer plusieurs logiciels en une seule commande :

```bash
sudo apt install git curl wget vim
```

Installer une version précise lorsqu'elle est disponible :

```bash
sudo apt install git=VERSION
```

La version doit correspondre à une version réellement disponible dans les dépôts configurés.

Pour afficher les versions disponibles :

```bash
apt policy git
```

### 4.4. Supprimer un logiciel

Supprimer un paquet :

```bash
sudo apt remove git
```

Cette commande supprime le paquet, mais conserve généralement ses fichiers de configuration système.

Pour supprimer également les fichiers de configuration gérés par le paquet :

```bash
sudo apt purge git
```

Pour supprimer les dépendances devenues inutiles :

```bash
sudo apt autoremove
```

Avant de confirmer une suppression, il faut vérifier les paquets concernés.

### 4.5. Rechercher un logiciel

```bash
apt search nginx
```

Afficher les informations d'un paquet :

```bash
apt show nginx
```

Connaître la version installée et les versions disponibles :

```bash
apt policy nginx
```

### 4.6. Lister les paquets installés

```bash
dpkg -l
```

Afficher uniquement les paquets dont le nom contient un terme :

```bash
dpkg -l | grep java
```

Afficher les fichiers installés par un paquet :

```bash
dpkg -L git
```

Identifier le paquet propriétaire d'un fichier :

```bash
dpkg -S /usr/bin/git
```

### 4.7. Télécharger un paquet sans l'installer

```bash
apt download git
```

Cette commande télécharge le paquet `.deb` dans le répertoire courant.

Elle est utile pour récupérer un paquet sans modifier directement l'installation du système.

---

## 5. Gestion des paquets avec DPKG

`dpkg` est l'outil de bas niveau utilisé pour manipuler les paquets Debian.

Contrairement à APT, il ne gère pas automatiquement la récupération des dépendances depuis les dépôts.

### 5.1. Installer un paquet local

```bash
sudo dpkg -i paquet.deb
```

Si certaines dépendances sont manquantes, l'installation peut rester incomplète.

On peut alors demander à APT de corriger les dépendances :

```bash
sudo apt --fix-broken install
```

### 5.2. Supprimer un paquet

```bash
sudo dpkg -r nom-paquet
```

Supprimer le paquet et ses fichiers de configuration gérés :

```bash
sudo dpkg -P nom-paquet
```

### 5.3. Vérifier l'état des paquets

```bash
dpkg -l
```

Les résultats contiennent des indicateurs d'état.

Quelques exemples :

- `ii` : paquet installé et correctement configuré.
- `rc` : paquet supprimé, mais fichiers de configuration encore présents.

### 5.4. APT ou DPKG : lequel utiliser ?

Dans la majorité des cas, il faut privilégier APT.

| Situation | Outil conseillé |
|---|---|
| Installer un logiciel depuis un dépôt | APT |
| Mettre à jour le système | APT |
| Gérer les dépendances | APT |
| Installer un fichier `.deb` local | DPKG ou APT selon le contexte |
| Examiner les fichiers d'un paquet installé | DPKG |

Pour installer un paquet local avec APT :

```bash
sudo apt install ./paquet.deb
```

Cette méthode est intéressante car APT peut résoudre les dépendances à partir des dépôts configurés.

---

## 6. Gestion des paquets avec DNF

DNF (*Dandified YUM*) est le gestionnaire de paquets utilisé notamment par Fedora et les versions modernes de RHEL.

### 6.1. Actualiser les métadonnées

```bash
sudo dnf makecache
```

Cette commande prépare ou actualise le cache des métadonnées des dépôts.

### 6.2. Mettre à jour les logiciels

```bash
sudo dnf upgrade
```

Pour mettre à jour l'ensemble des paquets disponibles selon les dépôts configurés.

### 6.3. Installer un logiciel

```bash
sudo dnf install git
```

Installer plusieurs logiciels :

```bash
sudo dnf install git curl wget vim
```

### 6.4. Supprimer un logiciel

```bash
sudo dnf remove git
```

### 6.5. Rechercher un logiciel

```bash
dnf search nginx
```

Afficher les informations d'un paquet :

```bash
dnf info nginx
```

### 6.6. Lister les paquets installés

```bash
dnf list installed
```

Afficher les dépôts configurés :

```bash
dnf repolist
```

### 6.7. Examiner les transactions

DNF conserve un historique des transactions effectuées.

Afficher l'historique :

```bash
sudo dnf history
```

Afficher les détails d'une transaction :

```bash
sudo dnf history info ID
```

Certaines transactions peuvent être annulées avec `dnf history undo`, mais cette opération dépend de l'état actuel des dépôts et des paquets. Elle doit être utilisée avec prudence.

---

## 7. Gestion des paquets avec RPM

`rpm` est l'outil de bas niveau permettant de manipuler directement les paquets au format RPM.

### 7.1. Installer un paquet local

```bash
sudo rpm -ivh paquet.rpm
```

### 7.2. Mettre à jour un paquet local

```bash
sudo rpm -Uvh paquet.rpm
```

### 7.3. Supprimer un paquet

```bash
sudo rpm -e nom-paquet
```

### 7.4. Lister les paquets installés

```bash
rpm -qa
```

Rechercher un paquet :

```bash
rpm -qa | grep java
```

### 7.5. Connaître les fichiers installés par un paquet

```bash
rpm -ql nom-paquet
```

Identifier le paquet propriétaire d'un fichier :

```bash
rpm -qf /usr/bin/git
```

### 7.6. RPM ou DNF ?

DNF est généralement préférable pour les opérations courantes.

RPM est utile pour examiner ou manipuler directement les paquets locaux.

Comme avec DPKG, RPM seul ne réalise pas la résolution complète des dépendances à partir des dépôts.

---

## 8. Comprendre les dépôts logiciels

### 8.1. Pourquoi utiliser des dépôts ?

Les dépôts permettent de centraliser la distribution des logiciels.

Ils offrent notamment :

- Une source centralisée de paquets.
- Des informations sur les versions disponibles.
- Des métadonnées sur les dépendances.
- Des mécanismes de vérification de l'intégrité et de l'authenticité des paquets.

Les dépôts officiels d'une distribution constituent généralement le choix à privilégier.

### 8.2. Dépôts sous Debian et Ubuntu

Les sources APT sont généralement définies dans :

```text
/etc/apt/sources.list
/etc/apt/sources.list.d/
```

Afficher les fichiers de configuration :

```bash
ls /etc/apt/sources.list.d/
```

Afficher les sources configurées :

```bash
grep -R "^deb " /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
```

Les versions récentes de Debian et Ubuntu peuvent également utiliser le format deb822 dans des fichiers `.sources`.

Exemple simplifié d'une source :

```text
Types: deb
URIs: https://example.org/repository
Suites: stable
Components: main
```

Les valeurs ci-dessus sont illustratives : il faut utiliser les sources adaptées à la distribution et à sa version.

Après une modification des sources :

```bash
sudo apt update
```

### 8.3. Dépôts sous Fedora et RHEL

Les fichiers de configuration des dépôts sont généralement stockés dans :

```text
/etc/yum.repos.d/
```

Afficher les dépôts :

```bash
dnf repolist
```

Afficher les fichiers :

```bash
ls /etc/yum.repos.d/
```

Un fichier de dépôt contient notamment :

- Le nom du dépôt.
- Son identifiant.
- Son URL.
- Son état d'activation.
- Les paramètres de vérification des signatures.

### 8.4. Ajouter un dépôt externe

Certains éditeurs proposent leurs propres dépôts afin de distribuer leurs logiciels ou des versions spécifiques.

Avant d'ajouter un dépôt externe, il faut vérifier :

- La fiabilité de l'éditeur.
- La compatibilité avec la distribution.
- La provenance des paquets.
- Le mécanisme de vérification des signatures.
- Les conséquences des futures mises à jour.

Éviter d'ajouter des dépôts inconnus ou de désactiver la vérification des signatures pour contourner une erreur.

---

## 9. Les signatures et la sécurité des paquets

### 9.1. Pourquoi vérifier les paquets ?

Un paquet installé sur une machine peut exécuter des scripts et modifier des fichiers système.

Il est donc important de s'assurer que le paquet provient d'une source fiable et qu'il n'a pas été altéré.

Les gestionnaires de paquets utilisent des mécanismes cryptographiques pour vérifier l'authenticité et l'intégrité des paquets et des métadonnées.

### 9.2. APT et les signatures

APT s'appuie notamment sur des signatures cryptographiques des métadonnées des dépôts.

Les clés de confiance permettent de vérifier que les métadonnées proviennent bien d'une source autorisée.

Les configurations modernes utilisent généralement des fichiers de clés dédiés et la directive `Signed-By`.

Il faut éviter :

- D'utiliser des dépôts non fiables.
- De désactiver les contrôles de signature.
- D'utiliser systématiquement des mécanismes de confiance globale pour des dépôts externes.

### 9.3. RPM et les signatures

Les paquets RPM peuvent être signés avec GPG.

Pour vérifier les informations de signature d'un paquet local :

```bash
rpm -K paquet.rpm
```

La vérification dépend de la configuration des clés de confiance et des politiques du système.

**Bonne pratique :** installer les logiciels depuis les dépôts officiels ou depuis les dépôts documentés par leur éditeur.

---

## 10. Installer les outils d'un environnement de développement

La gestion des paquets est particulièrement utile pour préparer une machine de développement.

### 10.1. Installer Java

Sur Debian ou Ubuntu :

```bash
sudo apt update
sudo apt install openjdk-21-jdk
```

Vérifier la version :

```bash
java -version
javac -version
```

Sur Fedora :

```bash
sudo dnf install java-21-openjdk-devel
```

Les versions disponibles dépendent de la distribution et de ses dépôts.

### 10.2. Installer Git

Debian / Ubuntu :

```bash
sudo apt install git
```

Fedora :

```bash
sudo dnf install git
```

Vérifier :

```bash
git --version
```

### 10.3. Installer Maven

Debian / Ubuntu :

```bash
sudo apt install maven
```

Fedora :

```bash
sudo dnf install maven
```

Vérifier :

```bash
mvn -version
```

Attention : les dépôts des distributions ne proposent pas nécessairement la dernière version de Maven. Pour un projet nécessitant une version précise, le Maven Wrapper (`mvnw`) est souvent préférable.

### 10.4. Installer des outils réseau

Exemple sous Debian / Ubuntu :

```bash
sudo apt install curl wget dnsutils iproute2 net-tools
```

Ces paquets fournissent notamment :

- `curl` : effectuer des requêtes HTTP et HTTPS.
- `wget` : télécharger des fichiers.
- `dig` et `nslookup` : diagnostiquer la résolution DNS.
- `ip` : examiner les interfaces et routes réseau.
- `netstat` : outil historique d'inspection réseau.

Pour les nouvelles utilisations, privilégier `ss` à `netstat`.

### 10.5. Installer Docker

Docker Engine est généralement installé depuis le dépôt officiel Docker lorsque l'on souhaite utiliser sa version maintenue par l'éditeur.

Il faut suivre la documentation officielle correspondant à sa distribution, car les paquets et les étapes d'installation varient selon le système.

Documentation :

https://docs.docker.com/engine/install/

Ne pas supposer que le paquet nommé `docker` dans les dépôts de toutes les distributions correspond à Docker Engine officiel.

---

## 11. Gestion des versions et reproductibilité

### 11.1. Pourquoi les versions sont importantes ?

Les dépôts d'une distribution ne proposent pas nécessairement les mêmes versions de logiciels.

Par exemple, deux machines Ubuntu de versions différentes peuvent proposer des versions différentes de Java ou de Maven.

Cela peut entraîner des différences entre les environnements de développement, de recette et de production.

### 11.2. Vérifier les versions disponibles

Avec APT :

```bash
apt policy openjdk-21-jdk
```

Avec DNF :

```bash
dnf info java-21-openjdk-devel
```

### 11.3. Éviter les installations non maîtrisées

Pour garantir la reproductibilité d'un environnement :

- Documenter les versions nécessaires.
- Utiliser des dépôts connus.
- Éviter les installations manuelles non documentées.
- Prévoir une procédure de mise à jour.
- Utiliser des outils de gestion de versions lorsque cela est pertinent.

Dans un projet Java, Maven Wrapper permet de fixer la version de Maven utilisée par le projet.

Dans un environnement conteneurisé, il est également important de maîtriser les versions des images de base.

---

## 12. Bonnes pratiques

### 12.1. Utiliser le gestionnaire adapté

- Debian et Ubuntu : privilégier APT.
- Fedora et RHEL : privilégier DNF.
- Utiliser DPKG et RPM pour les opérations de bas niveau ou l'inspection des paquets.

### 12.2. Mettre à jour les métadonnées avant l'installation

Sur Debian / Ubuntu :

```bash
sudo apt update
sudo apt install git
```

### 12.3. Examiner les opérations proposées

Avant de confirmer une installation, une mise à jour ou une suppression, vérifier les paquets concernés et les dépendances qui seront modifiées.

### 12.4. Ne pas multiplier les sources

Limiter les dépôts externes et privilégier les sources officielles.

### 12.5. Ne pas installer systématiquement avec root

L'utilisation de `sudo` doit être limitée aux opérations qui nécessitent réellement des privilèges administrateur.

### 12.6. Éviter les suppressions non maîtrisées

Ne pas utiliser des commandes de suppression sans comprendre les conséquences.

Par exemple :

```bash
sudo apt autoremove
```

peut supprimer des paquets qui ne sont plus considérés comme nécessaires. Il faut examiner la liste avant validation.

### 12.7. Documenter les installations

Dans un projet ou un environnement d'équipe, documenter :

- Les logiciels nécessaires.
- Les versions attendues.
- Les dépôts utilisés.
- Les commandes d'installation.
- Les éventuelles configurations spécifiques.

---

## 13. Exercices pratiques

### Exercice 1 : identifier le gestionnaire de paquets

Identifier la distribution :

```bash
cat /etc/os-release
```

Identifier les outils disponibles :

```bash
command -v apt
command -v dnf
command -v dpkg
command -v rpm
```

Objectifs :

- Identifier la famille de sa distribution.
- Comprendre quels outils sont disponibles.
- Distinguer gestionnaire de haut niveau et outil de bas niveau.

### Exercice 2 : installer Git

Sur Debian / Ubuntu :

```bash
sudo apt update
sudo apt install git
```

Sur Fedora :

```bash
sudo dnf install git
```

Vérifier :

```bash
git --version
```

Objectifs :

- Installer un paquet.
- Comprendre la résolution des dépendances.
- Vérifier que le logiciel est disponible.

### Exercice 3 : examiner un paquet

Sur Debian / Ubuntu :

```bash
apt show git
apt policy git
dpkg -L git
```

Sur Fedora :

```bash
dnf info git
rpm -ql git
```

Objectifs :

- Consulter les informations d'un paquet.
- Identifier sa version.
- Examiner les fichiers installés.

### Exercice 4 : comprendre les dépendances

Sur Debian / Ubuntu :

```bash
apt depends git
```

Sur Fedora :

```bash
dnf repoquery --requires git
```

Objectifs :

- Identifier les dépendances d'un logiciel.
- Comprendre pourquoi un gestionnaire doit les résoudre.

### Exercice 5 : rechercher un paquet

Sur Debian / Ubuntu :

```bash
apt search curl
```

Sur Fedora :

```bash
dnf search curl
```

Objectifs :

- Rechercher un logiciel.
- Examiner les informations disponibles.
- Identifier les versions proposées.

### Exercice 6 : comprendre les dépôts

Sur Debian / Ubuntu :

```bash
ls /etc/apt/sources.list.d/
apt policy
```

Sur Fedora :

```bash
dnf repolist
ls /etc/yum.repos.d/
```

Objectifs :

- Identifier les dépôts configurés.
- Comprendre d'où proviennent les paquets.

---

## 14. Résumé des commandes

### Debian / Ubuntu

| Besoin | Commande |
|---|---|
| Actualiser les dépôts | `sudo apt update` |
| Mettre à jour les paquets | `sudo apt upgrade` |
| Mise à jour avec changements de dépendances plus larges | `sudo apt full-upgrade` |
| Installer | `sudo apt install paquet` |
| Supprimer | `sudo apt remove paquet` |
| Supprimer avec configuration | `sudo apt purge paquet` |
| Nettoyer les dépendances inutiles | `sudo apt autoremove` |
| Rechercher | `apt search paquet` |
| Informations | `apt show paquet` |
| Versions disponibles | `apt policy paquet` |
| Lister les paquets installés | `dpkg -l` |
| Lister les fichiers d'un paquet | `dpkg -L paquet` |

### Fedora / RHEL

| Besoin | Commande |
|---|---|
| Actualiser le cache | `sudo dnf makecache` |
| Mettre à jour | `sudo dnf upgrade` |
| Installer | `sudo dnf install paquet` |
| Supprimer | `sudo dnf remove paquet` |
| Rechercher | `dnf search paquet` |
| Informations | `dnf info paquet` |
| Lister les paquets installés | `dnf list installed` |
| Lister les dépôts | `dnf repolist` |
| Historique | `sudo dnf history` |
| Lister les fichiers d'un paquet RPM | `rpm -ql paquet` |

---

## 15. Ressources complémentaires

### Documentation officielle

- [Debian Package Management](https://www.debian.org/doc/manuals/debian-reference/ch02.en.html)
- [Ubuntu - Apt](https://ubuntu.com/server/docs/package-management)
- [Fedora - DNF Documentation](https://docs.fedoraproject.org/en-US/quick-docs/dnf/)
- [RPM Documentation](https://rpm.org/documentation.html)
- [Docker Engine Installation](https://docs.docker.com/engine/install/)

---

## Conclusion

Les gestionnaires de paquets constituent un élément fondamental de l'administration Linux.

Ils facilitent l'installation, la mise à jour et la suppression des logiciels tout en assurant la gestion de leurs dépendances.

La maîtrise d'APT, DPKG, DNF et RPM permet de préparer et maintenir efficacement un environnement de développement ou un serveur.

Ces connaissances seront particulièrement utiles pour installer les outils nécessaires à nos futurs travaux pratiques autour de Java, Spring Boot, Docker et Kubernetes.