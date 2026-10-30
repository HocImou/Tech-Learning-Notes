# Permissions et utilisateurs sous Linux

Ce document explique comment Linux gère les utilisateurs, les groupes et les permissions d'accès aux fichiers et aux répertoires.

Ces notions sont essentielles pour administrer un serveur, déployer une application Java, gérer des fichiers de configuration ou diagnostiquer des erreurs d'accès dans un environnement Docker.

## 1. Utilisateurs et groupes

### Utilisateur (*user*)

Un utilisateur représente une identité à laquelle Linux associe des droits et des ressources. Il peut s'agir d'une personne ou d'un compte de service utilisé par une application.

Chaque fichier possède un propriétaire.

### Groupe (*group*)

Un groupe permet d'attribuer des droits communs à plusieurs utilisateurs.

Chaque fichier possède un groupe associé. Les permissions classiques distinguent trois catégories :

- **User (`u`)** : le propriétaire du fichier.
- **Group (`g`)** : les membres du groupe associé au fichier.
- **Others (`o`)** : les autres utilisateurs.

Un utilisateur peut appartenir à plusieurs groupes.

## 2. Comprendre les permissions

La commande suivante affiche les permissions, le propriétaire et le groupe des fichiers :

```bash
ls -l
```

Exemple de résultat :

```text
-rw-r--r-- 1 user dev 2456 Oct 28 10:30 fichier.txt
```

Décomposition des permissions :

```text
-rw-r--r--
││  │  │
││  │  └── Others : r--
││  └───── Group  : r--
│└──────── User   : rw-
└───────── Type de fichier
```

Le premier caractère indique le type d'élément. Les neuf caractères suivants correspondent aux droits du propriétaire, du groupe et des autres utilisateurs.

### Types de fichiers courants

| Symbole | Signification |
|---|---|
| `-` | Fichier ordinaire |
| `d` | Répertoire |
| `l` | Lien symbolique |

### Signification de `r`, `w` et `x`

| Droit | Sur un fichier | Sur un répertoire |
|---|---|---|
| `r` — Read | Lire le contenu | Lister les noms des entrées |
| `w` — Write | Modifier le contenu | Créer, supprimer ou renommer des entrées, sous réserve des autres permissions |
| `x` — Execute | Exécuter le fichier, si son type et son contenu le permettent | Traverser le répertoire et accéder aux éléments dont on connaît le chemin |

**Point essentiel :** pour accéder à un fichier via son chemin, il faut généralement disposer du droit `x` sur les répertoires traversés. Pour lister le contenu d'un répertoire, le droit `r` est nécessaire ; pour accéder aux éléments qu'il contient, le droit `x` l'est également.

Le droit `w` sur un répertoire, associé à `x`, permet notamment de créer et supprimer des entrées. Les permissions du fichier lui-même ne déterminent pas toujours si sa suppression est possible : les permissions du répertoire parent jouent un rôle essentiel.

## 3. Modifier les permissions avec `chmod`

La commande `chmod` signifie *change mode*.

### 3.1. Mode symbolique

```bash
# Ajouter la lecture au propriétaire
chmod u+r file.txt

# Ajouter l'écriture au groupe
chmod g+w file.txt

# Retirer la lecture aux autres utilisateurs
chmod o-r file.txt

# Donner le droit d'exécution à toutes les catégories
chmod a+x script.sh

# Retirer l'écriture aux autres utilisateurs
chmod o-w file.txt
```

Les lettres utilisées sont :

- `u` : propriétaire (*user*)
- `g` : groupe (*group*)
- `o` : autres (*others*)
- `a` : toutes les catégories (*all*)
- `+` : ajouter un droit
- `-` : retirer un droit
- `=` : définir précisément les droits de la catégorie concernée

Exemple :

```bash
chmod u=rwx,g=rx,o= file.txt
```

Le propriétaire obtient `rwx`, le groupe `r-x` et les autres n'ont aucun droit.

### 3.2. Mode numérique

Chaque droit possède une valeur :

| Droit | Lettre | Valeur |
|---|---|---:|
| Lecture | `r` | 4 |
| Écriture | `w` | 2 |
| Exécution | `x` | 1 |

