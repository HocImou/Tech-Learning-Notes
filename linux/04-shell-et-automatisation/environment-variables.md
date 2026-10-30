# Linux — Variables d'environnement

## 1. Qu'est-ce qu'une variable d'environnement ?

Une variable d'environnement est une valeur nommée mise à disposition d'un processus lors de son exécution.

Elle permet de configurer le comportement des applications sans avoir à modifier leur code source.

Exemples d'utilisation :

- Définir le répertoire personnel d'un utilisateur.
- Indiquer où se trouvent les exécutables.
- Configurer le JDK utilisé par Java.
- Définir le profil actif d'une application Spring Boot.
- Configurer l'accès à un cluster Kubernetes.
- Définir les paramètres de connexion à une base de données.

Les variables d'environnement sont particulièrement importantes dans les environnements de développement, de production et DevOps.

**À retenir :** une variable d'environnement permet de transmettre une configuration à un processus et à ses processus enfants.

## 2. Les variables d'environnement les plus importantes

### 2.1 Variables système et utilisateur

Certaines variables sont couramment utilisées par les systèmes Linux et les applications.

| Variable | Rôle |
|---|---|
| `USER` | Nom de l'utilisateur courant |
| `HOME` | Répertoire personnel |
| `PWD` | Répertoire de travail courant du shell |
| `PATH` | Liste des répertoires dans lesquels rechercher les exécutables |
| `SHELL` | Shell associé à l'utilisateur |
| `LANG` | Paramètres régionaux par défaut |
| `TERM` | Type de terminal |
| `HOSTNAME` | Nom de la machine, lorsqu'il est défini dans l'environnement |

Attention : toutes ces variables ne sont pas nécessairement définies dans chaque processus ou chaque environnement.

### 2.2 Variables utilisées dans le développement

Certains outils et frameworks utilisent des variables d'environnement pour récupérer leur configuration.

| Variable | Usage |
|---|---|
| `JAVA_HOME` | Répertoire d'installation du JDK |
| `SPRING_PROFILES_ACTIVE` | Profil Spring Boot actif |
| `NODE_ENV` | Environnement utilisé par certaines applications Node.js |
| `MAVEN_HOME` | Répertoire d'installation de Maven, selon la configuration |
| `KUBECONFIG` | Chemin vers un ou plusieurs fichiers de configuration Kubernetes |
| `EDITOR` | Éditeur de texte par défaut pour certains outils |

Ces variables ne sont pas toutes des variables prédéfinies par Linux. Elles sont définies et exploitées par les outils concernés.

## 3. Lire et afficher les variables d'environnement

Plusieurs commandes permettent de consulter les variables disponibles dans un processus shell.

### Afficher toutes les variables d'environnement

```bash
# Afficher les variables exportées
printenv

# Variante
env
```

### Afficher une variable spécifique

```bash
echo "$HOME"
echo "$PATH"
echo "$USER"
echo "$JAVA_HOME"
```

On peut également utiliser :

```bash
printenv HOME
printenv JAVA_HOME
```

Si une variable n'existe pas, `printenv` ne retourne aucune valeur.

### Variables shell et variables d'environnement

Il faut distinguer deux notions :

- Une variable shell existe dans le shell courant.
- Une variable d'environnement est exportée et transmise aux processus enfants.

Exemple :

```bash
# Créer une variable shell
MY_VAR=bonjour

# Elle est disponible dans le shell courant
echo "$MY_VAR"

# Mais elle n'est pas transmise aux processus enfants
bash -c 'echo "$MY_VAR"'

# Exporter la variable
export MY_VAR

# Elle devient disponible dans les processus enfants
bash -c 'echo "$MY_VAR"'
```

**À retenir :** `export` permet de rendre une variable disponible dans l'environnement des processus enfants.

## 4. Créer une variable d'environnement

### 4.1 Variable temporaire

Une variable peut être créée pour la session courante du shell.

