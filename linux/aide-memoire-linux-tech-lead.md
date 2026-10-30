# Aide-mémoire Linux – Tech Lead

## Objectif

Ce document rassemble les commandes Linux et les réflexes opérationnels qu'un Tech Lead doit connaître pour comprendre, superviser et diagnostiquer le fonctionnement des applications sur des environnements Linux.

Il ne remplace pas la documentation détaillée. Il constitue un référentiel rapide pour les interventions courantes, notamment dans un contexte Java / Spring Boot, Docker et Kubernetes.

---

## 1. Se repérer dans le système

### Commandes essentielles

| Commande | Utilité |
|---|---|
| `pwd` | Afficher le répertoire courant |
| `whoami` | Identifier l'utilisateur courant |
| `hostname` | Identifier le serveur |
| `uname -a` | Afficher les informations système |
| `ls -lah` | Lister les fichiers avec leurs détails |
| `cd /chemin` | Changer de répertoire |
| `find /chemin -name "*.log"` | Rechercher des fichiers |
| `less fichier.log` | Consulter un fichier volumineux |
| `head -n 20 fichier` | Afficher les premières lignes |
| `tail -n 100 fichier` | Afficher les dernières lignes |
| `tail -f fichier.log` | Suivre un fichier en temps réel |
| `grep -i "error" fichier.log` | Rechercher un texte sans distinction de casse |

### Réflexes professionnels

Avant toute intervention sur un serveur :

- Identifier le serveur et l'environnement.
- Vérifier l'utilisateur courant et ses permissions.
- Identifier le répertoire de travail.
- Vérifier les fichiers de configuration avant toute modification.
- Éviter les commandes destructives sans en maîtriser la portée.

Documentation : [Navigation et gestion des fichiers](./01-fondamentaux/navigation-fichiers.md)

---

## 2. Processus et services

Un service Linux peut héberger une application Java, un serveur web ou un composant technique.

### Commandes essentielles

| Commande | Utilité |
|---|---|
| `systemctl status nom-service` | Consulter l'état d'un service |
| `systemctl start nom-service` | Démarrer un service |
| `systemctl stop nom-service` | Arrêter un service |
| `systemctl restart nom-service` | Redémarrer un service |
| `systemctl is-enabled nom-service` | Vérifier son activation au démarrage |
| `systemctl list-units --type=service` | Lister les services actifs |
| `ps aux` | Afficher les processus |
| `pgrep -af java` | Rechercher les processus Java |
| `top` | Observer les processus et leur consommation |
| `kill -15 PID` | Demander l'arrêt d'un processus |
| `free -h` | Consulter la mémoire disponible |

### Réflexes professionnels

Lorsqu'une API Spring Boot ne répond plus :

1. Vérifier l'état du service.
2. Identifier si le processus Java est toujours actif.
3. Examiner les logs.
4. Contrôler la consommation CPU et mémoire.
5. Vérifier les ports réseau utilisés.
6. Analyser les changements récents avant d'envisager un redémarrage.

**Attention :** ne pas utiliser `kill -9` comme première solution. Privilégier un arrêt propre et comprendre pourquoi le processus ne répond plus.

Documentation : [Processus et services](./02-administration/processus-services.md)

---

## 3. Logs et diagnostic

Les logs sont essentiels pour comprendre le comportement d'une application et identifier l'origine d'un incident.

### Commandes essentielles

| Commande | Utilité |
|---|---|
| `journalctl -u nom-service` | Consulter les logs d'un service |
| `journalctl -u nom-service -n 100` | Afficher les 100 dernières lignes |
| `journalctl -u nom-service -f` | Suivre les logs en temps réel |
| `journalctl --since "1 hour ago"` | Consulter les logs récents |
| `tail -f application.log` | Suivre un fichier de logs |
| `grep -i "exception" application.log` | Rechercher les exceptions |
| `grep -iE "error|exception|timeout" application.log` | Rechercher plusieurs types d'erreurs |

### Réflexes professionnels

Pour diagnostiquer un incident :

- Identifier l'heure de début et la durée de l'incident.
- Rechercher les premières erreurs, pas uniquement les dernières.
- Identifier les exceptions Java et leurs causes.
- Repérer les erreurs de connexion à la base de données.
- Rechercher les timeouts et les erreurs réseau.
- Corréler les événements entre les différents services à l'aide des timestamps et des identifiants de corrélation.