Les valeurs sont additionnées pour chaque catégorie.

| Valeur | Permissions |
|---:|---|
| `7` | `rwx` |
| `6` | `rw-` |
| `5` | `r-x` |
| `4` | `r--` |
| `3` | `-wx` |
| `2` | `-w-` |
| `1` | `--x` |
| `0` | `---` |

Exemple :

```bash
chmod 755 script.sh
```

Cela donne :

```text
7 = 4 + 2 + 1 = rwx  # Propriétaire
5 = 4 + 1     = r-x  # Groupe
5 = 4 + 1     = r-x  # Autres
```

Résultat : `rwxr-xr-x`.

Le propriétaire peut lire, modifier et exécuter le fichier. Le groupe et les autres peuvent le lire et l'exécuter, mais pas le modifier via ces permissions.

### 3.3. Permissions courantes

| Permission | Utilisation courante | Interprétation |
|---|---|---|
| `755` | Répertoire accessible ou script exécutable | Propriétaire : tous les droits ; autres : lecture et traversée/exécution |
| `644` | Fichier texte ou configuration non sensible | Propriétaire : lecture/écriture ; autres : lecture |
| `700` | Répertoire ou script privé | Seul le propriétaire dispose de droits |
| `600` | Fichier sensible | Lecture/écriture pour le propriétaire uniquement |
| `775` | Répertoire partagé entre membres d'un groupe | Propriétaire et groupe : tous les droits ; autres : lecture/traversée |
| `777` | À éviter en règle générale | Tous les utilisateurs disposent de tous les droits |

Ces valeurs ne sont pas des obligations universelles. Le choix dépend de la fonction du fichier, de son propriétaire, de son groupe et du contexte de sécurité.

**Attention :** `chmod 777` ne doit pas être utilisé comme solution systématique à une erreur `Permission denied`. Il faut d'abord identifier l'utilisateur effectif du processus et les permissions nécessaires.

### 3.4. Rendre un script exécutable

```bash
chmod u+x script.sh
./script.sh
```

La première commande ajoute le droit d'exécution au propriétaire. La seconde exécute le script depuis le répertoire courant.

Un script peut aussi être lancé explicitement avec un interpréteur, par exemple :

```bash
bash script.sh
```

Dans ce cas, le droit d'exécution du fichier n'est généralement pas nécessaire pour l'utilisateur, mais celui-ci doit pouvoir lire le fichier et accéder aux répertoires du chemin.

### 3.5. Modifier les permissions récursivement

```bash
chmod -R 750 dossier/
```

L'option `-R` applique les permissions récursivement aux éléments contenus dans le répertoire.

**Précaution :** cette commande applique les mêmes permissions numériques aux fichiers et aux répertoires. Or, un fichier ordinaire n'a généralement pas besoin du droit `x`.

Pour un répertoire contenant des fichiers de types différents, il vaut mieux définir séparément les permissions des répertoires et des fichiers, par exemple avec `find` :

```bash
# Répertoires : propriétaire et groupe autorisés
find dossier/ -type d -exec chmod 750 {} +

# Fichiers ordinaires : pas d'exécution par défaut
find dossier/ -type f -exec chmod 640 {} +
```

Ces valeurs sont illustratives : adapte-les à l'usage réel et vérifie les besoins avant de les appliquer.

## 4. Changer le propriétaire et le groupe

### 4.1. `chown` : changer le propriétaire

```bash
# Changer le propriétaire
sudo chown user file.txt

# Changer le propriétaire et le groupe
sudo chown user:dev file.txt

# Changer récursivement le propriétaire et le groupe
sudo chown -R user:dev dossier/
```

`chown` signifie *change owner*. La commande nécessite généralement des privilèges administrateur pour attribuer un fichier à un autre utilisateur.

### 4.2. `chgrp` : changer le groupe

```bash
sudo chgrp dev file.txt
```

Cette commande modifie uniquement le groupe associé au fichier.

### 4.3. Vérifier le résultat

```bash
ls -l file.txt
stat file.txt
```

