# Navigation et manipulation des fichiers sous Linux

Ce document présente les commandes essentielles pour naviguer dans le système de fichiers Linux, créer et modifier des fichiers, consulter les logs et exploiter les flux de commandes.

## 1. Raccourcis clavier du terminal

Les raccourcis dépendent du terminal utilisé et de sa configuration.

| Raccourci | Action |
|---|---|
| `Ctrl + L` | Effacer l'affichage du terminal dans la plupart des shells |
| `Ctrl + C` | Interrompre la commande en cours |
| `Ctrl + D` | Envoyer la fin de l'entrée standard, notamment pour quitter certains shells |
| `Ctrl + R` | Rechercher une commande dans l'historique Bash |
| `Ctrl + Shift + C` | Copier dans de nombreux terminaux graphiques |
| `Ctrl + Shift + V` | Coller dans de nombreux terminaux graphiques |

La commande suivante permet également d'effacer l'affichage du terminal :

```bash
clear
```

**Attention :** effacer l'affichage ne supprime pas les commandes de l'historique et ne supprime aucun fichier.

## 2. Répertoires essentiels

Linux organise les fichiers dans une arborescence qui commence à la racine `/`.

| Répertoire | Rôle | Exemple d'utilisation |
|---|---|---|
| `/` | Racine du système de fichiers | Point de départ de l'arborescence |
| `/home` | Répertoires personnels des utilisateurs | Code source, scripts et documents |
| `/root` | Répertoire personnel du superutilisateur `root` | Fichiers de l'administrateur |
| `/etc` | Configuration du système et des applications | Configuration de services |
| `/var` | Données variables | Logs, files d'attente et données de services |
| `/usr` | Programmes, bibliothèques et ressources installés | Outils système et applications |
| `/bin` | Commandes essentielles, souvent sous `/usr/bin` sur les distributions modernes | `ls`, `cp`, `cat` |
| `/sbin` | Outils d'administration, souvent sous `/usr/sbin` sur les distributions modernes | Outils système |
| `/tmp` | Fichiers temporaires | Fichiers intermédiaires |
| `/opt` | Logiciels additionnels | Applications installées hors des emplacements habituels |
| `/dev` | Fichiers représentant des périphériques | Terminaux et disques |
| `/proc` | Informations virtuelles sur les processus et le noyau | Informations système |
| `/mnt` | Point de montage temporaire ou manuel | Monter un système de fichiers |
| `/media` | Points de montage de supports amovibles sur de nombreuses distributions | Clé USB |

**À retenir :** le contenu de ces répertoires dépend de la distribution et de sa configuration. Les fichiers de `/tmp` peuvent être supprimés automatiquement, mais ce n'est pas nécessairement immédiat.

## 3. Naviguer dans l'arborescence

### Afficher le répertoire courant

```bash
pwd
```

`pwd` signifie *Print Working Directory*. La commande affiche le chemin absolu du répertoire courant.

### Lister les fichiers et répertoires

```bash
ls
ls -l
ls -a
ls -la
ls -lh
```

| Commande | Résultat |
|---|---|
| `ls` | Liste le contenu du répertoire |
| `ls -l` | Affiche les détails : permissions, propriétaire, taille, date |
| `ls -a` | Affiche également les fichiers cachés |
| `ls -la` | Combine l'affichage détaillé et les fichiers cachés |
| `ls -lh` | Affiche les tailles dans un format lisible, par exemple `4K` ou `12M` |

Sous Linux, un nom de fichier commençant par un point, comme `.bashrc`, est généralement considéré comme caché.

### Changer de répertoire

```bash
cd /etc
cd ..
cd ~
cd -
```

- `cd /etc` : accéder à `/etc`.
- `cd ..` : remonter au répertoire parent.
- `cd ~` : accéder au répertoire personnel de l'utilisateur courant.
- `cd -` : revenir au répertoire précédent.

### Chemins absolus et relatifs

Un chemin absolu commence à la racine `/`.

```bash
cd /var/log
```

Un chemin relatif est interprété à partir du répertoire courant.

```bash
cd ..
cd documents
```

Pour comprendre un chemin, les deux symboles suivants sont essentiels :

- `.` représente le répertoire courant.
- `..` représente le répertoire parent.

## 4. Créer et manipuler des fichiers et des répertoires

### Créer des fichiers et des répertoires

```bash
touch file.txt
mkdir test
mkdir -p a/b/c
```

- `touch file.txt` : crée un fichier vide s'il n'existe pas. S'il existe déjà, actualise généralement ses dates d'accès ou de modification.
- `mkdir test` : crée le répertoire `test`.
- `mkdir -p a/b/c` : crée les répertoires parents nécessaires.