Documentation : [Journalisation et diagnostic](./05-exploitation-et-securite/journalisation-diagnostic.md)

---

## 4. Réseau et connectivité

Un problème réseau peut empêcher une API de communiquer avec une base de données, un autre microservice ou un composant externe.

### Commandes essentielles

| Commande | Utilité |
|---|---|
| `ip addr` | Afficher les interfaces réseau |
| `ip route` | Consulter la table de routage |
| `ss -lntp` | Afficher les ports TCP en écoute |
| `ss -antp` | Consulter les connexions TCP |
| `curl -v URL` | Diagnostiquer une requête HTTP |
| `nc -vz hote port` | Tester une connexion TCP |
| `getent hosts nom-hote` | Vérifier la résolution d'un nom |
| `dig nom-hote` | Interroger le DNS |

### Réflexes professionnels

Lorsqu'un service ne parvient pas à joindre une dépendance :

1. Vérifier que le nom d'hôte est correctement résolu.
2. Vérifier que le port cible est accessible.
3. Vérifier que le service distant écoute sur le port attendu.
4. Contrôler les règles de pare-feu et les éventuelles restrictions réseau.
5. Examiner les paramètres de connexion de l'application.

**À distinguer :** une résolution DNS réussie ne garantit pas qu'une connexion TCP ou HTTP fonctionne.

Documentation :
- [Networking](./03-reseau-et-communication/networking.md)
- [Résolution DNS](./03-reseau-et-communication/dns-resolution.md)

---

## 5. Configuration et variables d'environnement

Les applications Java utilisent des variables d'environnement pour récupérer leur configuration d'exécution.

### Commandes essentielles

| Commande | Utilité |
|---|---|
| `env` | Afficher les variables d'environnement |
| `printenv NOM_VARIABLE` | Afficher une variable précise |
| `export NOM=valeur` | Définir une variable pour le shell et ses processus enfants |
| `echo $JAVA_HOME` | Afficher le chemin du JDK |
| `echo $PATH` | Afficher les chemins de recherche des exécutables |

### Points de vigilance Spring Boot

- Distinguer les variables du shell de celles réellement transmises au service systemd.
- Vérifier les profils Spring actifs (`SPRING_PROFILES_ACTIVE`).
- Vérifier les paramètres JVM et les options de démarrage.
- Identifier la source de configuration : fichier YAML, variable d'environnement, argument JVM ou argument applicatif.
- Ne jamais exposer les mots de passe ou les secrets dans les logs ou les commandes partagées.

