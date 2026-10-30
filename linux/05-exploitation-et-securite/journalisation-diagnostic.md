# Journalisation et diagnostic Linux

## 1. Introduction

La journalisation (*logging*) consiste à enregistrer les événements qui se produisent sur un système informatique.

Ces événements peuvent concerner :

- Le démarrage et l'arrêt du système.
- L'exécution des services.
- Les connexions utilisateurs.
- Les erreurs applicatives.
- Les problèmes réseau.
- Les événements de sécurité.
- Les opérations réalisées par le noyau Linux.

Les journaux (*logs*) sont indispensables pour comprendre le comportement d'un système et identifier l'origine d'un incident.

### Pourquoi la journalisation est-elle importante ?

Prenons quelques exemples :

- Une application Spring Boot ne démarre plus.
- Un service systemd redémarre continuellement.
- Un serveur manque d'espace disque.
- Une connexion SSH échoue.
- Un conteneur Docker s'arrête immédiatement après son démarrage.
- Une application répond lentement.

Dans ces situations, les logs constituent une source essentielle d'informations.

## 2. Où sont stockés les logs sous Linux ?

Sous Linux, les journaux système sont généralement stockés dans :

```text
/var/log/
```

Ce répertoire contient différents fichiers et sous-répertoires selon la distribution et les services installés.

Exemples :

| Fichier ou répertoire | Utilité |
|---|---|
| `/var/log/syslog` | Journaux système sur Debian/Ubuntu |
| `/var/log/messages` | Journaux système sur certaines distributions RHEL |
| `/var/log/auth.log` | Authentification sur Debian/Ubuntu |
| `/var/log/secure` | Authentification sur certaines distributions RHEL |
| `/var/log/kern.log` | Journaux du noyau sur certaines distributions |
| `/var/log/dpkg.log` | Opérations réalisées avec dpkg |
| `/var/log/apt/` | Journaux des opérations APT |
| `/var/log/nginx/` | Journaux Nginx, si configuré ainsi |

Attention : tous ces fichiers ne sont pas nécessairement présents sur toutes les distributions.

Sur les systèmes utilisant systemd, une partie importante des journaux est également gérée par `systemd-journald`.

## 3. Comprendre les niveaux de journalisation

Les événements peuvent être classés selon leur gravité.

Linux utilise notamment les niveaux de sévérité syslog suivants :

| Niveau | Nom | Signification |
|---:|---|---|
| 0 | emerg | Système inutilisable |
| 1 | alert | Action immédiate nécessaire |
| 2 | crit | Situation critique |
| 3 | err | Erreur |
| 4 | warning | Avertissement |
| 5 | notice | Événement notable |
| 6 | info | Information |
| 7 | debug | Informations de débogage |

Exemple d'utilisation dans un diagnostic :

- `err` : une application n'arrive pas à se connecter à une base de données.
- `warning` : une ressource approche d'une limite.
- `info` : un service vient de démarrer.
- `debug` : détails techniques utiles pour comprendre un comportement.

Les applications peuvent utiliser leur propre convention de niveaux, mais les niveaux ci-dessus constituent la référence syslog.

## 4. Consulter les logs avec les commandes Linux classiques

Avant d'étudier systemd-journald, il faut maîtriser les commandes classiques de consultation des fichiers.

### 4.1 cat

Afficher le contenu d'un fichier :

```bash
cat /var/log/syslog
```

Cette commande affiche l'intégralité du fichier. Elle est donc peu adaptée aux fichiers volumineux.

### 4.2 less

Consulter un fichier de manière interactive :

```bash
less /var/log/syslog
```

Raccourcis utiles :

| Touche | Action |
|---|---|
| `Espace` | Page suivante |
| `b` | Page précédente |
| `/mot` | Rechercher un mot |
| `n` | Résultat suivant |
| `N` | Résultat précédent |
| `q` | Quitter |

### 4.3 head et tail

Afficher les premières lignes :

```bash
head -n 20 /var/log/syslog
```

Afficher les dernières lignes :

```bash
tail -n 50 /var/log/syslog
```

Suivre les nouvelles lignes en temps réel :

```bash
tail -f /var/log/syslog
```

Pour suivre un fichier même après sa rotation et son remplacement :

```bash
tail -F /var/log/syslog
```

C'est particulièrement utile lorsqu'on observe un fichier de logs pendant le démarrage d'un service.

