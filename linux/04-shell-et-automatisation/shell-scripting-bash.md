# Shell scripting et Bash

## 1. Introduction

Un *shell* est un interpréteur de commandes qui permet à l'utilisateur de communiquer avec le système d'exploitation.

Il permet notamment de :

- Exécuter des commandes Linux.
- Manipuler des fichiers et des répertoires.
- Lancer des programmes.
- Administrer des services.
- Automatiser des tâches répétitives.
- Enchaîner plusieurs opérations.

**Bash** (*Bourne Again SHell*) est l'un des shells les plus utilisés sous Linux.

Il permet également d'écrire des scripts pour automatiser des opérations complexes.

### Shell, terminal et Bash : quelle différence ?

- **Terminal** : interface permettant de saisir des commandes.
- **Shell** : programme qui interprète les commandes.
- **Bash** : implémentation particulière d'un shell.

D'autres shells existent, notamment Zsh, Fish et Dash.

## 2. Premier script Bash

Un script Bash est un fichier texte contenant une succession de commandes.

### 2.1 Créer un script

Créer un fichier :

```bash
nano hello.sh
```

Contenu :

```bash
#!/usr/bin/env bash

echo "Bonjour Linux !"
echo "Mon premier script Bash"
```

La première ligne est appelée *shebang* :

```bash
#!/usr/bin/env bash
```

Elle indique au système quel interpréteur utiliser pour exécuter le script.

### 2.2 Rendre le script exécutable

```bash
chmod +x hello.sh
```

Exécuter le script :

```bash
./hello.sh
```

Il est également possible de l'exécuter directement avec Bash :

```bash
bash hello.sh
```

Dans ce second cas, le fichier n'a pas besoin d'avoir le droit d'exécution.

### 2.3 Bonnes pratiques

Il est recommandé de commencer les scripts par :

```bash
#!/usr/bin/env bash

set -Eeuo pipefail
```

Ces options apportent des garanties supplémentaires :

- `-e` : interrompt généralement le script lorsqu'une commande échoue.
- `-u` : signale l'utilisation de variables non définies.
- `-o pipefail` : fait échouer un pipeline si une commande intermédiaire échoue.
- `-E` : permet notamment de propager les pièges `ERR` dans certains contextes de fonctions et sous-shells.

Attention : `set -e` ne constitue pas une gestion universelle des erreurs. Son comportement dépend du contexte, notamment dans les conditions, les boucles et les listes de commandes.

Il faut donc toujours gérer explicitement les erreurs importantes.

## 3. Variables Bash

Les variables permettent de stocker des informations utilisées par le script.

### 3.1 Déclarer une variable

```bash
#!/usr/bin/env bash

APPLICATION="spring-api"
VERSION="1.0.0"

echo "Application : $APPLICATION"
echo "Version : $VERSION"
```

Attention : il ne faut pas mettre d'espace autour du signe `=`.

Correct :

```bash
APPLICATION="spring-api"
```

Incorrect :

```bash
APPLICATION = "spring-api"
```

### 3.2 Utiliser les accolades

Il est recommandé d'utiliser `${VARIABLE}` lorsque cela améliore la lisibilité :

```bash
APPLICATION="spring-api"

echo "Déploiement de ${APPLICATION}"
```

Cela permet également de distinguer clairement le nom d'une variable du texte qui suit :

```bash
ENV="dev"

echo "application-${ENV}-01"
```

### 3.3 Variables en lecture seule

```bash
readonly ENVIRONMENT="dev"
```

Une variable déclarée avec `readonly` ne peut plus être modifiée.

### 3.4 Supprimer une variable

```bash
unset APPLICATION
```

### 3.5 Variables d'environnement

Une variable Bash n'est pas automatiquement transmise aux processus enfants.

Pour l'exporter :

```bash
export APPLICATION="spring-api"
```

Elle devient alors accessible aux programmes lancés depuis le shell.

Exemple :