Ces commandes permettent de contrôler les permissions, le propriétaire et le groupe.

**Attention avec `-R` :** une modification récursive peut affecter de nombreux fichiers. Vérifie toujours le chemin ciblé avant de l'exécuter.

## 5. Consulter les utilisateurs et les groupes

### 5.1. Identifier l'utilisateur courant

```bash
whoami
id
groups
```

- `whoami` affiche le nom de l'utilisateur effectif.
- `id` affiche les identifiants utilisateur et groupe, ainsi que les groupes associés.
- `groups` affiche les groupes auxquels l'utilisateur appartient.

Exemple :

```text
uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),999(docker)
```

Interprétation :

- `uid=1000(john)` : identifiant et nom de l'utilisateur.
- `gid=1000(john)` : groupe principal.
- `groups=...` : groupes supplémentaires.

Les identifiants numériques (`UID` et `GID`) sont importants car le noyau Linux utilise ces identifiants pour appliquer les permissions.

### 5.2. Créer un utilisateur

Sur les distributions qui proposent la commande `adduser` :

```bash
sudo adduser userdev
```

Cette commande crée un utilisateur et propose généralement de définir des informations et un mot de passe.

Sur d'autres distributions, la commande `useradd` peut être utilisée avec des options adaptées à la politique locale.

### 5.3. Créer un groupe

```bash
sudo groupadd dev
getent group dev
```

`groupadd` crée le groupe. `getent group dev` consulte la base des groupes pour vérifier son existence et ses informations.

### 5.4. Ajouter un utilisateur à un groupe

```bash
sudo usermod -aG dev userdev
```

- `-a` signifie *append* : ajouter sans remplacer les groupes supplémentaires existants.
- `-G` indique la liste des groupes supplémentaires.

**Important :** utiliser `-G` sans `-a` peut remplacer la liste des groupes supplémentaires de l'utilisateur. Vérifie toujours les groupes avant et après une modification.

```bash
id userdev
getent group dev
```

La nouvelle appartenance au groupe peut nécessiter une nouvelle session pour être prise en compte dans les processus déjà lancés.

### 5.5. Consulter les informations d'un groupe

```bash
getent group docker
grep '^docker:' /etc/group
```

`getent` est généralement préférable pour consulter les informations de groupe, car il tient compte des sources de comptes configurées sur le système. Une recherche dans `/etc/group` ne couvre que ce fichier local.

## 6. Utiliser `sudo`

`sudo` permet d'exécuter une commande avec des privilèges supplémentaires, selon la configuration des autorisations.

Exemple :

```bash
sudo systemctl status nginx
sudo chown userdev:dev fichier.txt
```

Bonnes pratiques :

- N'utiliser `sudo` que lorsque c'est nécessaire.
- Éviter de lancer une application entière avec des privilèges administrateur sans justification.
- Vérifier le chemin et les effets d'une commande avant de l'exécuter avec `sudo`.
- Ne pas confondre l'appartenance au groupe `sudo` avec les permissions classiques des fichiers.

## 7. Réflexes développeur et DevOps

### 7.1. Diagnostiquer une erreur d'accès

Lorsqu'une application ne peut pas lire ou écrire un fichier, commence par vérifier :

```bash
whoami
id
ls -ld /opt/app
ls -l /opt/app
namei -l /opt/app/config/application.properties
```

`namei -l` permet d'examiner les permissions des différents répertoires traversés dans un chemin.

Vérifie notamment :

1. Quel utilisateur exécute réellement le processus ?
2. Quel est le propriétaire du fichier ?
3. Quel groupe est associé au fichier ?
4. Les permissions permettent-elles l'opération demandée ?
5. Tous les répertoires du chemin autorisent-ils la traversée ?

### 7.2. Exemple pour une application

Imaginons qu'une application doive lire sa configuration et écrire dans un répertoire de logs.

Les permissions doivent être adaptées aux besoins :

- fichier de configuration : lecture pour l'utilisateur de l'application ou son groupe ;
- répertoire de logs : écriture pour l'utilisateur ou le groupe prévu ;
- fichiers contenant des secrets : accès restreint aux identités autorisées.