### 4.4 grep

Rechercher une erreur :

```bash
grep "ERROR" application.log
```

Ignorer la casse :

```bash
grep -i "error" application.log
```

Afficher les numéros de ligne :

```bash
grep -n "ERROR" application.log
```

Afficher quelques lignes autour d'une erreur :

```bash
grep -C 5 "ERROR" application.log
```

Rechercher récursivement dans un répertoire :

```bash
grep -r "Connection refused" /var/log/
```

### 4.5 Combiner les commandes

Les pipes permettent de construire des recherches plus précises.

Exemple : suivre les logs et afficher uniquement les erreurs :

```bash
tail -f application.log | grep --line-buffered -i "error"
```

Afficher les erreurs récentes d'un fichier :

```bash
grep -i "error" application.log | tail -n 20
```

## 5. systemd-journald et journalctl

Sur les distributions utilisant systemd, `systemd-journald` collecte et centralise les journaux provenant notamment :

- Du noyau Linux.
- Des services systemd.
- Des applications envoyant leurs messages au journal.
- De certaines sorties standard et sorties d'erreur des services.

La commande `journalctl` permet de consulter ces journaux.

### 5.1 Consulter les journaux

Afficher tous les journaux accessibles :

```bash
journalctl
```

Afficher les dernières entrées :

```bash
journalctl -n 50
```

Suivre les journaux en temps réel :

```bash
journalctl -f
```

Afficher les entrées depuis le dernier démarrage :

```bash
journalctl -b
```

Afficher les journaux du démarrage précédent :

```bash
journalctl -b -1
```

Cette dernière commande suppose que les journaux du démarrage précédent ont été conservés.

### 5.2 Filtrer par service

C'est l'une des fonctionnalités les plus utiles.

Afficher les logs d'un service Spring Boot géré par systemd :

```bash
journalctl -u spring-api
```

Afficher les 100 dernières lignes :

```bash
journalctl -u spring-api -n 100
```

Suivre les logs en temps réel :

```bash
journalctl -u spring-api -f
```

Afficher les logs depuis le dernier démarrage :

```bash
journalctl -u spring-api -b
```

### 5.3 Filtrer par période

Afficher les journaux depuis une date :

```bash
journalctl --since "2026-10-30 08:00:00"
```

Afficher les journaux entre deux dates :

```bash
journalctl --since "2026-10-30 08:00:00" \
           --until "2026-10-30 12:00:00"
```

Utiliser des périodes relatives :

```bash
journalctl --since "1 hour ago"
journalctl --since "30 minutes ago"
journalctl --since today
```

### 5.4 Filtrer par niveau de gravité

Afficher les erreurs et les niveaux plus graves :

```bash
journalctl -p err
```

Afficher les avertissements et les erreurs :

```bash
journalctl -p warning
```

Afficher uniquement les informations :

```bash
journalctl -p info
```

### 5.5 Combiner les filtres

Exemple : afficher les erreurs d'un service depuis une heure :

```bash
journalctl -u spring-api -p err --since "1 hour ago"
```

Suivre les logs d'un service avec les messages détaillés :

```bash
journalctl -u spring-api -f -o verbose
```

Le format `verbose` affiche davantage de métadonnées associées aux événements.

### 5.6 Identifier le processus à l'origine d'un événement

Le journal contient des métadonnées permettant d'identifier l'origine d'un message.

Quelques champs utiles :

| Champ | Description |
|---|---|
| `_PID` | Identifiant du processus |
| `_UID` | Identifiant de l'utilisateur |
| `_SYSTEMD_UNIT` | Unité systemd associée |
| `SYSLOG_IDENTIFIER` | Identifiant du programme |
| `MESSAGE` | Contenu du message |
| `PRIORITY` | Niveau de priorité |

Exemple :

```bash
journalctl _PID=1234
```

Cette commande affiche les messages associés au processus 1234.

## 6. Stockage et conservation des journaux systemd

Les journaux systemd peuvent être stockés de deux manières :

- En mémoire, dans `/run/log/journal/`.
- De manière persistante, dans `/var/log/journal/`.

La persistance dépend de la configuration du système.

### Vérifier l'utilisation du journal

```bash
journalctl --disk-usage
```

### Nettoyer les anciens journaux

Conserver uniquement les journaux des 7 derniers jours :