```bash
export MY_VAR=ma_valeur
```

Elle est alors :

- Disponible dans le terminal courant.
- Disponible dans les processus enfants lancés depuis ce shell.
- Supprimée lorsque la session du shell se termine, sauf si elle a été définie dans un fichier de configuration ou par un autre mécanisme.

Vérifier :

```bash
echo "$MY_VAR"
```

Supprimer une variable :

```bash
unset MY_VAR
```

### 4.2 Variable définie pour une seule commande

Il est également possible de définir une variable uniquement pour l'exécution d'une commande.

```bash
MY_VAR=ma_valeur ./mon-script.sh
```

La variable est transmise à cette commande, sans être ajoutée durablement à l'environnement du shell courant.

C'est particulièrement pratique pour lancer une application avec une configuration spécifique.

Exemple :

```bash
SPRING_PROFILES_ACTIVE=dev java -jar application.jar
```

Dans cet exemple, le processus Java reçoit la variable `SPRING_PROFILES_ACTIVE` avec la valeur `dev`.

## 5. Rendre les variables persistantes pour un utilisateur

Pour retrouver automatiquement certaines variables après l'ouverture d'un nouveau terminal ou d'une session, on peut les définir dans les fichiers de configuration du shell.

Sous Bash, les deux fichiers importants sont :

- `~/.bashrc`
- `~/.profile`

### 5.1 Le fichier ~/.bashrc

Le fichier `.bashrc` est généralement exécuté lors de l'ouverture d'un shell Bash interactif non-login.

On y place notamment :

- Les alias.
- Les fonctions shell.
- La personnalisation du prompt.
- Les variables nécessaires au confort du shell.
- Certaines configurations du PATH.

Ouvrir le fichier :

```bash
nano ~/.bashrc
```

Ajouter par exemple :

```bash
# Configuration Java
export JAVA_HOME=/usr/lib/jvm/java-17

# Ajouter les exécutables Java au PATH
export PATH="$PATH:$JAVA_HOME/bin"

# Éditeur par défaut
export EDITOR=nano

# Alias
alias ll='ls -lh'
```

Recharger la configuration :

```bash
source ~/.bashrc
```

Vérifier :

```bash
echo "$JAVA_HOME"
java -version
```

### 5.2 Le fichier ~/.profile

Le fichier `.profile` est généralement exécuté lors de l'ouverture d'une session de connexion (login shell), selon le mécanisme de connexion utilisé.

Il permet notamment d'initialiser l'environnement utilisateur.

Exemple :

```bash
nano ~/.profile
```

Ajouter :

```bash
# Configuration de l'environnement utilisateur
export JAVA_HOME=/usr/lib/jvm/java-17
export PATH="$PATH:$JAVA_HOME/bin"
```

Les changements seront pris en compte lors de la prochaine ouverture d'une session de connexion.

On peut également recharger le fichier manuellement :

```bash
source ~/.profile
```

### Différence entre .bashrc et .profile

| Fichier | Rôle principal |
|---|---|
| `~/.bashrc` | Configuration des shells Bash interactifs non-login |
| `~/.profile` | Initialisation de l'environnement des sessions de connexion |

**Bonne pratique :**

- Placer les alias et fonctions dans `.bashrc`.
- Placer les variables d'environnement nécessaires à la session utilisateur dans `.profile`.
- Éviter de définir plusieurs fois les mêmes variables dans différents fichiers sans raison.

Attention : le comportement exact dépend du shell et de la manière dont la session est ouverte. Bash possède notamment des règles spécifiques pour les shells login et non-login.

## 6. Variables système globales

Jusqu'ici, nous avons travaillé avec des variables propres à un utilisateur.

Il est également possible de définir des variables destinées à être disponibles pour plusieurs utilisateurs.

### 6.1 Le fichier /etc/environment

Le fichier `/etc/environment` permet de définir des variables d'environnement au niveau du système.

