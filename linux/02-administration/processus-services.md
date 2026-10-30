# Processus et services Linux

## 1. Comprendre les processus Linux

### 1.1. Qu'est-ce qu'un processus ?

Un processus est une instance d'un programme en cours d'exécution. Il possède notamment :

- Un **PID** (*Process ID*) : identifiant unique du processus à un instant donné.
- Un **PPID** (*Parent Process ID*) : identifiant du processus parent.
- Un environnement d'exécution, notamment ses variables d'environnement.
- Des droits et un utilisateur d'exécution.
- Un état : en cours d'exécution, en attente, arrêté, etc.
- Des ressources consommées : CPU, mémoire, fichiers ouverts, etc.

Lorsqu'on ouvre un terminal, un shell comme Bash démarre. Lorsque l'on exécute une commande, le shell peut créer un processus pour l'exécuter.

### 1.2. Exemple : lancer une application Java

```bash
java -jar app.jar
```

Le système d'exploitation lance la JVM, qui exécute l'application Java dans un processus.

Ce processus possède son propre PID et reçoit notamment l'environnement transmis par son processus parent, auquel s'ajoutent les éventuelles configurations de lancement.

Les variables d'environnement disponibles peuvent inclure `PATH`, `JAVA_HOME` ou des variables propres à l'application.

**À retenir :** les variables d'environnement sont transmises aux processus enfants lors de leur création. Modifier une variable dans un terminal ne modifie pas automatiquement l'environnement d'un processus déjà lancé.

---

## 2. Lister et rechercher les processus

### 2.1. Afficher les processus du terminal

```bash
ps
```

Cette commande affiche les processus associés au terminal courant, selon les options et le contexte d'exécution.

### 2.2. Afficher les processus du système

```bash
ps aux
```

Cette commande affiche une vue détaillée des processus, y compris ceux qui ne sont pas associés à un terminal.

Les colonnes les plus utiles sont :

| Colonne | Signification |
|---|---|
| `USER` | Utilisateur propriétaire du processus |
| `PID` | Identifiant du processus |
| `%CPU` | Utilisation du processeur |
| `%MEM` | Part de mémoire physique utilisée |
| `VSZ` | Taille de la mémoire virtuelle |
| `RSS` | Mémoire physique résidente |
| `TTY` | Terminal associé, s'il existe |
| `STAT` | État du processus |
| `COMMAND` | Commande exécutée |

Dans cette commande, `a`, `u` et `x` correspondent à des options de style BSD :

- `a` : élargit l'affichage aux processus associés aux terminaux d'autres utilisateurs ;
- `u` : affiche un format orienté utilisateur, avec notamment CPU et mémoire ;
- `x` : inclut les processus sans terminal associé.

### 2.3. Rechercher un processus Java

Une méthode simple consiste à utiliser `pgrep` :

```bash
pgrep -a java
```

Cette commande affiche les PID et les lignes de commande des processus correspondant à `java`.

Autre possibilité :

```bash
ps aux | grep '[j]ava'
```

La notation `[j]ava` évite généralement que la commande `grep` apparaisse elle-même dans les résultats.

Ces commandes sont utiles pour vérifier si une application Java est lancée. Elles ne permettent pas, à elles seules, de déterminer si l'application fonctionne correctement.

### 2.4. Surveiller les ressources en temps réel

```bash
top
```

`top` affiche une vue actualisée des processus et des ressources système.

Il permet notamment de repérer :

- Un processus qui consomme beaucoup de CPU.
- Une consommation mémoire importante.
- Des processus bloqués ou dans un état inhabituel.
- Une charge système élevée.

Pour diagnostiquer un batch Java lent ou une application qui consomme trop de mémoire, `top` constitue un bon point de départ.

---

## 3. Comprendre les états des processus

La colonne `STAT` de `ps` permet d'identifier l'état d'un processus.