```bash
sudo journalctl --vacuum-time=7d
```

Limiter la taille totale :

```bash
sudo journalctl --vacuum-size=500M
```

Attention : ces commandes suppriment des journaux archivés. Il faut tenir compte des exigences de conservation et de conformité avant de les utiliser sur un serveur de production.

### Configuration de journald

Le fichier principal de configuration est :

```text
/etc/systemd/journald.conf
```

Exemple de paramètres :

```ini
[Journal]
Storage=persistent
SystemMaxUse=1G
SystemKeepFree=2G
MaxRetentionSec=30day
```

Explications :

- `Storage=persistent` : demande un stockage persistant.
- `SystemMaxUse` : limite l'espace disque utilisé par les journaux système.
- `SystemKeepFree` : espace que journald tente de préserver.
- `MaxRetentionSec` : durée maximale de conservation.

Les limites peuvent interagir entre elles et avec les contraintes du système de fichiers.

Après modification :

```bash
sudo systemctl restart systemd-journald
```

## 7. rsyslog : centralisation des journaux

`rsyslog` est un service de journalisation qui permet notamment de collecter, traiter, filtrer et transférer des messages syslog.

Il peut fonctionner en complément de `systemd-journald`.

### Vérifier le service

```bash
sudo systemctl status rsyslog
```

### Installation sur Ubuntu

```bash
sudo apt install rsyslog
```

### Configuration

Le répertoire de configuration est généralement :

```text
/etc/rsyslog.d/
```

Le fichier principal est :

```text
/etc/rsyslog.conf
```

Exemple de règle :

```text
*.err /var/log/errors.log
```

Cette règle demande à rsyslog d'enregistrer les messages de niveau erreur et plus graves dans `errors.log`.

Après modification :

```bash
sudo systemctl restart rsyslog
```

### Centralisation des logs

rsyslog peut transmettre les journaux vers un serveur central.

Cela permet de regrouper les événements de plusieurs machines et de faciliter leur analyse.

Dans un environnement distribué, cette centralisation est particulièrement utile pour corréler les événements provenant de plusieurs serveurs.

## 8. Rotation des logs avec logrotate

Les fichiers de logs peuvent rapidement devenir volumineux.

`logrotate` permet d'automatiser leur rotation et leur archivage.

Il permet notamment de :

- Renommer les anciens fichiers.
- Créer de nouveaux fichiers de logs.
- Compresser les anciennes versions.
- Supprimer les archives trop anciennes.
- Limiter l'espace disque utilisé.

### 8.1 Configuration

Le fichier principal est :

```text
/etc/logrotate.conf
```

Les configurations spécifiques se trouvent généralement dans :

```text
/etc/logrotate.d/
```

### 8.2 Exemple de configuration

Créer :

```text
/etc/logrotate.d/spring-api
```

Contenu :

```text
/var/log/spring-api/application.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
```

Explications :

| Directive | Description |
|---|---|
| `daily` | Rotation quotidienne |
| `rotate 14` | Conserver 14 archives |
| `compress` | Compresser les archives |
| `delaycompress` | Reporter la compression à la rotation suivante |
| `missingok` | Ne pas signaler d'erreur si le fichier est absent |
| `notifempty` | Ne pas effectuer de rotation si le fichier est vide |
| `copytruncate` | Copier le fichier puis tronquer l'original |

**Attention à `copytruncate` :** il existe une petite fenêtre pendant laquelle des messages peuvent être perdus entre la copie et la troncature. Lorsque l'application sait rouvrir ses fichiers de logs, une rotation par renommage et signal de réouverture est souvent préférable.

### 8.3 Tester la configuration

Vérifier la configuration sans effectuer de rotation :

```bash
sudo logrotate -d /etc/logrotate.d/spring-api
```

Forcer une rotation :

```bash
sudo logrotate -f /etc/logrotate.d/spring-api
```

La rotation forcée doit être utilisée avec précaution, particulièrement en production.

## 9. Diagnostiquer les ressources système

La journalisation ne suffit pas toujours. Il faut également savoir examiner l'état du système.

### 9.1 CPU

Afficher la charge système :

```bash
uptime
```

Afficher les processus :

```bash
top
```

Une alternative plus ergonomique :

```bash
htop
```

Installation :

```bash
sudo apt install htop
```

Quelques informations importantes :

- Utilisation CPU.
- Nombre de processus.
- Charge système.
- Processus consommant le plus de ressources.