Il est généralement pris en charge par les mécanismes de connexion du système, notamment PAM.

Ouvrir le fichier :

```bash
sudo nano /etc/environment
```

Ajouter par exemple :

```text
JAVA_HOME=/usr/lib/jvm/java-17
APP_ENV=production
```

**Important :** `/etc/environment` n'est pas un script shell.

Il contient des paires `clé=valeur`.

Il ne faut donc pas utiliser :

```bash
export JAVA_HOME=/usr/lib/jvm/java-17
```

ni compter sur l'expansion de variables comme :

```text
PATH=$PATH:/opt/mytool/bin
```

La syntaxe et l'interprétation sont différentes de celles d'un script Bash.

Les changements sont généralement pris en compte lors d'une nouvelle session de connexion. Ils ne modifient pas rétroactivement l'environnement des processus déjà démarrés.

### Quand utiliser /etc/environment ?

Ce fichier peut être utile pour :

- Des variables communes à plusieurs utilisateurs.
- Des paramètres d'environnement utilisateur.
- Des configurations simples gérées par le mécanisme de connexion.

Pour des configurations plus complexes, il faut utiliser le mécanisme adapté : scripts de profil, configuration du gestionnaire de session ou configuration des services.

**Attention :** `/etc/environment` n'est pas un mécanisme universel pour configurer les services systemd.

## 7. La variable PATH : une variable essentielle

La variable `PATH` est l'une des variables les plus importantes sous Linux.

Elle contient une liste de répertoires dans lesquels le shell recherche les exécutables lorsqu'on saisit une commande sans préciser son chemin complet.

Afficher son contenu :

```bash
echo "$PATH"
```

Exemple :

```text
/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin
```

Les différents répertoires sont séparés par le caractère `:`.

### 7.1 Comment Linux retrouve une commande ?

Lorsque l'on exécute :

```bash
ls
```

Le shell recherche un exécutable nommé `ls` dans les répertoires présents dans `PATH`, dans l'ordre.

On peut afficher le chemin utilisé :

```bash
which ls
```

Ou utiliser :

```bash
command -v ls
```

### 7.2 Ajouter un répertoire au PATH

Imaginons que l'on installe un outil dans :

```text
/opt/mytool/bin/mytool
```

Sans modifier le PATH, on peut l'exécuter avec son chemin complet :

```bash
/opt/mytool/bin/mytool
```

Pour pouvoir utiliser directement :

```bash
mytool
```

il faut ajouter `/opt/mytool/bin` au PATH.

```bash
export PATH="$PATH:/opt/mytool/bin"
```

On peut vérifier :

```bash
echo "$PATH"
command -v mytool
```

### 7.3 Attention à l'écrasement du PATH

Une erreur fréquente consiste à remplacer entièrement la variable :

```bash
export PATH=/opt/mytool/bin
```

Cette commande supprime les anciens répertoires du PATH pour le shell courant.

Conséquence possible : des commandes habituelles comme `ls`, `grep` ou `java` peuvent ne plus être trouvées.

Il faut généralement ajouter un répertoire à la valeur existante :

```bash
export PATH="$PATH:/opt/mytool/bin"
```

Ou le placer en priorité :

```bash
export PATH="/opt/mytool/bin:$PATH"
```

La position du répertoire peut modifier la commande exécutée lorsqu'il existe plusieurs exécutables portant le même nom.

### 7.4 Bonnes pratiques pour PATH

- Ne jamais remplacer le PATH sans raison.
- Ajouter les nouveaux répertoires à la valeur existante.
- Vérifier le chemin avec `echo "$PATH"`.
- Vérifier l'exécutable utilisé avec `command -v`.
- Éviter d'ajouter des répertoires non fiables au PATH.
- Définir le PATH dans le fichier de configuration adapté au contexte.

Pour une configuration utilisateur persistante, on peut utiliser `.profile` ou `.bashrc`, selon le type de session.