| État | Signification |
|---|---|
| `R` | En cours d'exécution ou prêt à être exécuté |
| `S` | En sommeil interruptible, en attente d'un événement |
| `D` | En sommeil non interruptible, souvent pendant une opération d'entrée-sortie |
| `Z` | Zombie : le processus est terminé, mais son parent n'a pas encore récupéré son statut |
| `T` | Processus arrêté ou suspendu |

Les états peuvent être accompagnés d'autres caractères précisant certaines propriétés.

**Cas particulier : le processus zombie**

Un zombie a déjà terminé son exécution. Il ne consomme normalement plus de CPU et ne peut pas être relancé en lui envoyant simplement un signal d'arrêt.

Le système conserve temporairement son entrée dans la table des processus jusqu'à ce que le parent récupère son statut de fin.

La présence ponctuelle d'un zombie n'est pas nécessairement anormale. En revanche, une accumulation durable peut révéler un problème de gestion des processus enfants par le programme parent.

---

## 4. Comprendre la relation parent-enfant

Un processus peut créer des processus enfants. La commande suivante permet d'examiner la hiérarchie des processus :

```bash
ps -ef
```

Les colonnes importantes sont :

- `PID` : identifiant du processus.
- `PPID` : identifiant de son parent.
- `CMD` : commande exécutée.

Pour afficher une arborescence plus lisible, on peut utiliser :

```bash
pstree -p
```

La disponibilité de `pstree` dépend des outils installés sur le système.

### Arrêter un processus

Avant d'arrêter un processus, identifie précisément celui que tu souhaites arrêter :

```bash
ps -p 1234 -o pid,ppid,user,stat,cmd
```

Remplace `1234` par le PID recherché.

Pour demander un arrêt propre :

```bash
kill 1234
```

Par défaut, `kill` envoie le signal `SIGTERM`. Le processus peut alors effectuer son nettoyage avant de terminer, s'il gère correctement ce signal.

Si le processus ne répond pas et qu'un arrêt forcé est nécessaire :

```bash
kill -9 1234
```

Cette commande envoie `SIGKILL`. Le processus ne peut ni intercepter ni traiter ce signal pour effectuer un nettoyage.

**Attention :** privilégie `SIGTERM` et réserve `SIGKILL` aux situations où l'arrêt normal échoue. Un arrêt forcé peut entraîner la perte de données non enregistrées ou interrompre un traitement en cours.

Tuer un processus parent ne signifie pas automatiquement que tous ses enfants seront tués. Le comportement dépend de la manière dont les processus ont été créés et de la gestion de leur terminaison.

---

## 5. Comprendre les services Linux

### 5.1. Processus et service : quelle différence ?

Un processus représente un programme en cours d'exécution. Un service représente généralement une fonctionnalité système ou applicative gérée par un gestionnaire de services.

| Notion | Processus | Service |
|---|---|---|
| Nature | Programme en cours d'exécution | Unité gérée par un gestionnaire de services |
| Identifiant | PID | Nom d'unité, par exemple `nginx.service` |
| Gestion | Outils comme `ps` et `kill` | Outils comme `systemctl` |
| Cycle de vie | Dépend du programme et de son lancement | Peut être démarré, arrêté, surveillé et configuré |
| Nombre de processus | Un processus identifié par son PID | Peut gérer un ou plusieurs processus |

Exemples de services courants :

- SSH : accès distant.
- Nginx : serveur web ou reverse proxy.
- Docker : gestion du moteur de conteneurs.
- PostgreSQL : serveur de base de données.
- Une application Spring Boot : API ou service métier.

Un service n'est pas nécessairement limité à un unique processus, et tous les processus ne sont pas des services.

### 5.2. Le rôle de systemd

Sur de nombreuses distributions Linux modernes, `systemd` est le gestionnaire de services et de nombreux éléments du système.

Il permet notamment de :

- Démarrer des services au démarrage du système.
- Démarrer, arrêter et redémarrer des services.
- Définir des dépendances entre unités.
- Configurer des variables d'environnement.
- Définir une politique de redémarrage.
- Consulter les journaux associés aux services.

`systemctl` est la commande principale permettant d'interagir avec `systemd`.