```bash
export SPRING_PROFILES_ACTIVE=dev
java -jar application.jar
```

Spring Boot peut récupérer cette variable pour activer le profil `dev`.

## 4. Arguments d'un script

Un script peut recevoir des paramètres au moment de son exécution.

Exemple :

```bash
#!/usr/bin/env bash

echo "Nom du script : $0"
echo "Premier argument : $1"
echo "Deuxième argument : $2"
echo "Nombre d'arguments : $#"
echo "Tous les arguments : $@"
```

Exécution :

```bash
./deploy.sh spring-api dev
```

Résultat :

```text
Nom du script : ./deploy.sh
Premier argument : spring-api
Deuxième argument : dev
Nombre d'arguments : 2
Tous les arguments : spring-api dev
```

### Variables spéciales

| Variable | Signification |
|---|---|
| `$0` | Nom du script |
| `$1`, `$2` | Arguments positionnels |
| `$#` | Nombre d'arguments |
| `"$@"` | Tous les arguments, conservés individuellement |
| `"$*"` | Tous les arguments regroupés en une seule chaîne lorsqu'il est entre guillemets |
| `$?` | Code de retour de la dernière commande |
| `$$` | PID du shell courant |
| `$!` | PID du dernier processus lancé en arrière-plan |

**Bonne pratique :** utiliser `"$@"` plutôt que `$@` pour préserver les arguments contenant des espaces.

## 5. Lire une saisie utilisateur

La commande `read` permet de récupérer une valeur saisie par l'utilisateur.

```bash
#!/usr/bin/env bash

read -r -p "Entrez le nom de l'application : " APPLICATION

echo "Application sélectionnée : ${APPLICATION}"
```

Pour masquer la saisie, par exemple lors de la saisie d'un secret :

```bash
read -r -s -p "Mot de passe : " PASSWORD
echo
```

Éviter d'afficher ou de journaliser les mots de passe.

## 6. Conditions : if, elif et else

Les conditions permettent d'exécuter des instructions selon une situation.

### 6.1 Condition if

```bash
#!/usr/bin/env bash

ENVIRONMENT="dev"

if [[ "$ENVIRONMENT" == "dev" ]]; then
    echo "Environnement de développement"
fi
```

### 6.2 if / else

```bash
if [[ "$ENVIRONMENT" == "prod" ]]; then
    echo "Déploiement en production"
else
    echo "Déploiement hors production"
fi
```

### 6.3 if / elif / else

```bash
if [[ "$ENVIRONMENT" == "dev" ]]; then
    echo "Développement"
elif [[ "$ENVIRONMENT" == "preprod" ]]; then
    echo "Préproduction"
elif [[ "$ENVIRONMENT" == "prod" ]]; then
    echo "Production"
else
    echo "Environnement inconnu"
fi
```

### 6.4 Comparaisons

Pour les chaînes de caractères :

| Opérateur | Signification |
|---|---|
| `==` | Égalité |
| `!=` | Différence |
| `-z` | Chaîne vide |
| `-n` | Chaîne non vide |

Pour les nombres :

| Opérateur | Signification |
|---|---|
| `-eq` | Égal |
| `-ne` | Différent |
| `-gt` | Supérieur |
| `-ge` | Supérieur ou égal |
| `-lt` | Inférieur |
| `-le` | Inférieur ou égal |

Exemple :

```bash
PORT=8080

if [[ "$PORT" -gt 1024 ]]; then
    echo "Port supérieur à 1024"
fi
```

### 6.5 Vérifier l'existence d'un fichier

```bash
FILE="/opt/application/application.jar"

if [[ -f "$FILE" ]]; then
    echo "Le fichier existe"
else
    echo "Fichier introuvable"
fi
```

Tests courants :

| Expression | Signification |
|---|---|
| `-e fichier` | Le chemin existe |
| `-f fichier` | Fichier régulier |
| `-d chemin` | Répertoire |
| `-r fichier` | Lisible |
| `-w fichier` | Modifiable |
| `-x fichier` | Exécutable |