## 8. Cas pratiques pour un développeur Java et DevOps

### 8.1 Configurer JAVA_HOME

`JAVA_HOME` permet d'indiquer le répertoire d'installation du JDK.

Exemple :

```bash
export JAVA_HOME=/usr/lib/jvm/java-17
export PATH="$PATH:$JAVA_HOME/bin"
```

Vérifier :

```bash
echo "$JAVA_HOME"
java -version
javac -version
```

Pour identifier le chemin réel de Java :

```bash
readlink -f "$(command -v java)"
```

Le chemin de l'exécutable Java se trouve généralement dans un sous-répertoire `bin` du JDK ou du JRE.

### 8.2 Configurer un profil Spring Boot

Spring Boot permet de définir le profil actif à l'aide de la variable :

```bash
SPRING_PROFILES_ACTIVE=dev
```

Exemple :

```bash
export SPRING_PROFILES_ACTIVE=dev
java -jar application.jar
```

Spring Boot utilise alors le profil `dev` pour charger la configuration correspondante.

On peut également définir cette variable uniquement pour une exécution :

```bash
SPRING_PROFILES_ACTIVE=prod java -jar application.jar
```

### 8.3 Configurer Kubernetes

La variable `KUBECONFIG` permet d'indiquer le fichier de configuration utilisé par `kubectl`.

Exemple :

```bash
export KUBECONFIG=/home/user/.kube/config
```

Vérifier :

```bash
echo "$KUBECONFIG"
kubectl config get-contexts
kubectl config current-context
```

Si `KUBECONFIG` n'est pas défini, `kubectl` utilise généralement le fichier par défaut :

```text
~/.kube/config
```

### 8.4 Configurer Maven

Selon l'installation, Maven peut utiliser `MAVEN_HOME` ou `M2_HOME` pour référencer son installation.

Exemple :

```bash
export MAVEN_HOME=/opt/maven
export PATH="$PATH:$MAVEN_HOME/bin"
```

Vérifier :

```bash
mvn -version
echo "$MAVEN_HOME"
```

À noter : les distributions Maven modernes n'exigent pas nécessairement que `MAVEN_HOME` soit défini pour fonctionner. Le point essentiel est que la commande `mvn` soit accessible via le PATH.

## 9. Variables d'environnement et services systemd

Un point important dans les environnements serveur : un service Linux ne récupère pas automatiquement les variables définies dans le `.bashrc` ou le `.profile` d'un utilisateur.

Un service systemd possède son propre environnement d'exécution.

### Exemple : service Spring Boot

Imaginons un service :

```text
/etc/systemd/system/my-api.service
```

Configuration :

```ini
[Unit]
Description=My Spring Boot API
After=network.target

[Service]
User=appuser
WorkingDirectory=/opt/my-api
Environment="SPRING_PROFILES_ACTIVE=prod"
Environment="JAVA_HOME=/usr/lib/jvm/java-17"
ExecStart=/usr/bin/java -jar /opt/my-api/application.jar
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Ici :

- `User` définit l'utilisateur qui exécute le service.
- `WorkingDirectory` définit le répertoire de travail.
- `Environment` définit des variables d'environnement.
- `ExecStart` définit la commande de démarrage.

Après une modification du fichier :

```bash
sudo systemctl daemon-reload
sudo systemctl restart my-api
```

Vérifier le service :

```bash
sudo systemctl status my-api
```

Consulter les logs :

```bash
sudo journalctl -u my-api -f
```

### Utiliser un fichier EnvironmentFile

Il est également possible de centraliser les variables dans un fichier.

Créer :

```bash
sudo nano /etc/my-api/my-api.env
```

Contenu :

```text
SPRING_PROFILES_ACTIVE=prod
JAVA_HOME=/usr/lib/jvm/java-17
```

Dans le fichier systemd :

```ini
[Service]
EnvironmentFile=/etc/my-api/my-api.env
ExecStart=/usr/bin/java -jar /opt/my-api/application.jar
```

Cette approche permet de séparer la configuration du service de sa définition systemd.

**Attention :**

- Le fichier doit être accessible au service.
- Il faut protéger les fichiers qui contiennent des informations sensibles.
- Le format `EnvironmentFile` n'est pas un script Bash : ne pas y utiliser `export` ni supposer que les substitutions shell seront interprétées.

## 10. Variables d'environnement et Docker

Docker permet de transmettre des variables d'environnement aux conteneurs.

Exemple :

```bash
docker run \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DB_HOST=database \
  mon-api:latest