---

## 6. Gérer les services avec systemctl

### 6.1. Vérifier l'état d'un service

```bash
systemctl status nginx
```

Cette commande affiche notamment :

- L'état du service.
- Son état d'activation.
- Des informations sur les processus associés.
- Des journaux récents, lorsqu'ils sont disponibles.

Commandes utiles pour vérifier rapidement son état :

```bash
systemctl is-active nginx
systemctl is-enabled nginx
```

La première vérifie si le service est actif. La seconde vérifie s'il est configuré pour être activé automatiquement selon sa configuration systemd.

### 6.2. Démarrer, arrêter et redémarrer

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
```

- `start` : démarre le service.
- `stop` : arrête le service.
- `restart` : redémarre le service.

Certains services prennent en charge `reload`, qui demande de recharger leur configuration sans effectuer un redémarrage complet :

```bash
sudo systemctl reload nginx
```

Cette commande ne fonctionne que si le service prend en charge cette opération.

### 6.3. Activer le démarrage automatique

```bash
sudo systemctl enable nginx
```

Cette commande configure le démarrage automatique du service selon les mécanismes prévus par son unité systemd. Elle ne démarre pas nécessairement le service immédiatement.

Pour activer le démarrage automatique et démarrer le service dans la même opération :

```bash
sudo systemctl enable --now nginx
```

Pour désactiver le démarrage automatique :

```bash
sudo systemctl disable nginx
```

**À retenir :** `enable` concerne le démarrage automatique ; `start` concerne le démarrage immédiat.

---

## 7. Pourquoi un service systemd ne récupère-t-il pas mes variables ?

Un service systemd ne démarre généralement pas depuis ton shell interactif.

Il ne lit donc pas automatiquement les fichiers personnels tels que :

```text
~/.bashrc
~/.profile
```

Les variables disponibles dans un terminal ne sont pas nécessairement présentes dans le processus lancé par systemd.

Si une application Spring Boot dépend d'une variable comme `SPRING_PROFILES_ACTIVE`, il faut prévoir sa transmission au service.

Plusieurs mécanismes existent :

- `Environment=` directement dans le fichier de service.
- `EnvironmentFile=` pour charger des variables depuis un fichier externe.
- Des mécanismes de configuration et de gestion des secrets adaptés à l'environnement de déploiement.

---

## 8. Configurer un service Spring Boot avec systemd

### 8.1. Exemple de fichier `.service`

Supposons que l'application soit installée dans `/opt/my-app` et que son exécution soit assurée par l'utilisateur Linux `appuser`.

Créer le fichier :

```text
/etc/systemd/system/my-app.service
```

Exemple de configuration :

```ini
[Unit]
Description=Application Spring Boot
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/my-app

Environment="SPRING_PROFILES_ACTIVE=prod"
Environment="DB_HOST=localhost"

ExecStart=/usr/bin/java -Xms512m -Xmx1024m -jar /opt/my-app/app.jar

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Explications :

| Directive | Rôle |
|---|---|
| `[Unit]` | Décrit l'unité et ses relations |
| `Description` | Donne un nom lisible au service |
| `After=network.target` | Définit un ordre de démarrage relatif à `network.target` |
| `[Service]` | Configure l'exécution du service |
| `Type=simple` | Indique que le processus lancé constitue le service |
| `User=appuser` | Définit l'utilisateur d'exécution |
| `WorkingDirectory` | Définit le répertoire de travail |
| `Environment` | Définit des variables d'environnement |
| `ExecStart` | Définit la commande de lancement |
| `Restart=on-failure` | Demande un redémarrage en cas d'échec, selon les règles de systemd |
| `RestartSec=5` | Définit un délai avant le redémarrage |
| `[Install]` | Définit les modalités d'activation |
| `WantedBy=multi-user.target` | Permet l'activation habituelle du service au démarrage multi-utilisateur |

Le choix de `Restart=` dépend du comportement attendu. Pour une application de longue durée, `on-failure` est souvent un bon point de départ, mais il faut l'adapter au besoin opérationnel.