Attention : la *load average* ne représente pas directement le pourcentage d'utilisation CPU. Elle prend notamment en compte les processus exécutables et ceux en attente non interruptible.

### 9.2 Mémoire

Afficher l'utilisation de la mémoire :

```bash
free -h
```

Afficher les processus consommateurs :

```bash
ps aux --sort=-%mem | head
```

Consulter les informations mémoire :

```bash
cat /proc/meminfo
```

### 9.3 Disque

Afficher l'espace disponible :

```bash
df -h
```

Identifier les répertoires volumineux :

```bash
du -sh /var/log/*
```

Identifier les fichiers volumineux :

```bash
find /var/log -type f -size +100M
```

### 9.4 Inodes

Un système de fichiers peut manquer d'inodes même s'il reste de l'espace disque.

Vérifier leur utilisation :

```bash
df -i
```

### 9.5 Processus

Afficher les processus :

```bash
ps aux
```

Rechercher un processus Java :

```bash
pgrep -af java
```

Afficher les fichiers ouverts par un processus :

```bash
lsof -p 1234
```

Ces commandes sont particulièrement utiles lorsqu'une application consomme trop de ressources ou conserve des fichiers ouverts.

## 10. Diagnostic réseau

Lorsqu'une application ne répond plus, il faut aussi examiner le réseau.

### Vérifier les interfaces

```bash
ip addr
```

Afficher les routes :

```bash
ip route
```

Tester la connectivité :

```bash
ping 8.8.8.8
```

Vérifier la résolution DNS :

```bash
getent hosts google.com
```

### Vérifier les ports en écoute

```bash
ss -tulnp
```

Afficher les connexions TCP :

```bash
ss -tnp
```

Vérifier si un port est ouvert localement :

```bash
ss -tlnp | grep :8080
```

Tester un service HTTP :

```bash
curl -v http://localhost:8080/actuator/health
```

Pour Spring Boot, cette dernière commande peut permettre de vérifier rapidement si l'application répond et si son endpoint de santé est accessible.

## 11. Diagnostic des services systemd

Lorsqu'un service ne démarre pas, plusieurs commandes permettent d'identifier le problème.

Afficher son état :

```bash
systemctl status spring-api
```

Consulter ses journaux :

```bash
journalctl -u spring-api -n 100
```

Suivre les journaux :

```bash
journalctl -u spring-api -f
```

Afficher les dernières erreurs :

```bash
journalctl -u spring-api -p err --since today
```

Vérifier la configuration de l'unité :

```bash
systemctl cat spring-api
```

Afficher les propriétés effectives :

```bash
systemctl show spring-api
```

Vérifier si le service redémarre continuellement :

```bash
systemctl show spring-api -p NRestarts
```

## 12. Diagnostic d'une application Spring Boot

Dans un environnement Java, les logs applicatifs sont essentiels pour comprendre les erreurs métier et techniques.

### 12.1 Configuration du logging

Spring Boot utilise généralement Logback par défaut avec le starter de logging.

Exemple de configuration dans `application.yml` :

```yaml
logging:
  level:
    root: INFO
    com.example: DEBUG
  file:
    name: /var/log/spring-api/application.log
  logback:
    rollingpolicy:
      max-file-size: 100MB
      max-history: 14
```

Cette configuration permet :

- De définir le niveau global à `INFO`.
- D'activer `DEBUG` pour le package applicatif.
- D'écrire les logs dans un fichier.
- De limiter la taille des fichiers et leur historique.

Les paramètres de rotation dépendent du système de logging utilisé et de sa configuration.

### 12.2 Consulter les logs

Si l'application est gérée par systemd :

```bash
journalctl -u spring-api -f
```

Si elle écrit dans un fichier :

```bash
tail -f /var/log/spring-api/application.log
```

### 12.3 Rechercher une exception

```bash
grep -n -i "exception" application.log
```

Rechercher les erreurs SQL :

```bash
grep -n -i "SQLException" application.log
```

Rechercher les problèmes de connexion :

```bash
grep -n -i "connection refused" application.log
```

### 12.4 Actuator et Health Checks

Spring Boot Actuator permet d'exposer des informations de santé et de supervision.

Exemple :

```bash
curl http://localhost:8080/actuator/health
```

Il peut fournir des informations sur :