### Copier, déplacer et renommer

```bash
cp file.txt file2.txt
cp file.txt test/
mv file2.txt test/
mv ancien-nom.txt nouveau-nom.txt
```

- `cp` copie un fichier.
- `mv` déplace un fichier ou le renomme.
- `test/` désigne le répertoire de destination.

Pour copier un répertoire et son contenu :

```bash
cp -r dossier/ sauvegarde/
```

### Supprimer des fichiers et des répertoires

```bash
rm file.txt
rmdir dossier-vide
rm -r dossier/
```

- `rm` supprime un fichier.
- `rmdir` supprime un répertoire vide.
- `rm -r` supprime récursivement un répertoire et son contenu.

**Attention :** `rm` ne déplace généralement pas les éléments dans une corbeille. Vérifie le chemin avant d'exécuter une suppression, en particulier avec `rm -r` et les chemins contenant des variables.

## 5. Lire le contenu des fichiers

### `cat` : afficher rapidement le contenu

```bash
cat file.txt
```

À utiliser pour les fichiers courts. Évite de l'utiliser pour afficher intégralement un gros fichier de log : la sortie peut être très volumineuse.

### `less` : consulter un fichier de manière interactive

```bash
less file.txt
less -N file.txt
```

`less` permet de parcourir un fichier sans afficher tout son contenu d'un seul coup. L'option `-N` affiche les numéros de ligne.

| Touche | Action |
|---|---|
| `↑` / `↓` | Se déplacer ligne par ligne |
| `PageUp` / `PageDown` | Se déplacer page par page |
| `Space` | Afficher la page suivante |
| `g` | Aller au début |
| `G` | Aller à la fin |
| `/mot` | Rechercher `mot` |
| `n` | Occurrence suivante |
| `N` | Occurrence précédente |
| `q` | Quitter |

Pour consulter une sortie de commande :

```bash
ps aux | less
kubectl logs nom-du-pod | less
```

### `head` : afficher le début d'un fichier

```bash
head file.txt
head -n 20 file.txt
```

Par défaut, `head` affiche les dix premières lignes. `-n 20` demande les vingt premières lignes.

Utile pour vérifier le format d'un fichier CSV ou examiner le début d'un fichier de configuration.

### `tail` : afficher la fin d'un fichier

```bash
tail file.log
tail -n 20 file.log
```

Par défaut, `tail` affiche les dix dernières lignes.

Pour suivre l'arrivée de nouvelles lignes dans un log :

```bash
tail -f app.log
tail -F app.log
```

`-f` suit le fichier ouvert. `-F` permet également de retrouver le fichier lorsqu'il est recréé ou remplacé, par exemple lors d'une rotation des logs.

Pour arrêter le suivi, utilise `Ctrl + C`.

## 6. Rechercher du texte avec `grep`

```bash
grep ERROR app.log
grep -i error app.log
grep -n ERROR app.log
grep -n -i ERROR app.log
```

| Option | Rôle |
|---|---|
| `-i` | Ignore la différence entre majuscules et minuscules |
| `-n` | Affiche le numéro de ligne |
| `-r` | Recherche récursivement dans les fichiers d'un répertoire |
| `-E` | Active les expressions régulières étendues |

Exemple : rechercher les erreurs dans les logs d'une application :

```bash
grep -n ERROR app.log
```

Afficher les dernières lignes d'un log contenant `ERROR` :

```bash
tail -n 500 app.log | grep ERROR
```

Cette dernière commande examine les 500 dernières lignes et filtre celles qui contiennent `ERROR`.

## 7. Les flux Linux

Une commande Linux utilise trois flux standards :

| Numéro | Nom | Signification |
|---|---|---|
| `0` | `stdin` | Entrée standard |
| `1` | `stdout` | Sortie standard |
| `2` | `stderr` | Sortie d'erreur |

Par défaut, dans un terminal interactif :

- le clavier fournit généralement l'entrée standard ;
- la sortie standard est affichée à l'écran ;
- la sortie d'erreur est également affichée à l'écran.

Ces flux peuvent être redirigés vers des fichiers ou transmis à d'autres commandes.

## 8. Les redirections

### Rediriger la sortie standard avec `>` et `>>`

```bash
ls > fichiers.txt
ls >> fichiers.txt
```

- `>` écrit la sortie standard dans un fichier en remplaçant son contenu existant.
- `>>` ajoute la sortie standard à la fin du fichier.

**Attention :** `>` peut vider immédiatement un fichier existant, même si la commande ne produit ensuite aucun résultat utile.

### Lire depuis un fichier avec `<`

```bash
wc -l < file.txt
```