## 7. Opérateurs logiques

Les opérateurs logiques permettent de combiner plusieurs conditions.

### ET logique

```bash
if [[ "$ENVIRONMENT" == "prod" && "$PORT" -eq 8080 ]]; then
    echo "Configuration de production valide"
fi
```

Les deux conditions doivent être vraies.

### OU logique

```bash
if [[ "$ENVIRONMENT" == "dev" || "$ENVIRONMENT" == "preprod" ]]; then
    echo "Environnement non productif"
fi
```

Une seule condition vraie suffit.

### Négation

```bash
if [[ ! -f "application.jar" ]]; then
    echo "Application absente"
fi
```

## 8. Boucles

Les boucles permettent de répéter des opérations.

### 8.1 Boucle for

```bash
for ENV in dev preprod prod; do
    echo "Déploiement sur ${ENV}"
done
```

Parcours de fichiers :

```bash
for FILE in /var/log/*.log; do
    [[ -f "$FILE" ]] || continue
    echo "Fichier : $FILE"
done
```

### 8.2 Boucle while

```bash
COUNT=1

while [[ "$COUNT" -le 5 ]]; do
    echo "Tentative ${COUNT}"
    ((COUNT += 1))
done
```

### 8.3 Boucle until

La boucle `until` continue tant que la condition est fausse.

```bash
COUNT=1

until [[ "$COUNT" -gt 5 ]]; do
    echo "Valeur : ${COUNT}"
    ((COUNT += 1))
done
```

### 8.4 break et continue

`break` permet de quitter une boucle.

`continue` permet de passer directement à l'itération suivante.

```bash
for NUMBER in 1 2 3 4 5; do
    if [[ "$NUMBER" -eq 3 ]]; then
        continue
    fi

    echo "$NUMBER"
done
```

## 9. Fonctions Bash

Les fonctions permettent de regrouper des instructions réutilisables.

Exemple :

```bash
#!/usr/bin/env bash

log() {
    echo "[INFO] $1"
}

log "Démarrage du déploiement"
log "Déploiement terminé"
```

### 9.1 Arguments d'une fonction

```bash
deploy() {
    local APPLICATION="$1"
    local ENVIRONMENT="$2"

    echo "Déploiement de ${APPLICATION} sur ${ENVIRONMENT}"
}

deploy "spring-api" "dev"
```

Utiliser `local` permet de limiter la portée d'une variable à la fonction.

### 9.2 Retourner un code de statut

Une fonction Bash retourne principalement un code de statut compris entre 0 et 255.

```bash
check_file() {
    local FILE="$1"

    if [[ -f "$FILE" ]]; then
        return 0
    else
        return 1
    fi
}

if check_file "application.jar"; then
    echo "Fichier disponible"
else
    echo "Fichier absent"
fi
```

Le code `0` indique le succès. Une valeur différente de `0` indique généralement un échec.

Pour transmettre une valeur calculée, on peut utiliser `printf` et la substitution de commande :

```bash
get_application_name() {
    printf '%s\n' "spring-api"
}

APPLICATION="$(get_application_name)"
```

## 10. Codes de retour et gestion des erreurs

Chaque commande Linux retourne un code de statut.

- `0` : succès.
- Valeur différente de `0` : erreur ou situation particulière.

Consulter le code de retour :

```bash
ls /tmp
echo $?
```

Exemple de gestion explicite :

```bash
if ! mkdir -p /opt/application; then
    echo "Impossible de créer le répertoire" >&2
    exit 1
fi
```

### Utiliser exit

La commande `exit` permet de terminer le script avec un code de statut.

```bash
if [[ ! -f "application.jar" ]]; then
    echo "Fichier application.jar introuvable" >&2
    exit 1
fi

echo "Fichier disponible"
exit 0
```

### Gestion centralisée des erreurs

Il est possible d'utiliser un piège `ERR` :

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

trap 'echo "Erreur à la ligne ${LINENO}" >&2' ERR