**Pourquoi utiliser un chemin absolu dans `ExecStart` ?**

```ini
ExecStart=/usr/bin/java -jar /opt/my-app/app.jar
```

Le chemin absolu évite de dépendre de la résolution du binaire par le `PATH`.

Les options JVM sont indiquées directement dans la commande :

```bash
-Xms512m -Xmx1024m
```

Une variable `JAVA_OPTS` définie dans l'environnement n'est pas automatiquement interprétée par systemd. Si tu écris seulement :

```ini
Environment="JAVA_OPTS=-Xms512m -Xmx1024m"
ExecStart=/usr/bin/java -jar /opt/my-app/app.jar
```

les options ne seront pas appliquées à la JVM. Il faut les transmettre explicitement à la commande ou mettre en place un mécanisme de lancement approprié.

### 8.2. Appliquer les modifications

Après avoir créé ou modifié un fichier d'unité systemd :

```bash
sudo systemctl daemon-reload
sudo systemctl restart my-app
```

- `daemon-reload` demande à systemd de relire les fichiers d'unités.
- `restart` redémarre le service pour appliquer les changements qui nécessitent un nouveau processus.

Il n'est normalement pas nécessaire d'utiliser `daemon-reexec` pour une modification classique d'un fichier `.service`.

Vérifie ensuite le résultat :

```bash
systemctl status my-app
journalctl -u my-app -n 100 --no-pager
```

Si l'unité n'a pas encore été activée au démarrage, tu peux le faire avec :

```bash
sudo systemctl enable my-app
```

---

## 9. Utiliser EnvironmentFile pour externaliser la configuration

### 9.1. Pourquoi externaliser les variables ?

Lorsque la même application doit être déployée en développement, en recette et en production, il est utile de séparer :

- La configuration du service : comment démarrer l'application.
- La configuration de l'environnement : paramètres propres à l'environnement.
- Les secrets : mots de passe, jetons et autres données sensibles.

La directive `EnvironmentFile=` permet à systemd de lire les variables d'environnement dans un fichier externe.

Exemple de structure :

```text
/etc/my-app/
├── my-app.env
└── my-app-secrets.env
```

Le fichier `.service` peut alors contenir :

```ini
[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/my-app

EnvironmentFile=/etc/my-app/my-app.env
EnvironmentFile=/etc/my-app/my-app-secrets.env

ExecStart=/usr/bin/java -Xms512m -Xmx1024m -jar /opt/my-app/app.jar

Restart=on-failure
RestartSec=5
```

Les deux fichiers sont lus par systemd au démarrage du service. Les valeurs des fichiers d'environnement doivent respecter la syntaxe prise en charge par systemd ; ce ne sont pas des scripts Bash.

### 9.2. Exemple de fichier d'environnement

Contenu de `/etc/my-app/my-app.env` :

```text
SPRING_PROFILES_ACTIVE=prod
DB_HOST=db-prod
DB_PORT=5432
```

Les règles pratiques :

- Utiliser des affectations sous la forme `NOM=valeur`.
- Ne pas ajouter `export` comme dans un script shell classique.
- Éviter les espaces autour du signe `=`.
- Respecter la syntaxe d'échappement et de citation documentée par systemd lorsque les valeurs contiennent des caractères particuliers.

Exemple :

```text
# Correct
APP_ENV=production

# Incorrect
APP_ENV = production
```

Pour des variables sensibles, un fichier séparé peut simplifier la gestion des droits, mais cela ne remplace pas une solution dédiée aux secrets.

### 9.3. Plusieurs fichiers d'environnement

On peut déclarer plusieurs fichiers :

```ini
EnvironmentFile=/etc/my-app/base.env
EnvironmentFile=/etc/my-app/prod.env
EnvironmentFile=/etc/my-app/secrets.env
```

Lorsqu'une même variable est définie dans plusieurs fichiers `EnvironmentFile=`, la valeur provenant du dernier fichier concerné prend le dessus.