La commande reçoit le contenu de `file.txt` sur son entrée standard. `wc -l` compte les lignes reçues.

### Rediriger les erreurs avec `2>`

```bash
commande 2> erreurs.log
```

La sortie d'erreur est écrite dans `erreurs.log`. La sortie standard reste affichée normalement.

### Séparer la sortie normale et les erreurs

```bash
commande > resultat.log 2> erreurs.log
```

Cette commande permet de conserver les résultats et les erreurs dans deux fichiers différents.

### Réunir la sortie standard et les erreurs

```bash
commande > execution.log 2>&1
```

La sortie standard est redirigée vers `execution.log`, puis la sortie d'erreur est dirigée vers la même destination.

Pour ajouter les deux flux à un fichier existant :

```bash
commande >> execution.log 2>&1
```

## 9. Les pipes : `|`

Un pipe transmet la sortie standard d'une commande à l'entrée standard de la commande suivante.

```bash
commande1 | commande2
```

Exemples :

```bash
ls | grep log
grep ERROR app.log
tail -n 500 app.log | grep ERROR
ps aux | less
```

- `ls | grep log` : filtre la liste des noms affichés par `ls`.
- `grep ERROR app.log` : recherche directement les lignes contenant `ERROR` dans le fichier.
- `tail -n 500 app.log | grep ERROR` : filtre les 500 dernières lignes.
- `ps aux | less` : consulte les processus dans un affichage interactif.

**Bonne pratique :** lorsque `grep` peut lire directement un fichier, préfère `grep ERROR app.log` à `cat app.log | grep ERROR`. Cela évite une commande inutile.

Par défaut, le pipe transmet `stdout`, pas `stderr`. Pour transmettre les deux flux dans Bash :

```bash
commande 2>&1 | less
```

## 10. Modifier un fichier

### Ajouter une ligne

```bash
echo "Nouvelle ligne" >> file.txt
```

Cette commande ajoute une ligne au fichier. Elle est utile pour de petits ajouts automatisés.

### Remplacer le contenu

```bash
echo "test" > file.txt
```

Cette commande remplace le contenu existant du fichier par le texte indiqué.

### Utiliser Nano

```bash
nano file.txt
```

Raccourcis courants dans Nano :

- `Ctrl + O` : enregistrer.
- `Ctrl + X` : quitter.

### Utiliser Vi ou Vim

```bash
vi file.txt
```

Quelques commandes essentielles :

| Touche ou commande | Action |
|---|---|
| `i` | Passer en mode insertion |
| `Esc` | Revenir au mode commande |
| `:w` | Enregistrer |
| `:q` | Quitter |
| `:wq` | Enregistrer et quitter |
| `:q!` | Quitter sans enregistrer les modifications |

## 11. Exercice pratique

Crée un répertoire de travail et manipule quelques fichiers :

```bash
mkdir -p linux-lab/logs
cd linux-lab

touch app.log
echo "INFO Application started" >> app.log
echo "ERROR Database connection failed" >> app.log
echo "INFO Application stopped" >> app.log
```

Exécute ensuite les commandes suivantes :

```bash
pwd
ls -la
cat app.log
grep ERROR app.log
tail -n 2 app.log
```

Pour enregistrer les erreurs filtrées dans un fichier :

```bash
grep ERROR app.log > logs/errors.log
cat logs/errors.log
```

Enfin, consulte le résultat de manière interactive :

```bash
cat app.log | less
```

Tu peux aussi utiliser directement :

```bash
less app.log
```

L'exercice permet de pratiquer la navigation, la création de fichiers, les redirections et le filtrage du contenu.

## 12. À retenir

- `pwd`, `ls` et `cd` sont les commandes fondamentales de navigation.
- `cp`, `mv` et `rm` permettent de copier, déplacer et supprimer des fichiers.
- `less`, `head` et `tail` permettent de consulter des fichiers sans nécessairement tout afficher.
- `grep` recherche du texte dans des fichiers ou dans une sortie.
- `>` remplace le contenu d'un fichier ; `>>` ajoute du contenu.
- `stdout` et `stderr` sont deux flux distincts.
- `|` transmet la sortie standard d'une commande à la suivante.
- Vérifie toujours les chemins et les effets des commandes avant d'exécuter une opération destructive.

## Références officielles

- [GNU Coreutils — documentation des commandes](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [GNU Grep — manuel officiel](https://www.gnu.org/software/grep/manual/grep.html)
- [GNU Bash — manuel officiel](https://www.gnu.org/software/bash/manual/bash.html)
- [Linux man-pages — documentation des commandes et appels système](https://man7.org/linux/man-pages/)