Il n'est pas nécessaire d'accorder les mêmes droits à toute l'arborescence.

### 7.3. Partager un répertoire entre les membres d'un groupe

Exemple :

```bash
sudo chown -R root:dev /opt/app
sudo chmod 2775 /opt/app
```

Le premier chiffre `2` correspond au bit *setgid* sur un répertoire. Il permet aux nouveaux éléments créés dans ce répertoire d'hériter généralement de son groupe, sous réserve du comportement du système de fichiers et des applications.

La permission `2775` autorise le propriétaire et le groupe à lire, écrire et traverser le répertoire, tandis que les autres disposent de la lecture et de la traversée.

Cette configuration n'est appropriée que si tous les membres du groupe `dev` doivent réellement pouvoir modifier le contenu. Il peut être nécessaire de configurer également l'`umask` ou les ACL pour contrôler les permissions des nouveaux fichiers.

### 7.4. Accès au groupe Docker

```bash
sudo usermod -aG docker "$USER"
```

Cette commande ajoute l'utilisateur courant au groupe `docker`, si ce groupe existe.

**Alerte sécurité :** l'accès au socket du démon Docker permet généralement de lancer des conteneurs avec des privilèges importants sur l'hôte. L'appartenance au groupe `docker` doit donc être accordée uniquement aux utilisateurs de confiance.

Après cette modification, une nouvelle session peut être nécessaire. Pour vérifier :

```bash
id
docker ps
```

Ne modifie pas les permissions du socket Docker avec `chmod 777` pour contourner une erreur d'accès.

## 8. Notions complémentaires

### 8.1. `umask`

L'`umask` définit les permissions que le système retire par défaut lors de la création de fichiers et de répertoires.

```bash
umask
umask -S
```

À titre d'exemple, avec une `umask` de `022`, les permissions habituelles sont :

- fichiers : `644`, à partir de permissions initiales généralement fixées à `666` ;
- répertoires : `755`, à partir de permissions initiales généralement fixées à `777`.

Le résultat exact dépend des permissions demandées par le programme lors de la création.

### 8.2. ACL — Access Control Lists

Les permissions classiques distinguent le propriétaire, le groupe et les autres. Les ACL permettent d'accorder des droits supplémentaires à des utilisateurs ou groupes précis.

Sur les systèmes qui les prennent en charge :

```bash
getfacl fichier.txt
setfacl -m u:userdev:r fichier.txt
```

La deuxième commande ajoute une règle permettant à `userdev` de lire le fichier. Les ACL peuvent introduire un masque de permissions : il faut donc examiner le résultat avec `getfacl`.

Les ACL ne sont pas nécessaires dans tous les cas. Un groupe bien conçu suffit souvent à gérer les accès partagés.

## 9. À retenir

- `chmod` modifie les permissions.
- `chown` modifie le propriétaire et éventuellement le groupe.
- `chgrp` modifie le groupe.
- `id` et `whoami` permettent d'identifier l'utilisateur et ses groupes.
- `usermod -aG` ajoute un utilisateur à un groupe sans remplacer ses groupes supplémentaires existants.
- Le droit `x` sur un répertoire est indispensable pour le traverser.
- `umask` influence les permissions des nouveaux fichiers et répertoires.
- `sudo` et l'accès au groupe Docker doivent être utilisés avec prudence.
- En cas d'erreur d'accès, vérifier l'identité du processus, les permissions du fichier et celles des répertoires parents avant de modifier les droits.

## Références officielles

- [GNU Coreutils — chmod](https://www.gnu.org/software/coreutils/manual/html_node/chmod-invocation.html)
- [GNU Coreutils — chown](https://www.gnu.org/software/coreutils/manual/html_node/chown-invocation.html)
- [Linux man-pages — chmod(2)](https://man7.org/linux/man-pages/man2/chmod.2.html)
- [Linux man-pages — chown(2)](https://man7.org/linux/man-pages/man2/chown.2.html)
- [Documentation Docker — sécurité](https://docs.docker.com/engine/security/)