L'ordre de définition des variables est donc important. En cas de doute, consulte la documentation de la version de systemd utilisée.

### 9.4. Protéger les fichiers sensibles

Les fichiers contenant des secrets ne doivent pas être accessibles à tous les utilisateurs.

Par exemple :

```bash
sudo chown root:root /etc/my-app/my-app-secrets.env
sudo chmod 600 /etc/my-app/my-app-secrets.env
```

Le mode `600` autorise uniquement le propriétaire à lire et modifier le fichier. Le gestionnaire systemd, qui lit le fichier lors du lancement du service, doit pouvoir y accéder.

Il faut également protéger les secrets présents dans les variables d'environnement du processus : selon les permissions du système et le contexte d'exécution, certaines informations peuvent être inspectées par des utilisateurs privilégiés.

**Bonne pratique :** ne jamais versionner des mots de passe ou des secrets de production dans Git. Pour un déploiement plus avancé, privilégier un gestionnaire de secrets adapté à l'infrastructure.

---

## 10. Modifier l'environnement d'un service existant avec un override

Il n'est pas toujours nécessaire de modifier directement le fichier d'unité fourni par un paquet système.

Pour personnaliser le service Docker, par exemple :

```bash
sudo systemctl edit docker
```

Cette commande ouvre un fichier de surcharge (*drop-in*). On peut y ajouter :

```ini
[Service]
Environment="HTTP_PROXY=http://proxy:8080"
```

Puis appliquer les modifications :

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

L'override est fusionné avec la configuration d'origine selon les règles de systemd.

Cette méthode est utile lorsque l'on souhaite conserver les fichiers fournis par les paquets et isoler les personnalisations locales.

---

## 11. Diagnostiquer un problème de variables d'environnement

Prenons le cas d'une application Spring Boot qui ne démarre pas parce qu'une variable est absente.

### Étape 1 — Vérifier l'unité systemd

```bash
systemctl cat my-app
```

Cette commande affiche le contenu de l'unité ainsi que les éventuels fichiers de surcharge.

Vérifie la présence de `Environment=` et `EnvironmentFile=`.

### Étape 2 — Vérifier le fichier de configuration

```bash
sudo ls -l /etc/my-app/
```

Vérifie que les fichiers existent et que leur chemin est correct.

Évite d'afficher inutilement les fichiers contenant des secrets dans un terminal ou dans des journaux partagés.

### Étape 3 — Vérifier les propriétés du service

```bash
systemctl show my-app --property=MainPID
systemctl show my-app --property=Environment
```

`MainPID` permet de récupérer le PID principal du service.

Attention : la propriété `Environment` n'est pas une représentation garantie de l'ensemble des variables réellement présentes dans le processus. Elle ne suffit pas à vérifier toutes les valeurs issues des fichiers externes.

### Étape 4 — Examiner l'environnement du processus

Une fois le PID récupéré, on peut inspecter son environnement sur les systèmes Linux qui exposent cette information via `/proc` :

```bash
sudo cat /proc/1234/environ | tr '\0' '\n'
```

Remplace `1234` par le PID réel.

**Attention :** cette commande peut afficher des secrets. Utilise-la avec prudence, évite de copier sa sortie dans un ticket ou un journal et vérifie les restrictions d'accès applicables sur le système.

### Étape 5 — Consulter les journaux

Afficher les derniers journaux du service :

```bash
journalctl -u my-app -n 100 --no-pager
```

Suivre les journaux en temps réel :

```bash
journalctl -u my-app -f
```

Afficher les informations récentes avec le contexte de diagnostic :

```bash
journalctl -u my-app -xe
```

Les journaux permettent notamment de repérer :

- Une variable obligatoire absente.
- Un fichier de configuration introuvable.
- Une erreur de connexion à la base de données.
- Un chemin incorrect vers Java ou le fichier JAR.
- Une erreur de permission ou un échec au démarrage.

---

## 12. Comparaison : systemd, Docker et Kubernetes

