# Tâches planifiées avec Cron et Systemd Timers

## 1. Introduction

Linux propose plusieurs mécanismes permettant d'exécuter automatiquement des commandes ou des scripts à des moments précis.

Les deux mécanismes les plus courants sont :

- **Cron** : planificateur historique permettant d'exécuter des commandes selon une expression temporelle.
- **Systemd Timers** : mécanisme de planification intégré à systemd, permettant de déclencher des services à des moments précis ou à intervalles réguliers.

Ils permettent notamment de :

- Lancer une sauvegarde chaque nuit.
- Nettoyer des fichiers temporaires.
- Exécuter un batch Java.
- Contrôler périodiquement l'état d'un service.
- Générer des rapports.
- Effectuer des opérations de maintenance.

## 2. Cron : le planificateur historique

Cron est un service Linux qui permet d'exécuter automatiquement des commandes selon une planification définie.

Le service cron surveille les tâches configurées et déclenche leur exécution aux horaires prévus.

### 2.1 Vérifier le service Cron

Sur Debian / Ubuntu :

```bash
sudo systemctl status cron
```

Sur certaines distributions, notamment Fedora et RHEL :

```bash
sudo systemctl status crond
```

Démarrer le service :

```bash
sudo systemctl start cron
```

Activer son démarrage automatique :

```bash
sudo systemctl enable cron
```

Ou combiner les deux :

```bash
sudo systemctl enable --now cron
```

Le nom du service dépend de la distribution.

## 3. Les crontabs

Une *crontab* est un fichier contenant les tâches planifiées d'un utilisateur.

Chaque utilisateur peut disposer de sa propre crontab.

### 3.1 Consulter les tâches planifiées

Afficher les tâches de l'utilisateur courant :

```bash
crontab -l
```

Modifier la crontab :

```bash
crontab -e
```

Supprimer toutes les tâches de l'utilisateur :

```bash
crontab -r
```

Attention : `crontab -r` supprime toutes les tâches de l'utilisateur sans demander de confirmation.

### 3.2 Crontab système

Il existe également des tâches planifiées au niveau système.

Les emplacements courants sont :

| Emplacement | Description |
|---|---|
| `/etc/crontab` | Crontab système |
| `/etc/cron.d/` | Fichiers de planification spécifiques |
| `/etc/cron.hourly/` | Scripts exécutés approximativement chaque heure |
| `/etc/cron.daily/` | Scripts exécutés quotidiennement |
| `/etc/cron.weekly/` | Scripts exécutés chaque semaine |
| `/etc/cron.monthly/` | Scripts exécutés chaque mois |

Différence importante :

- Dans la crontab personnelle, le nom de l'utilisateur n'est pas indiqué.
- Dans `/etc/crontab` et les fichiers de `/etc/cron.d/`, il faut préciser l'utilisateur qui exécutera la commande.

## 4. Syntaxe d'une expression Cron

Une expression Cron standard contient cinq champs temporels :

```text
* * * * * commande
│ │ │ │ │
│ │ │ │ └── Jour de la semaine (0-7)
│ │ │ └──── Mois (1-12)
│ │ └────── Jour du mois (1-31)
│ └──────── Heure (0-23)
└────────── Minute (0-59)
```

Les valeurs 0 et 7 représentent généralement le dimanche.

### 4.1 Les opérateurs Cron

| Opérateur | Signification | Exemple |
|---|---|---|
| `*` | Toutes les valeurs | `* * * * *` |
| `,` | Plusieurs valeurs | `0,30 * * * *` |
| `-` | Intervalle | `0 8-18 * * *` |
| `/` | Pas ou fréquence | `*/15 * * * *` |

### 4.2 Exemples de planification

Exécuter une commande toutes les minutes :

```cron
* * * * * /opt/scripts/monitor.sh
```

Toutes les 15 minutes :

```cron
*/15 * * * * /opt/scripts/monitor.sh
```

Toutes les heures :

```cron
0 * * * * /opt/scripts/monitor.sh
```

Tous les jours à 2 heures du matin :

```cron
0 2 * * * /opt/scripts/backup.sh
```

Tous les jours à 8 h 30 :

```cron
30 8 * * * /opt/scripts/report.sh
```

Tous les lundis à 6 heures :