- L'état général de l'application.
- La connexion à la base de données.
- Certains composants applicatifs.
- Les dépendances configurées.

Les endpoints Actuator doivent être sécurisés et ne pas exposer d'informations sensibles publiquement.

## 13. Diagnostic Docker

Dans un environnement Docker, les logs des conteneurs constituent une source importante d'informations.

Afficher les conteneurs actifs :

```bash
docker ps
```

Afficher tous les conteneurs :

```bash
docker ps -a
```

Consulter les logs :

```bash
docker logs mon-conteneur
```

Afficher les dernières lignes :

```bash
docker logs --tail 100 mon-conteneur
```

Suivre les logs en temps réel :

```bash
docker logs -f mon-conteneur
```

Afficher les logs depuis une période :

```bash
docker logs --since 30m mon-conteneur
```

Inspecter un conteneur :

```bash
docker inspect mon-conteneur
```

Consulter sa consommation de ressources :

```bash
docker stats
```

### Attention à la rotation

Docker utilise un pilote de journalisation configurable.

Le pilote `json-file` peut notamment être configuré avec des limites :

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Cette configuration concerne le démon Docker et doit être placée dans `/etc/docker/daemon.json`.

Après modification :

```bash
sudo systemctl restart docker
```

Attention : le redémarrage du démon peut affecter les conteneurs selon la configuration de l'environnement. Les options de rotation s'appliquent généralement aux conteneurs créés après le changement ; il peut être nécessaire de les recréer.

## 14. Diagnostic Kubernetes

Dans Kubernetes, les logs sont principalement consultés à travers `kubectl`.

Afficher les pods :

```bash
kubectl get pods
```

Afficher les détails d'un pod :

```bash
kubectl describe pod mon-pod
```

Consulter les logs :

```bash
kubectl logs mon-pod
```

Suivre les logs :

```bash
kubectl logs -f mon-pod
```

Afficher les logs d'un conteneur spécifique :

```bash
kubectl logs mon-pod -c mon-conteneur
```

Afficher les logs de l'instance précédente après un redémarrage :

```bash
kubectl logs mon-pod --previous
```

Afficher les événements :

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Cas d'un pod en CrashLoopBackOff

Un pod en `CrashLoopBackOff` redémarre de manière répétée.

Premières vérifications :

```bash
kubectl describe pod mon-pod
kubectl logs mon-pod --previous
kubectl get events
```

Les causes possibles incluent :

- Une erreur de configuration.
- Une variable d'environnement manquante.
- Une dépendance indisponible.
- Une erreur au démarrage de l'application.
- Une sonde de santé mal configurée.
- Une limite de ressources trop restrictive.

## 15. Méthodologie de diagnostic d'un incident

Lorsqu'un incident survient, il est préférable de suivre une démarche structurée.

### Étape 1 : identifier le problème

Exemples :

- Application inaccessible.
- Service arrêté.
- Consommation mémoire anormale.
- Erreur de connexion à PostgreSQL.

### Étape 2 : vérifier l'état du service

```bash
systemctl status spring-api
```

### Étape 3 : consulter les logs

```bash
journalctl -u spring-api -n 200
```

### Étape 4 : vérifier les ressources

```bash
top
free -h
df -h
```

### Étape 5 : vérifier le réseau

```bash
ss -tulnp
curl -v http://localhost:8080/actuator/health
```

### Étape 6 : identifier la cause

Il faut corréler les événements :

- Heure de l'incident.
- Erreurs applicatives.
- Logs système.
- Changements de configuration.
- Redémarrages de services.
- État des dépendances.

### Étape 7 : corriger et vérifier

Après correction :

- Vérifier le retour du service.
- Vérifier les logs.
- Vérifier les métriques.
- Contrôler que l'incident ne se reproduit pas.

## 16. Bonnes pratiques de journalisation

### Côté système

- Centraliser les logs lorsque l'environnement le nécessite.
- Configurer une rotation adaptée.
- Limiter l'espace disque consommé.
- Définir une politique de conservation.
- Surveiller les erreurs système.
- Protéger les fichiers de logs contre les accès non autorisés.

### Côté application

- Utiliser des niveaux de logs cohérents.
- Ajouter des informations de contexte.
- Inclure un identifiant de corrélation pour suivre une requête entre plusieurs services.
- Éviter les logs trop volumineux.
- Ne jamais journaliser les mots de passe, tokens JWT ou secrets.
- Éviter de journaliser des données personnelles sensibles.
- Privilégier des logs structurés, notamment au format JSON, dans les architectures distribuées.