mkdir -p /tmp/demo
```

Le piège `ERR` est utile pour le diagnostic, mais il ne remplace pas les contrôles explicites avec `if`, ni les vérifications des commandes critiques.

## 11. Redirections et pipes

Bash permet de contrôler les flux d'entrée et de sortie.

Les trois flux standards sont :

| Flux | Descripteur | Description |
|---|---:|---|
| stdin | 0 | Entrée standard |
| stdout | 1 | Sortie standard |
| stderr | 2 | Sortie d'erreur |

### Rediriger la sortie

```bash
echo "Déploiement terminé" > deployment.log
```

Remplace le contenu du fichier.

```bash
echo "Nouvelle opération" >> deployment.log
```

Ajoute du contenu à la fin du fichier.

### Rediriger les erreurs

```bash
commande_inexistante 2> error.log
```

Rediriger stdout et stderr :

```bash
commande > output.log 2>&1
```

Avec Bash, une autre écriture possible est :

```bash
commande &> output.log
```

### Ignorer une sortie

```bash
commande > /dev/null 2>&1
```

### Utiliser un pipe

Un pipe transmet la sortie d'une commande à l'entrée d'une autre.

```bash
ps aux | grep java
```

Exemple :

```bash
df -h | grep /dev
```

Le pipe classique transmet stdout. Pour transmettre également stderr, on peut utiliser :

```bash
commande 2>&1 | grep erreur
```

## 12. Tableaux Bash

Les tableaux permettent de stocker plusieurs valeurs dans une variable.

### Déclarer un tableau

```bash
ENVIRONMENTS=("dev" "preprod" "prod")
```

Accéder à un élément :

```bash
echo "${ENVIRONMENTS[0]}"
```

Afficher tous les éléments :

```bash
echo "${ENVIRONMENTS[@]}"
```

Connaître le nombre d'éléments :

```bash
echo "${#ENVIRONMENTS[@]}"
```

Parcourir le tableau :

```bash
for ENV in "${ENVIRONMENTS[@]}"; do
    echo "Environnement : ${ENV}"
done
```

Ajouter un élément :

```bash
ENVIRONMENTS+=("integration")
```

## 13. Manipulation des chaînes de caractères

Bash propose plusieurs mécanismes pour manipuler les chaînes.

### Connaître la longueur

```bash
APPLICATION="spring-api"

echo "${#APPLICATION}"
```

### Remplacer une partie

```bash
APPLICATION="spring-api"

echo "${APPLICATION/api/service}"
```

Résultat :

```text
spring-service
```

### Extraire une partie

```bash
VERSION="1.2.3"

echo "${VERSION:0:3}"
```

### Valeur par défaut

```bash
ENVIRONMENT="${ENVIRONMENT:-dev}"
```

Si `ENVIRONMENT` n'est pas définie ou est vide, la valeur `dev` sera utilisée.

Différence importante :

```bash
${VARIABLE:-valeur}
```

Utilise la valeur par défaut si la variable est absente ou vide.

```bash
${VARIABLE-valeur}
```

Utilise la valeur par défaut uniquement si la variable est absente.

## 14. Substitution de commande

La substitution de commande permet de récupérer le résultat d'une commande dans une variable.

```bash
CURRENT_DATE="$(date)"
```

Exemple :

```bash
JAVA_VERSION="$(java -version 2>&1 | head -n 1)"

echo "Version Java : ${JAVA_VERSION}"
```

La syntaxe recommandée est `$(...)`, plutôt que l'ancienne syntaxe avec des accents graves.

## 15. Arguments nommés et options

Pour les scripts plus complexes, il est préférable d'utiliser des options explicites.

Exemple :

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

APPLICATION=""
ENVIRONMENT=""

while getopts ":a:e:" OPTION; do
    case "$OPTION" in
        a)
            APPLICATION="$OPTARG"
            ;;
        e)
            ENVIRONMENT="$OPTARG"
            ;;
        \?)
            echo "Option inconnue : -$OPTARG" >&2
            exit 1
            ;;
        :)
            echo "L'option -$OPTARG nécessite une valeur" >&2
            exit 1
            ;;
    esac
done

if [[ -z "$APPLICATION" || -z "$ENVIRONMENT" ]]; then
    echo "Usage : $0 -a application -e environnement" >&2
    exit 1
fi

echo "Application : ${APPLICATION}"
echo "Environnement : ${ENVIRONMENT}"
```