```cron
0 6 * * 1 /opt/scripts/weekly-task.sh
```

Le premier jour de chaque mois à minuit :

```cron
0 0 1 * * /opt/scripts/monthly-task.sh
```

Du lundi au vendredi à 9 heures :

```cron
0 9 * * 1-5 /opt/scripts/workday-task.sh
```

### 4.3 Attention à la combinaison jour du mois / jour de la semaine

Dans Cron, lorsque les deux champs sont renseignés avec des valeurs spécifiques, leur combinaison suit généralement une logique OU.

Exemple :

```cron
0 9 1 * 1 /opt/scripts/task.sh
```

Cette tâche s'exécute à 9 heures le premier jour du mois **ou** chaque lundi.

Ce comportement est important pour éviter les erreurs de planification.

## 5. Redirections et journalisation des tâches Cron

Par défaut, les sorties des commandes peuvent être envoyées par le mécanisme de courrier local, si celui-ci est configuré.

Il est généralement préférable de gérer explicitement les sorties.

### 5.1 Rediriger la sortie vers un fichier

```cron
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

Explication :

- `>>` : ajoute la sortie standard au fichier.
- `2>&1` : redirige également les erreurs vers le même fichier.

### 5.2 Ignorer les sorties

```cron
0 2 * * * /opt/scripts/backup.sh > /dev/null 2>&1
```

Cette solution évite la production de sorties, mais elle peut rendre le diagnostic plus difficile.

Il est préférable de conserver des logs pour les tâches importantes.

## 6. Bonnes pratiques avec Cron

Cron exécute les commandes dans un environnement différent d'une session interactive.

Il ne faut donc pas supposer qu'il dispose automatiquement des mêmes variables et du même `PATH` que ton terminal.

### 6.1 Utiliser des chemins absolus

À éviter :

```cron
0 2 * * * backup.sh
```

À privilégier :

```cron
0 2 * * * /opt/scripts/backup.sh
```

Utiliser également des chemins absolus dans les scripts pour les exécutables importants.

### 6.2 Définir explicitement l'environnement

Exemple de crontab :

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

0 2 * * * /opt/scripts/backup.sh
```

Si un script Java nécessite une configuration spécifique, il faut définir explicitement les variables nécessaires ou charger un fichier d'environnement prévu à cet effet.

### 6.3 Vérifier les permissions

Le script doit être accessible et exécutable par l'utilisateur qui l'exécute.

```bash
chmod 750 /opt/scripts/backup.sh
```

Il faut également vérifier les droits sur les fichiers et répertoires utilisés par le script.

### 6.4 Éviter les exécutions simultanées

Une tâche peut durer plus longtemps que son intervalle de planification.

Exemple : un batch démarre toutes les 10 minutes mais dure 20 minutes.

Deux exécutions peuvent alors se chevaucher.

La commande `flock` permet d'éviter ce problème :

```cron
*/10 * * * * /usr/bin/flock -n /var/lock/my-batch.lock /opt/scripts/batch.sh
```

Si une instance détient déjà le verrou, la nouvelle exécution ne démarre pas.

Il faut vérifier que le chemin du fichier de verrouillage est accessible à l'utilisateur exécutant la tâche.

## 7. Exemple pratique : planifier un batch Java avec Cron

Imaginons un batch Spring Batch qui doit être exécuté chaque nuit à 2 heures.

Commande Java :

```bash
/usr/bin/java -jar /opt/batch/application.jar
```

Créer un script :

```bash
sudo nano /opt/scripts/run-batch.sh
```

Contenu :

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

APP="/opt/batch/application.jar"
LOG_DIR="/var/log/batch"

mkdir -p "$LOG_DIR"

if [[ ! -f "$APP" ]]; then
    echo "Fichier JAR introuvable : $APP" >&2
    exit 1
fi

LOG_FILE="${LOG_DIR}/batch-$(date +%F).log"

echo "Démarrage du batch : $(date)"

if /usr/bin/java -jar "$APP" >> "$LOG_FILE" 2>&1; then
    echo "Batch terminé avec succès : $(date)"
else
    echo "Échec du batch : $(date)" >&2
    exit 1