### En environnement distribué

Pour une architecture microservices, il devient intéressant d'utiliser une plateforme centralisée d'observabilité.

Quelques solutions :

- **Loki + Grafana** : collecte, consultation et visualisation des logs.
- **Elastic Stack** : Elasticsearch, Logstash et Kibana.
- **OpenSearch** : moteur de recherche et d'analyse de données.
- **Fluent Bit** : collecte et transfert des logs.

Il faut distinguer trois piliers complémentaires :

| Pilier | Objectif | Exemples |
|---|---|---|
| Logs | Comprendre les événements | Erreurs, exceptions, traces |
| Metrics | Mesurer l'état du système | CPU, mémoire, latence |
| Traces | Suivre une requête entre services | Appels distribués |

Prometheus est principalement utilisé pour les métriques, tandis que Grafana permet de visualiser différentes sources de données.

## 17. Exercices pratiques

### Exercice 1 : consultation des logs

1. Afficher les dernières lignes de `/var/log/syslog` si le fichier existe.
2. Rechercher les erreurs.
3. Afficher les événements depuis une heure.
4. Suivre les nouveaux événements en temps réel.

### Exercice 2 : journalisation systemd

1. Identifier les services actifs.
2. Consulter les logs de SSH.
3. Afficher uniquement les erreurs.
4. Consulter les journaux du dernier démarrage.

### Exercice 3 : rotation des logs

1. Installer logrotate si nécessaire.
2. Créer une configuration pour un fichier applicatif.
3. Tester la configuration en mode simulation.
4. Effectuer une rotation et vérifier les fichiers générés.

### Exercice 4 : diagnostic Spring Boot

1. Démarrer une application Spring Boot.
2. Consulter ses logs.
3. Provoquer une erreur de connexion à PostgreSQL.
4. Identifier l'erreur dans les logs.
5. Vérifier l'état de l'application avec Actuator.

### Exercice 5 : diagnostic Docker

1. Démarrer un conteneur.
2. Consulter ses logs.
3. Provoquer une erreur de démarrage.
4. Identifier la cause avec `docker logs` et `docker inspect`.
5. Examiner la consommation de ressources avec `docker stats`.

### Exercice 6 : diagnostic Kubernetes

1. Déployer une application.
2. Consulter ses logs.
3. Examiner les événements du namespace.
4. Identifier les raisons d'un redémarrage.
5. Consulter les logs du conteneur précédent.

## 18. Résumé des commandes essentielles

| Commande | Utilité |
|---|---|
| `less fichier.log` | Consulter un fichier |
| `tail -f fichier.log` | Suivre les logs |
| `grep "ERROR" fichier.log` | Rechercher une erreur |
| `journalctl` | Consulter le journal systemd |
| `journalctl -u service` | Logs d'un service |
| `journalctl -f` | Suivre les journaux |
| `journalctl -p err` | Afficher les erreurs |
| `journalctl --since` | Filtrer par période |
| `journalctl --disk-usage` | Taille des journaux |
| `journalctl --vacuum-time` | Nettoyer les anciens journaux |
| `systemctl status` | État d'un service |
| `logrotate -d` | Simuler une rotation |
| `df -h` | Espace disque |
| `df -i` | Utilisation des inodes |
| `free -h` | Utilisation mémoire |
| `top` | Activité des processus |
| `ss -tulnp` | Ports en écoute |
| `lsof -p` | Fichiers ouverts par un processus |
| `docker logs` | Logs d'un conteneur |
| `kubectl logs` | Logs d'un pod |
| `kubectl describe pod` | Détails et événements d'un pod |

## 19. Ressources officielles

- systemd-journald : https://www.freedesktop.org/software/systemd/man/latest/systemd-journald.service.html
- journalctl : https://www.freedesktop.org/software/systemd/man/latest/journalctl.html
- rsyslog : https://www.rsyslog.com/doc/
- logrotate : https://github.com/logrotate/logrotate
- Docker Logging : https://docs.docker.com/engine/logging/
- Kubernetes Logging : https://kubernetes.io/docs/concepts/cluster-administration/logging/
- Spring Boot Logging : https://docs.spring.io/spring-boot/reference/features/logging.html