Exécution :

```bash
./deploy.sh -a spring-api -e dev
```

Pour des scripts nécessitant de nombreuses options, on peut aussi utiliser une bibliothèque dédiée ou un parseur d'arguments plus complet.

## 16. Exemple pratique : script de déploiement Spring Boot

Nous allons maintenant combiner les notions précédentes dans un exemple concret.

Objectif :

- Vérifier la présence du fichier JAR.
- Vérifier la disponibilité du serveur distant.
- Copier le JAR.
- Redémarrer le service.
- Afficher le résultat.

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

APPLICATION="spring-api"
JAR_FILE="target/application.jar"
REMOTE_USER="deploy"
REMOTE_HOST="192.168.1.100"
REMOTE_DIR="/opt/${APPLICATION}"
SERVICE_NAME="${APPLICATION}"

log() {
    printf '[INFO] %s\n' "$1"
}

error() {
    printf '[ERROR] %s\n' "$1" >&2
}

if [[ ! -f "$JAR_FILE" ]]; then
    error "Fichier JAR introuvable : ${JAR_FILE}"
    exit 1
fi

if ! ssh -o BatchMode=yes -o ConnectTimeout=5 \
    "${REMOTE_USER}@${REMOTE_HOST}" "true"; then
    error "Connexion SSH impossible"
    exit 1
fi

log "Création du répertoire distant"

if ! ssh "${REMOTE_USER}@${REMOTE_HOST}" \
    "sudo mkdir -p '${REMOTE_DIR}'"; then
    error "Impossible de créer le répertoire distant"
    exit 1
fi

log "Transfert du fichier JAR"

if ! scp "$JAR_FILE" \
    "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/application.jar"; then
    error "Échec du transfert"
    exit 1
fi

log "Redémarrage du service"

if ! ssh "${REMOTE_USER}@${REMOTE_HOST}" \
    "sudo systemctl restart '${SERVICE_NAME}'"; then
    error "Échec du redémarrage"
    exit 1
fi

log "Déploiement terminé"
```

Cet exemple est pédagogique. Dans un environnement réel, il faudra notamment :

- Préparer les droits sudo nécessaires pour l'utilisateur de déploiement.
- Vérifier les empreintes SSH du serveur.
- Gérer les versions et les retours arrière.
- Vérifier la santé de l'application après redémarrage.
- Éviter de remplacer directement un JAR en cours d'utilisation si le mode de déploiement ne le permet pas.

## 17. Bonnes pratiques Bash

- Toujours utiliser un shebang explicite.
- Privilégier `#!/usr/bin/env bash` pour les scripts Bash portables.
- Utiliser `set -Eeuo pipefail` avec une bonne compréhension de ses effets.
- Toujours entourer les expansions de variables avec des guillemets : `"$VARIABLE"`.
- Utiliser `[[ ... ]]` pour les tests Bash.
- Utiliser `local` dans les fonctions.
- Prévoir une gestion explicite des erreurs critiques.
- Écrire des fonctions courtes et spécialisées.
- Utiliser des noms de variables explicites.
- Éviter de stocker des secrets directement dans les scripts.
- Ne jamais exécuter aveuglément une commande construite à partir d'une entrée utilisateur.
- Tester les scripts avant de les utiliser en production.
- Utiliser ShellCheck pour détecter les erreurs et les mauvaises pratiques.

## 18. ShellCheck et débogage

ShellCheck est un outil d'analyse statique pour les scripts shell.