Ces technologies peuvent toutes intervenir dans l'exécution d'une application, mais elles ne jouent pas exactement le même rôle.

| Besoin | systemd | Docker | Kubernetes |
|---|---|---|---|
| Déclarer le lancement | Unité `.service` | Configuration d'exécution, souvent avec Docker Compose | Ressource comme `Deployment` |
| Fournir des variables | `Environment=` | Options d'exécution ou Compose | `env` ou `envFrom` |
| Externaliser une configuration | `EnvironmentFile=` | `--env-file` ou configuration Compose | `ConfigMap` |
| Gérer des secrets | Fichier protégé ou gestionnaire dédié | Mécanisme de secrets adapté à la plateforme | Ressource `Secret` et contrôles d'accès |
| Superviser l'exécution | Gestionnaire systemd | Moteur de conteneurs | Contrôleurs Kubernetes |

### Exemple avec Docker

```bash
docker run --env-file my-app.env my-image
```

Cette commande transmet les variables du fichier au conteneur.

Même si cette approche ressemble à `EnvironmentFile=` de systemd, les formats et les règles de traitement ne sont pas identiques dans tous les cas.

### Exemple avec Kubernetes

Une partie d'un manifeste peut référencer une `ConfigMap` et un `Secret` :

```yaml
envFrom:
  - configMapRef:
      name: my-app-config
  - secretRef:
      name: my-app-secrets
```

Kubernetes injecte alors les variables provenant de ces ressources dans le conteneur concerné.

Une `ConfigMap` sert à la configuration non sensible. Un `Secret` est prévu pour les données sensibles, mais sa seule utilisation ne garantit pas une protection suffisante : les permissions d'accès, le chiffrement et les pratiques opérationnelles restent importants.

**À retenir :** les trois environnements permettent de configurer l'exécution d'une application, mais ils utilisent des mécanismes différents. Il faut adapter la configuration à la plateforme plutôt que copier directement les fichiers d'un environnement vers un autre.

---

## 13. Exercices pratiques

Ces exercices permettent de vérifier les notions abordées.

### Exercice 1 — Rechercher un processus Java

1. Lance une application Java dans un terminal.
2. Dans un autre terminal, exécute `pgrep -a java`.
3. Identifie son PID.
4. Vérifie ses informations avec `ps -p PID -o pid,ppid,user,stat,cmd`.

### Exercice 2 — Comprendre le cycle de vie d'un service

Sur une machine Linux disposant d'un service de test :

1. Vérifie son état avec `systemctl status`.
2. Démarre-le avec `systemctl start`.
3. Vérifie son état avec `systemctl is-active`.
4. Examine ses journaux avec `journalctl`.
5. Vérifie son activation automatique avec `systemctl is-enabled`.

Évite de modifier ou d'arrêter un service système important sur une machine de travail ou de production sans autorisation.

### Exercice 3 — Configurer une application Spring Boot

Dans une machine de test :

1. Crée une unité `my-app.service`.
2. Configure `User`, `WorkingDirectory` et `ExecStart`.
3. Ajoute `SPRING_PROFILES_ACTIVE` avec `Environment=`.
4. Démarre le service et vérifie les journaux.
5. Remplace ensuite la variable par une définition dans `EnvironmentFile=`.
6. Modifie le fichier et redémarre le service pour appliquer la nouvelle valeur.

Observe les différences entre un lancement depuis un terminal et un lancement via systemd.

---

## 14. Références officielles

- [systemd — Documentation officielle](https://systemd.io/)
- [systemctl — Documentation de référence](https://www.freedesktop.org/software/systemd/man/latest/systemctl.html)
- [systemd.service — Documentation des unités de service](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html)
- [systemd.exec — Environnement d'exécution et directives Environment](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html)
- [ps — Documentation Linux](https://man7.org/linux/man-pages/man1/ps.1.html)
- [kill — Documentation Linux](https://man7.org/linux/man-pages/man1/kill.1.html)
- [journalctl — Documentation de référence](https://www.freedesktop.org/software/systemd/man/latest/journalctl.html)