Documentation : [Variables d'environnement](./04-shell-et-automatisation/environment-variables.md)

---

## 6. Utilisateurs et permissions

Les permissions Linux déterminent les droits de lecture, d'écriture et d'exécution sur les fichiers et répertoires.

### Commandes essentielles

| Commande | Utilité |
|---|---|
| `id` | Afficher l'identité et les groupes de l'utilisateur |
| `groups` | Afficher les groupes de l'utilisateur |
| `ls -l` | Consulter les permissions |
| `namei -l /chemin/fichier` | Examiner les permissions de chaque répertoire du chemin |
| `chmod` | Modifier les permissions |
| `chown` | Modifier le propriétaire |
| `sudo -l` | Consulter les commandes autorisées avec sudo |

### Réflexes professionnels

- Privilégier un utilisateur système dédié pour exécuter une application.
- Appliquer le principe du moindre privilège.
- Vérifier les permissions des fichiers de configuration et des répertoires de logs.
- Éviter les permissions excessives comme `chmod 777`.
- Ne pas exécuter systématiquement les applications en tant que `root`.

Documentation : [Permissions et utilisateurs](./02-administration/permissions-utilisateurs.md)

---

## 7. Ressources système et stockage

Un manque de mémoire, d'espace disque ou d'inodes peut provoquer des dysfonctionnements applicatifs.

### Commandes essentielles

| Commande | Utilité |
|---|---|
| `free -h` | Consulter la mémoire |
| `top` | Observer CPU et mémoire des processus |
| `df -h` | Vérifier l'espace disque |
| `df -i` | Vérifier l'utilisation des inodes |
| `du -sh /chemin` | Mesurer la taille d'un répertoire |
| `lsblk` | Lister les disques et partitions |
| `mount` | Consulter les systèmes de fichiers montés |

### Réflexes professionnels

Lorsqu'une application rencontre des erreurs inhabituelles :

- Vérifier l'espace disque disponible.
- Vérifier les inodes, qui peuvent être épuisés même lorsque l'espace disque semble suffisant.
- Contrôler la mémoire disponible et la consommation des processus.
- Identifier les fichiers de logs volumineux.
- Vérifier les volumes utilisés par l'application.

Documentation : [Gestion des disques et systèmes de fichiers](./02-administration/gestion-disques-systemes-fichiers.md)

---

## 8. Automatisation et tâches planifiées

Il est important de comprendre comment les traitements et tâches automatisés sont exécutés sur un serveur.

### Commandes essentielles

| Commande | Utilité |
|---|---|
| `crontab -l` | Afficher les tâches Cron de l'utilisateur |
| `crontab -e` | Modifier les tâches Cron |
| `systemctl list-timers` | Lister les timers systemd |
| `systemctl status nom.timer` | Consulter l'état d'un timer |
| `systemctl status nom.service` | Consulter le service associé |

### Réflexes professionnels

- Identifier le mécanisme utilisé : Cron, systemd timer ou ordonnanceur applicatif.
- Vérifier la fréquence et les horaires d'exécution.
- Vérifier les logs des traitements.
- S'assurer qu'un traitement ne peut pas être lancé simultanément de manière non maîtrisée.
- Contrôler le comportement en cas d'échec.

Documentation : [Tâches planifiées : Cron et systemd](./02-administration/taches-planifiees-cron-systemd.md)

---

## 9. Sécurité et exploitation

Un Tech Lead doit connaître les principaux risques liés à l'exploitation d'une application sur Linux.

### Points à maîtriser

- Principe du moindre privilège.
- Utilisateurs dédiés aux applications.
- Gestion sécurisée des secrets.
- Mise à jour régulière des paquets.
- Contrôle des ports exposés et des règles de pare-feu.
- Compréhension des mécanismes AppArmor et SELinux.
- Sécurisation des accès SSH.
- Vérification des permissions des fichiers sensibles.

### Commandes utiles

| Commande | Utilité |
|---|---|
| `ss -lntp` | Identifier les ports en écoute |
| `sudo -l` | Vérifier les privilèges sudo |
| `systemctl status ssh` | Vérifier le service SSH (nom variable selon la distribution) |
| `apt list --upgradable` | Consulter les paquets pouvant être mis à jour sur Debian/Ubuntu |
| `ufw status` | Consulter l'état du pare-feu UFW |

Documentation :
- [Sécurité et durcissement Linux](./05-exploitation-et-securite/securite-durcissement-linux.md)
- [Accès distant avec SSH](./03-reseau-et-communication/ssh-acces-distant.md)

---

## 10. Méthodologie de diagnostic d'incident

Lorsqu'une application rencontre un problème en environnement Linux, adopter une démarche structurée permet d'éviter les interventions inutiles.

| Étape | Vérification |
|---|---|
| 1. Périmètre | Identifier les utilisateurs, services et environnements impactés |
| 2. État du service | Vérifier que le service est actif |
| 3. Logs | Rechercher les erreurs et exceptions |
| 4. Processus | Vérifier le processus applicatif et sa consommation |
| 5. Ressources | Contrôler CPU, mémoire, disque et inodes |
| 6. Réseau | Vérifier DNS, ports et connectivité |
| 7. Configuration | Vérifier les variables d'environnement et les paramètres |
| 8. Changements récents | Examiner les derniers déploiements et modifications |
| 9. Correction | Appliquer une correction ciblée et maîtrisée |
| 10. Validation | Vérifier le retour à la normale et documenter l'incident |

### Les erreurs à éviter

- Redémarrer un service sans avoir collecté les informations nécessaires.
- Utiliser `kill -9` sans rechercher la cause du problème.
- Appliquer des permissions excessives pour résoudre rapidement un problème d'accès.
- Modifier directement une configuration de production sans procédure de validation.
- Exposer des secrets dans les commandes, les logs ou l'historique du shell.
- Confondre un problème DNS avec un problème de connectivité réseau.

---

## Références

Ce document est une synthèse opérationnelle. Pour approfondir les sujets :

- [Architecture Linux](./01-fondamentaux/architecture-linux.md)
- [Navigation et gestion des fichiers](./01-fondamentaux/navigation-fichiers.md)
- [Gestion des paquets](./01-fondamentaux/gestion-paquets.md)
- [Shell scripting Bash](./04-shell-et-automatisation/shell-scripting-bash.md)
- [Documentation Linux complète](./README.md)