Il détecte notamment :

- Les variables mal utilisées.
- Les problèmes de quoting.
- Les erreurs de syntaxe.
- Les constructions dangereuses.
- Les mauvaises pratiques.

Installation sur Ubuntu :

```bash
sudo apt install shellcheck
```

Analyser un script :

```bash
shellcheck deploy.sh
```

### Mode debug

Pour exécuter un script en mode trace :

```bash
bash -x deploy.sh
```

Pour activer la trace directement dans un script :

```bash
set -x
```

Désactiver la trace :

```bash
set +x
```

Attention : le mode trace peut afficher des informations sensibles. Il faut éviter de l'activer autour de commandes manipulant des secrets.

## 19. Exercices pratiques

### Exercice 1 : premier script

Créer un script qui :

1. Affiche le nom de l'utilisateur courant.
2. Affiche la date.
3. Affiche le répertoire courant.
4. Affiche la version de Bash.

Commandes utiles :

```bash
whoami
date
pwd
bash --version
```

### Exercice 2 : gestion des arguments

Créer un script qui reçoit deux arguments :

- Le nom d'une application.
- Un environnement.

Le script doit afficher un message de déploiement.

Exemple :

```bash
./deploy.sh spring-api dev
```

### Exercice 3 : conditions

Créer un script qui vérifie :

1. Si un fichier JAR existe.
2. Si le fichier est lisible.
3. Si sa taille est supérieure à zéro.

### Exercice 4 : boucles

Créer un script qui parcourt les environnements suivants :

- dev
- preprod
- prod

Pour chaque environnement, afficher un message.

### Exercice 5 : fonctions

Créer une fonction `check_service` qui reçoit le nom d'un service Linux et vérifie s'il est actif.

Commande utile :

```bash
systemctl is-active nginx
```

### Exercice 6 : automatisation

Créer un script qui :

1. Vérifie que Docker est installé.
2. Vérifie que le service Docker est actif.
3. Affiche les conteneurs en cours d'exécution.
4. Affiche un message en cas d'erreur.

### Exercice 7 : script complet

Créer un script de sauvegarde qui :

1. Reçoit un répertoire source.
2. Vérifie qu'il existe.
3. Crée une archive datée.
4. Stocke l'archive dans un répertoire de sauvegarde.
5. Vérifie que l'archive a bien été créée.

## 20. Résumé des commandes et syntaxes essentielles

| Syntaxe / commande | Utilité |
|---|---|
| `#!/usr/bin/env bash` | Définir l'interpréteur |
| `chmod +x script.sh` | Rendre un script exécutable |
| `bash script.sh` | Exécuter un script |
| `export VARIABLE=value` | Exporter une variable |
| `read -r` | Lire une saisie |
| `$1`, `$2` | Arguments positionnels |
| `$#` | Nombre d'arguments |
| `"$@"` | Tous les arguments séparément |
| `if / elif / else` | Conditions |
| `for` | Boucle |
| `while` | Boucle conditionnelle |
| `until` | Boucle jusqu'à condition vraie |
| `break` | Quitter une boucle |
| `continue` | Passer à l'itération suivante |
| `function` ou `nom()` | Déclarer une fonction |
| `return` | Retourner un code de statut |
| `exit` | Terminer le script |
| `case` | Sélectionner selon une valeur |
| `getopts` | Analyser des options |
| `trap` | Intercepter certains événements |
| `set -Eeuo pipefail` | Activer des options de contrôle |
| `bash -x` | Déboguer un script |
| `shellcheck` | Analyser statiquement un script |

## 21. Ressources officielles

- GNU Bash Manual : https://www.gnu.org/software/bash/manual/
- Bash Reference Manual : https://www.gnu.org/software/bash/manual/html_node/
- ShellCheck : https://www.shellcheck.net/
- GNU Coreutils : https://www.gnu.org/software/coreutils/manual/coreutils.html
- Linux man-pages : https://man7.org/linux/man-pages/