fi
```

Rendre le script exécutable :

```bash
sudo chmod 750 /opt/scripts/run-batch.sh
```

Ajouter une tâche dans la crontab de l'utilisateur concerné :

```bash
crontab -e
```

Ajouter :

```cron
0 2 * * * /usr/bin/flock -n /tmp/batch.lock /opt/scripts/run-batch.sh
```

Le batch est maintenant planifié chaque jour à 2 heures.

En production, il faudra également prévoir une stratégie de rotation des logs, une gestion des erreurs et une supervision de l'exécution.

## 8. Systemd Timers : une alternative moderne à Cron

Systemd propose un mécanisme de planification intégré appelé *Systemd Timers*.

Contrairement à Cron, un timer déclenche une unité systemd, généralement un service.

Il permet notamment de :

- Planifier l'exécution d'un service.
- Définir des intervalles temporels.
- Exécuter une tâche après le démarrage du système.
- Gérer les dépendances entre unités.
- Consulter les logs avec journalctl.
- Définir le comportement d'une tâche lorsque le système était arrêté au moment prévu.

Un timer est constitué de deux fichiers :

- Un fichier `.service` qui définit ce qu'il faut exécuter.
- Un fichier `.timer` qui définit quand l'exécuter.

## 9. Créer un service systemd

Prenons le même exemple : exécuter un batch Java chaque nuit.

Créer le fichier :

```bash
sudo nano /etc/systemd/system/batch.service
```

Contenu :

```ini
[Unit]
Description=Spring Batch execution

[Service]
Type=oneshot
User=batch
WorkingDirectory=/opt/batch

ExecStart=/usr/bin/java -jar /opt/batch/application.jar

StandardOutput=journal
StandardError=journal
```

Explications :

| Directive | Description |
|---|---|
| `Description` | Description du service |
| `Type=oneshot` | Service exécutant une tâche ponctuelle |
| `User` | Utilisateur exécutant la tâche |
| `WorkingDirectory` | Répertoire de travail |
| `ExecStart` | Commande à exécuter |
| `StandardOutput` | Destination de la sortie standard |
| `StandardError` | Destination des erreurs |

L'utilisateur `batch` doit exister et disposer des permissions nécessaires sur l'application, ses fichiers et les ressources utilisées.

Contrairement à un service applicatif classique, ce service ne reste pas nécessairement actif après la fin du traitement.

## 10. Créer un Systemd Timer

Créer le fichier :

```bash
sudo nano /etc/systemd/system/batch.timer
```

Contenu :

```ini
[Unit]
Description=Daily Spring Batch execution

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
Unit=batch.service

[Install]
WantedBy=timers.target
```

### Explication des directives

**OnCalendar**

Définit la date et l'heure d'exécution.

```ini
OnCalendar=*-*-* 02:00:00
```

Signifie : tous les jours à 2 heures du matin.

**Persistent=true**

Si le système était arrêté à l'heure prévue, systemd déclenche la tâche au prochain démarrage, lorsque le timer est activé et que le service est à nouveau disponible.

Cela ne signifie pas que toutes les exécutions manquées seront rejouées : une exécution manquée peut être rattrapée.

**Unit=batch.service**

Indique le service à déclencher.

## 11. Activer et vérifier un Systemd Timer

Après création des fichiers :

Recharger la configuration systemd :

```bash
sudo systemctl daemon-reload
```

Activer le timer au démarrage et le démarrer immédiatement :

```bash
sudo systemctl enable --now batch.timer
```

Vérifier son état :

```bash
systemctl status batch.timer
```

Lister les timers actifs :

```bash
systemctl list-timers
```

Afficher également les timers inactifs :

```bash
systemctl list-timers --all
```

Consulter les prochaines exécutions :

```bash
systemctl list-timers batch.timer
```

## 12. Consulter les logs d'un Systemd Timer

Les logs sont centralisés dans journald.

Consulter les logs du service :

```bash
journalctl -u batch.service
```

Afficher les dernières exécutions :

```bash
journalctl -u batch.service -n 100
```

Suivre les logs :

```bash
journalctl -u batch.service -f
```

Afficher les informations du timer :

```bash
systemctl status batch.timer
```

Afficher les propriétés du service :

```bash
systemctl show batch.service
```

Cette intégration avec systemd et journald facilite le diagnostic des traitements planifiés.

## 13. Les différentes directives temporelles

Systemd Timers propose plusieurs directives.

### 13.1 OnCalendar

Permet de définir une planification basée sur le calendrier.

Tous les jours à 2 heures :

```ini
OnCalendar=*-*-* 02:00:00
```

Tous les lundis à 8 heures :

```ini
OnCalendar=Mon *-*-* 08:00:00
```

Le premier jour de chaque mois :

```ini
OnCalendar=*-*-01 00:00:00
```

Toutes les heures :

```ini
OnCalendar=hourly
```

Tous les jours :

```ini
OnCalendar=daily
```

Toutes les semaines :

```ini
OnCalendar=weekly
```

### 13.2 OnBootSec

Permet de déclencher une tâche après le démarrage du système.

Exemple :

```ini
OnBootSec=5min
```

La tâche est déclenchée cinq minutes après le démarrage.

### 13.3 OnUnitActiveSec

Permet de planifier une exécution après la dernière activation de l'unité associée.

Exemple :

```ini
OnUnitActiveSec=1h
```

La prochaine activation est planifiée une heure après la dernière activation.

### 13.4 OnStartupSec

Permet de définir un délai à partir du démarrage du gestionnaire systemd.

Exemple :

```ini
OnStartupSec=10min
```

### 13.5 AccuracySec

Permet de définir la précision de déclenchement souhaitée.

Exemple :

```ini
AccuracySec=1min
```

Systemd peut regrouper certains déclenchements dans la fenêtre de précision définie afin de limiter les réveils du système.

## 14. Exemple : exécuter un traitement toutes les 15 minutes

Fichier `/etc/systemd/system/monitor.service` :

```ini
[Unit]
Description=Application monitoring