```

L'application qui tourne dans le conteneur reçoit ces variables.

Avec Docker Compose :

```yaml
services:
  api:
    image: mon-api:latest
    environment:
      SPRING_PROFILES_ACTIVE: prod
      DB_HOST: database
```

Pour consulter les variables d'un conteneur :

```bash
docker exec -it mon-api printenv
```

**À retenir :** les variables d'environnement du conteneur sont distinctes de celles du shell de la machine hôte. Elles sont transmises explicitement par la configuration du conteneur.

Attention aux secrets : les mots de passe et autres informations sensibles ne doivent pas être intégrés directement dans les fichiers de configuration versionnés. Il faut utiliser un mécanisme de gestion des secrets adapté.

## 11. Variables d'environnement et Kubernetes

Kubernetes permet également de transmettre des variables d'environnement aux conteneurs.

Exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-api
spec:
  containers:
    - name: api
      image: mon-api:latest
      env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: DB_HOST
          value: "database-service"
```

On peut également utiliser des ConfigMaps pour les paramètres non sensibles et des Secrets pour les données sensibles.

Exemple avec une ConfigMap :

```yaml
envFrom:
  - configMapRef:
      name: api-config
```

Pour consulter les variables d'un conteneur :

```bash
kubectl exec -it my-api -- printenv
```

**À retenir :** dans Kubernetes, les variables d'environnement sont souvent définies au niveau du Pod ou du conteneur, via des manifests, des ConfigMaps ou des Secrets.

## 12. Sécurité et bonnes pratiques

Les variables d'environnement sont pratiques, mais elles ne constituent pas un coffre-fort.

Quelques recommandations :

- Ne jamais versionner des mots de passe ou des clés privées dans Git.
- Ne pas afficher les secrets dans les logs.
- Limiter les permissions sur les fichiers de configuration.
- Utiliser des mécanismes de gestion des secrets adaptés.
- Éviter de définir plusieurs fois une même variable dans différents fichiers sans raison.
- Vérifier l'environnement réel du processus en cas de problème.
- Éviter les modifications dangereuses de PATH.

Une variable d'environnement peut être accessible à certains processus ou utilisateurs selon les permissions et le contexte d'exécution. Elle ne doit donc pas être considérée comme un mécanisme de protection des secrets.

## 13. Exercices pratiques

### Exercice 1 — Explorer les variables

```bash
printenv
echo "$HOME"
echo "$USER"
echo "$PATH"
echo "$SHELL"
```

Questions :

- À quoi sert chaque variable ?
- Quelle est la différence entre `env` et `printenv` ?
- Pourquoi le PATH contient-il plusieurs répertoires ?

### Exercice 2 — Créer une variable temporaire

```bash
export MY_VAR=bonjour
echo "$MY_VAR"
bash -c 'echo "$MY_VAR"'
```

Questions :

- Pourquoi la variable est-elle accessible dans le shell enfant ?
- Que se passe-t-il après `unset MY_VAR` ?
- Quelle différence entre une variable shell et une variable exportée ?

### Exercice 3 — Rendre une variable persistante

Ajouter dans `~/.bashrc` :

```bash
export MY_PROJECT=Tech-Learning-Notes
```

Recharger :

```bash
source ~/.bash