[Service]
Type=oneshot
ExecStart=/opt/scripts/monitor.sh
```

Fichier `/etc/systemd/system/monitor.timer` :

```ini
[Unit]
Description=Monitor every 15 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=15min
Unit=monitor.service

[Install]
WantedBy=timers.target
```

Activer :

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now monitor.timer
```

Le premier déclenchement intervient deux minutes après le démarrage du système, puis les activations suivantes sont espacées de 15 minutes à partir de l'activation du service.

Cette approche est différente d'une planification strictement calée sur les minutes 00, 15, 30 et 45.

## 15. Vérifier et tester un timer

Pour vérifier la syntaxe d'un timer :

```bash
systemd-analyze calendar "*-*-* 02:00:00"
```

Cette commande permet de vérifier l'expression et d'afficher les prochaines occurrences.

Pour déclencher manuellement le service :

```bash
sudo systemctl start batch.service
```

Pour arrêter le timer :

```bash
sudo systemctl stop batch.timer
```

Pour désactiver son démarrage automatique :

```bash
sudo systemctl disable batch.timer
```

Pour désactiver et arrêter :

```bash
sudo systemctl disable --now batch.timer
```

Attention : arrêter le timer ne signifie pas nécessairement arrêter un service déjà lancé.

## 16. Cron vs Systemd Timers

Les deux mécanismes permettent d'automatiser des tâches, mais ils n'ont pas exactement les mêmes caractéristiques.

| Critère | Cron | Systemd Timers |
|---|---|---|
| Configuration | Crontab | Fichiers `.timer` et `.service` |
| Planification | Expressions temporelles | Calendrier et intervalles |
| Logs | Redirections ou courrier local | Intégration journald |
| Gestion des dépendances | Limitée | Intégration systemd |
| Utilisateur d'exécution | Crontab utilisateur ou système | Directive `User` |
| Environnement d'exécution | Environnement Cron | Environnement systemd |
| Exécution après arrêt du système | Pas de rattrapage natif | Possible avec `Persistent=true` pour les timers calendrier |
| Gestion des ressources | Limitée | Possibilités de contrôle systemd |
| Gestion des erreurs | À implémenter dans le script | Statut du service et journaux systemd |

### Quand utiliser Cron ?

Cron est intéressant lorsque :

- La tâche est simple.
- La planification est classique.
- On souhaite une configuration rapide.
- On travaille avec des scripts existants.

### Quand utiliser Systemd Timers ?

Systemd Timers est particulièrement intéressant lorsque :

- Le système utilise systemd.
- On veut gérer les tâches comme des services.
- On souhaite une intégration native avec journald.
- On veut contrôler l'utilisateur, les permissions et les ressources.
- On souhaite bénéficier de `Persistent=true`.

Il n'existe pas de règle universelle : le choix dépend du contexte et des besoins d'exploitation.

## 17. Bonnes pratiques pour les tâches planifiées

- Utiliser des chemins absolus.
- Définir explicitement les variables d'environnement nécessaires.
- Prévoir une gestion des erreurs.
- Journaliser les exécutions importantes.
- Éviter les exécutions simultanées non souhaitées.
- Vérifier les permissions de l'utilisateur exécutant la tâche.
- Tester les scripts manuellement avant de les planifier.
- Surveiller les échecs d'exécution.
- Prévoir des mécanismes de reprise pour les traitements critiques.
- Éviter de placer des secrets directement dans les fichiers de configuration.
- Vérifier le fuseau horaire du serveur.
- Documenter les tâches et leurs fréquences.

### Attention aux traitements batch

Pour un batch métier, la planification ne doit pas être confondue avec la gestion de son exécution.

Cron ou systemd déclenchent le traitement, mais ne garantissent pas à eux seuls :

- L'absence de doublons métier.
- La reprise après interruption.
- L'idempotence.
- La gestion des transactions.
- La traçabilité fonctionnelle.
- La cohérence des données.

Ces responsabilités doivent être traitées dans l'application ou dans l'orchestrateur de traitements.

## 18. Exercices pratiques

### Exercice 1 : premières tâches Cron

Créer une tâche qui :

1. Affiche la date toutes les minutes.
2. Enregistre le résultat dans un fichier.
3. S'exécute tous les jours à 2 heures.
4. S'exécute uniquement du lundi au vendredi à 8 heures.

### Exercice 2 : sauvegarde automatisée

Créer un script qui :

1. Archive un répertoire.
2. Ajoute la date au nom de l'archive.
3. Stocke le résultat dans un répertoire de sauvegarde.
4. Vérifie que l'archive a été créée.

Planifier son exécution quotidienne avec Cron.

### Exercice 3 : éviter les exécutions simultanées

Créer un script qui dure deux minutes.

Le planifier toutes les minutes avec Cron.

Observer les exécutions simultanées, puis utiliser `flock` pour les empêcher.

### Exercice 4 : premier Systemd Timer

Créer un service qui affiche la date dans les journaux systemd.

Créer un timer qui déclenche ce service toutes les cinq minutes.

Vérifier les prochaines exécutions avec :

```bash
systemctl list-timers
```

### Exercice 5 : batch Java

Créer un service systemd qui exécute une application Spring Batch.

Créer un timer qui lance le traitement chaque nuit à 2 heures.

Consulter les journaux avec :

```bash
journalctl -u batch.service
```

### Exercice 6 : comparaison

Mettre en place deux tâches équivalentes :

- Une avec Cron.
- Une avec Systemd Timer.

Comparer leur configuration, leurs logs, leur diagnostic et leur comportement après un arrêt du système.

## 19. Résumé des commandes essentielles

| Commande | Utilité |
|---|---|
| `crontab -l` | Lister les tâches utilisateur |
| `crontab -e` | Modifier les tâches utilisateur |
| `crontab -r` | Supprimer la crontab |
| `systemctl status cron` | Vérifier Cron |
| `systemctl status crond` | Vérifier Cron sur certaines distributions |
| `systemctl daemon-reload` | Recharger les unités systemd |
| `systemctl enable --now timer.timer` | Activer et démarrer un timer |
| `systemctl list-timers` | Lister les timers |
| `systemctl start service.service` | Déclencher manuellement un service |
| `systemctl stop timer.timer` | Arrêter un timer |
| `journalctl -u service.service` | Consulter les logs du service |
| `systemd-analyze calendar` | Vérifier une expression calendrier |
| `flock` | Empêcher certaines exécutions concurrentes |

## 20. Ressources officielles

- Cron : https://man7.org/linux/man-pages/man8/cron.8.html
- Crontab : https://man7.org/linux/man-pages/man5/crontab.5.html
- systemd.timer : https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html
- systemd.service : https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html
- systemd-analyze : https://www.freedesktop.org/software/systemd/man/latest/systemd-analyze.html
- flock : https://man7.org/linux/man-pages/man1/flock.1.html