# Sécurité Linux et durcissement des serveurs

## 1. Introduction

Un serveur Linux hébergeant une application doit être configuré pour limiter les risques d'intrusion, réduire les possibilités d'exploitation d'une vulnérabilité et protéger les données.

Le **durcissement (hardening)** consiste à appliquer un ensemble de mesures de sécurité afin de réduire la surface d'attaque d'un système.

L'objectif n'est pas de rendre un serveur invulnérable — ce qui est impossible — mais de limiter les risques et les conséquences d'une compromission.

Les principaux axes du durcissement sont :

- La gestion des utilisateurs et des privilèges.
- La sécurisation des accès administratifs.
- Le contrôle des connexions réseau.
- La gestion des mises à jour de sécurité.
- La protection des fichiers et des répertoires.
- La limitation des services exposés.
- La surveillance et l'audit du système.
- Le confinement des applications.

## 2. Comprendre la surface d'attaque

La surface d'attaque représente l'ensemble des points par lesquels un système peut être attaqué.

Sur un serveur Linux, elle comprend notamment :

- Les ports réseau ouverts.
- Les services accessibles.
- Les comptes utilisateurs.
- Les applications installées.
- Les vulnérabilités des paquets.
- Les permissions incorrectes.
- Les interfaces d'administration exposées.

### 2.1 Identifier les services actifs

```bash
systemctl list-units --type=service --state=running
```

Cette commande affiche les services actuellement actifs.

Pour identifier les ports en écoute :

```bash
sudo ss -tulpen
```

Options importantes :

- `-t` : connexions TCP.
- `-u` : connexions UDP.
- `-l` : sockets en écoute.
- `-p` : processus associé.
- `-e` : informations complémentaires.
- `-n` : affichage numérique des ports et adresses.

Exemple de résultat :

```text
Netid State  Local Address:Port
tcp   LISTEN 0.0.0.0:22
tcp   LISTEN 0.0.0.0:8080
tcp   LISTEN 127.0.0.1:5432
```

Interprétation :

- Le port `22` correspond généralement à SSH.
- Le port `8080` peut correspondre à une application web.
- Le port `5432` correspond généralement à PostgreSQL.
- PostgreSQL écoute ici uniquement sur l'interface locale.

**Bonne pratique :** ne laisser accessibles que les services réellement nécessaires.

### 2.2 Désactiver les services inutiles

Lorsqu'un service n'est pas nécessaire, il est préférable de le désactiver.

```bash
sudo systemctl stop nom-service
sudo systemctl disable nom-service
```

Pour arrêter le service et empêcher son démarrage automatique :

```bash
sudo systemctl disable --now nom-service
```

Attention : avant de désactiver un service, il faut vérifier ses dépendances et son utilité.

Pour empêcher également son démarrage manuel ou indirect :

```bash
sudo systemctl mask nom-service
```

Pour revenir en arrière :

```bash
sudo systemctl unmask nom-service
```

La commande `mask` est plus restrictive que `disable`.

## 3. Gestion des utilisateurs et des privilèges

L'une des règles fondamentales de sécurité consiste à appliquer le principe du **moindre privilège**.

Chaque utilisateur et chaque application doivent disposer uniquement des permissions nécessaires à leur fonctionnement.

### 3.1 Éviter l'utilisation directe de root

L'utilisateur `root` dispose de privilèges complets sur le système.

Il est préférable d'utiliser un compte nominatif et d'exécuter ponctuellement les commandes administratives avec `sudo`.

```bash
sudo apt update
sudo systemctl restart nginx
```

Pour vérifier les droits sudo :

```bash
sudo -l
```

Pour connaître les groupes d'un utilisateur :

```bash
groups utilisateur
```

### 3.2 Limiter les droits sudo

Les règles sudo sont généralement définies dans :

```text
/etc/sudoers
```

Il est fortement recommandé de modifier ce fichier avec :

```bash
sudo visudo
```

`visudo` vérifie la syntaxe avant de valider les modifications.

Exemple de règle :

```text
utilisateur ALL=(ALL:ALL) /usr/bin/systemctl restart nginx
```

Cette règle autorise l'utilisateur à redémarrer Nginx avec sudo, sans lui donner automatiquement tous les droits administratifs.

Attention : autoriser l'exécution d'une commande ne garantit pas qu'elle soit sans risque. Certaines commandes peuvent permettre de contourner les restrictions selon leurs arguments ou leur configuration.

### 3.3 Désactiver les comptes inutilisés

Pour identifier les comptes :

```bash
cat /etc/passwd
```

Pour verrouiller un compte :

```bash
sudo passwd -l utilisateur
```

Pour vérifier son état :

```bash
sudo passwd -S utilisateur
```

Le verrouillage du mot de passe ne désactive pas nécessairement toutes les méthodes d'authentification du compte. Il faut également vérifier les clés SSH, les services et les mécanismes d'authentification utilisés.

### 3.4 Créer un utilisateur dédié à une application

Une application ne devrait pas fonctionner avec le compte `root`.

Exemple pour une application Spring Boot :

```bash
sudo useradd \
  --system \
  --no-create-home \
  --shell /usr/sbin/nologin \
  springapp
```

Cette commande crée un compte système sans répertoire personnel et sans shell de connexion classique.

Vérification :

```bash
id springapp
```

On peut ensuite attribuer les fichiers de l'application à cet utilisateur :

```bash
sudo chown -R springapp:springapp /opt/myapp
```

Le service systemd pourra utiliser :

```ini
[Service]
User=springapp
Group=springapp
ExecStart=/usr/bin/java -jar /opt/myapp/application.jar
```

L'application s'exécutera ainsi avec les droits de `springapp`.

## 4. Sécuriser les permissions des fichiers

Les permissions Linux permettent de contrôler qui peut lire, modifier ou exécuter un fichier.

Quelques règles importantes :

- Éviter les permissions trop permissives.
- Ne pas utiliser systématiquement `chmod 777`.
- Protéger les fichiers de configuration sensibles.
- Limiter l'accès aux clés privées et aux secrets.
- Vérifier les permissions des répertoires contenant les données applicatives.

### 4.1 Protéger les fichiers sensibles

Exemple pour un fichier contenant des paramètres sensibles :

```bash
sudo chown root:root /etc/myapp/application.properties
sudo chmod 600 /etc/myapp/application.properties
```

Seul `root` pourra lire et modifier le fichier.

Si l'application doit le lire, il faut adapter les propriétaires et les groupes en fonction de l'architecture retenue, sans ouvrir inutilement les permissions.

Par exemple :

```bash
sudo chown root:springapp /etc/myapp/application.properties
sudo chmod 640 /etc/myapp/application.properties
```

Le propriétaire `root` peut lire et modifier le fichier. Le groupe `springapp` peut le lire.

### 4.2 Vérifier les permissions

```bash
ls -l /etc/myapp/
```

Pour inspecter les permissions de chaque niveau d'un chemin :

```bash
namei -l /etc/myapp/application.properties
```

Cette commande est particulièrement utile lorsqu'un utilisateur ne parvient pas à accéder à un fichier malgré des permissions correctes sur celui-ci : un répertoire parent peut bloquer l'accès.

## 5. Configurer un pare-feu

Un pare-feu permet de contrôler les communications réseau entrantes et sortantes.

Sur Linux, plusieurs solutions existent :

- UFW : interface simplifiée, notamment utilisée sur Ubuntu.
- firewalld : solution courante sur les distributions de la famille Red Hat.
- nftables : infrastructure moderne de filtrage réseau du noyau Linux.

### 5.1 Utiliser UFW

Vérifier son état :

```bash
sudo ufw status verbose
```

Installer UFW sur Ubuntu ou Debian si nécessaire :

```bash
sudo apt install ufw
```

Avant d'activer le pare-feu, définir les règles nécessaires.

Par exemple, autoriser SSH :

```bash
sudo ufw allow 22/tcp
```

Autoriser le trafic HTTP et HTTPS :

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Définir les politiques par défaut :

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Activer le pare-feu :

```bash
sudo ufw enable
```

Vérifier les règles :

```bash
sudo ufw status numbered
```

**Attention :** si tu travailles à distance en SSH, autorise et vérifie d'abord le port SSH avant d'activer UFW. Sinon, tu risques de perdre l'accès au serveur.

### 5.2 Limiter l'exposition des applications

Prenons un exemple avec une application Spring Boot sur le port `8080`.

Une mauvaise configuration consiste à exposer directement ce port à tout le réseau alors que l'application devrait uniquement être accessible par un reverse proxy.

Dans Spring Boot :

```properties
server.address=127.0.0.1
server.port=8080
```

L'application écoute uniquement sur l'interface locale.

Nginx peut ensuite servir de point d'entrée HTTP/HTTPS.

Dans une architecture distribuée, les règles doivent être adaptées aux communications entre services et aux contraintes de l'infrastructure.

**Important :** un pare-feu système ne remplace pas les règles réseau du cloud, du réseau d'entreprise ou de Kubernetes. Ces couches doivent être cohérentes.

## 6. Gestion des mises à jour de sécurité

Les mises à jour permettent de corriger des vulnérabilités et des bugs.

Un serveur non maintenu peut devenir une cible facile.

### 6.1 Mettre à jour Debian / Ubuntu

Actualiser les informations des dépôts :

```bash
sudo apt update
```

Mettre à jour les paquets :

```bash
sudo apt upgrade
```

Pour examiner les paquets pouvant être mis à jour :

```bash
apt list --upgradable
```

Pour identifier les paquets nécessitant un redémarrage :

```bash
test -f /var/run/reboot-required && cat /var/run/reboot-required
```

### 6.2 Mettre à jour Rocky Linux / RHEL / Fedora

```bash
sudo dnf check-update
sudo dnf upgrade
```

### 6.3 Automatiser les mises à jour de sécurité

Sur Debian et Ubuntu, le paquet `unattended-upgrades` permet d'automatiser certaines mises à jour.

Installation :

```bash
sudo apt install unattended-upgrades
```

Configuration :

```bash
sudo dpkg-reconfigure unattended-upgrades
```

Il faut toutefois définir une politique adaptée à l'environnement.

En production, les mises à jour doivent être maîtrisées :

- Tester les mises à jour en environnement de développement ou de recette.
- Prévoir une fenêtre de maintenance.
- Vérifier les dépendances applicatives.
- Prévoir une stratégie de retour arrière.
- Contrôler les redémarrages nécessaires.

Automatiser les mises à jour ne signifie pas supprimer le processus de validation.

## 7. Renforcer la sécurité avec AppArmor et SELinux

Linux propose des mécanismes de contrôle d'accès obligatoire, également appelés **Mandatory Access Control (MAC)**.

Contrairement aux permissions classiques, ces mécanismes peuvent limiter les actions d'un processus même lorsque celui-ci dispose de permissions Unix suffisantes.

Deux solutions importantes :

- AppArmor.
- SELinux.

### 7.1 AppArmor

AppArmor est notamment utilisé par défaut sur Ubuntu.

Il applique des profils de sécurité aux applications.

Vérifier son état :

```bash
sudo aa-status
```

Les profils peuvent fonctionner en deux modes :

- Enforce : les règles sont appliquées et les actions interdites sont bloquées.
- Complain : les violations sont journalisées, mais ne sont pas bloquées.

Afficher les profils :

```bash
ls /etc/apparmor.d/
```

AppArmor permet par exemple de limiter les fichiers auxquels un service peut accéder.

### 7.2 SELinux

SELinux est couramment utilisé sur les distributions Red Hat.

Vérifier son état :

```bash
getenforce
```

Les principaux modes sont :

- Enforcing : les règles sont appliquées.
- Permissive : les violations sont journalisées sans être bloquées.
- Disabled : SELinux est désactivé.

Afficher le contexte de sécurité :

```bash
ls -Z
```

Afficher les processus avec leurs contextes :

```bash
ps -eZ
```

**Bonne pratique :** ne pas désactiver AppArmor ou SELinux pour résoudre rapidement un problème applicatif. Il est préférable de comprendre la règle qui bloque l'accès et d'adapter le profil correctement.

## 8. Auditer et surveiller le système

Sécuriser un serveur ne consiste pas uniquement à appliquer une configuration initiale. Il faut aussi surveiller son fonctionnement et détecter les comportements anormaux.

### 8.1 Examiner les connexions et les utilisateurs

Utilisateurs actuellement connectés :

```bash
who
```

Historique des connexions :

```bash
last
```

Tentatives de connexion échouées :

```bash
sudo lastb
```

Cette dernière commande dépend de la disponibilité et de la configuration de `/var/log/btmp`.

### 8.2 Consulter les événements de sécurité

Sur un système utilisant systemd :

```bash
sudo journalctl -p warning
```

Pour consulter les événements liés à SSH :

```bash
sudo journalctl -u ssh
```

Sur certaines distributions, le service porte le nom `sshd` :

```bash
sudo journalctl -u sshd
```

Sur Debian et Ubuntu, certains événements d'authentification sont également disponibles dans :

```bash
sudo less /var/log/auth.log
```

Sur les systèmes Red Hat, consulter notamment :

```bash
sudo less /var/log/secure
```

Les chemins et la disponibilité des journaux dépendent de la distribution et de sa configuration.

### 8.3 Installer et utiliser auditd

`auditd` permet de collecter des événements d'audit du système Linux.

Installation sur Debian / Ubuntu :

```bash
sudo apt install auditd audispd-plugins
```

Vérifier le service :

```bash
sudo systemctl status auditd
```

Afficher les règles d'audit :

```bash
sudo auditctl -l
```

Exemple : surveiller les modifications d'un fichier sensible :

```bash
sudo auditctl -w /etc/passwd -p wa -k identity_changes
```

Explication :

- `-w` : chemin à surveiller.
- `-p wa` : surveiller les accès en écriture et les changements d'attributs.
- `-k` : associer une clé à l'événement.

Rechercher les événements :

```bash
sudo ausearch -k identity_changes
```

Cette règle est temporaire et peut disparaître au redémarrage. Pour une configuration persistante, les règles doivent être placées dans les fichiers prévus par la distribution, généralement sous `/etc/audit/rules.d/`.

## 9. Sécuriser les services systemd

Systemd propose plusieurs options permettant de réduire les privilèges et les capacités d'un service.

Prenons un exemple de service Spring Boot.

```ini
[Unit]
Description=Spring Boot Application
After=network.target

[Service]
Type=simple

User=springapp
Group=springapp

WorkingDirectory=/opt/myapp

ExecStart=/usr/bin/java -jar /opt/myapp/application.jar

Restart=on-failure
RestartSec=5

NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true

ReadWritePaths=/var/lib/myapp /var/log/myapp

[Install]
WantedBy=multi-user.target
```

### 9.1 Comprendre les directives de sécurité

| Directive | Rôle |
|---|---|
| `User` | Exécute le service avec un utilisateur dédié |
| `Group` | Définit le groupe du service |
| `NoNewPrivileges` | Empêche le processus d'acquérir de nouveaux privilèges |
| `PrivateTmp` | Fournit un espace `/tmp` privé au service |
| `ProtectSystem=strict` | Rend le système de fichiers en lecture seule pour le service, sauf exceptions |
| `ProtectHome` | Restreint l'accès aux répertoires personnels |
| `ReadWritePaths` | Autorise l'écriture dans des chemins explicitement définis |

Ces options sont des mécanismes de confinement supplémentaires. Elles ne remplacent pas une bonne gestion des permissions Unix.

### 9.2 Attention aux applications Java

Une application Spring Boot peut avoir besoin d'écrire :

- Des fichiers temporaires.
- Des fichiers d'export.
- Des fichiers de logs.
- Des fichiers de traitement batch.
- Des fichiers de configuration générés.

Avec `ProtectSystem=strict`, les chemins nécessaires doivent être explicitement autorisés.

Il faut donc tester les options de confinement avant leur déploiement en production.

Après modification du service :

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

Vérifier son état :

```bash
sudo systemctl status myapp
```

Consulter les logs :

```bash
sudo journalctl -u myapp -f
```

## 10. Gestion des secrets

Les applications ont souvent besoin de secrets :

- Mots de passe de bases de données.
- Clés API.
- Tokens.
- Certificats.
- Clés privées.

Il faut éviter de stocker ces informations directement dans le code source ou dans un dépôt Git.

### 10.1 Éviter les secrets dans les fichiers versionnés

Mauvaise pratique :

```properties
spring.datasource.password=MonMotDePasseSecret
```

dans un fichier `application.properties` versionné dans Git.

Préférer une variable d'environnement :

```properties
spring.datasource.password=${DB_PASSWORD}
```

Puis fournir la variable à l'application via un mécanisme adapté.

Exemple systemd :

```ini
[Service]
EnvironmentFile=/etc/myapp/myapp.env
```

Protéger le fichier :

```bash
sudo chown root:springapp /etc/myapp/myapp.env
sudo chmod 640 /etc/myapp/myapp.env
```

Attention : les variables d'environnement ne constituent pas à elles seules un coffre-fort. Elles peuvent être exposées par certains mécanismes de diagnostic ou à des processus suffisamment privilégiés.

Dans une infrastructure plus avancée, on peut utiliser un gestionnaire de secrets comme HashiCorp Vault ou les mécanismes de gestion des secrets de la plateforme de déploiement.

## 11. Sécuriser les applications dans Docker

La sécurité d'un conteneur dépend également de la configuration de son exécution.

Quelques bonnes pratiques :

- Ne pas exécuter les conteneurs en root lorsque ce n'est pas nécessaire.
- Utiliser des images de base minimales et maintenues.
- Mettre à jour régulièrement les images.
- Limiter les capacités Linux.
- Éviter le mode privilégié.
- Limiter les volumes montés.
- Ne pas intégrer les secrets dans l'image.
- Scanner les images pour détecter les vulnérabilités.

Exemple Docker Compose :

```yaml
services:
  api:
    image: my-api:1.0.0

    user: "10001:10001"

    read_only: true

    tmpfs:
      - /tmp

    cap_drop:
      - ALL

    security_opt:
      - no-new-privileges:true

    ports:
      - "127.0.0.1:8080:8080"
```

### 11.1 Explication des options

| Option | Rôle |
|---|---|
| `user` | Exécute le processus avec un UID/GID non privilégié |
| `read_only` | Rend le système de fichiers du conteneur en lecture seule |
| `tmpfs` | Fournit un espace temporaire en mémoire |
| `cap_drop` | Supprime les capacités Linux |
| `no-new-privileges` | Empêche l'acquisition de nouveaux privilèges |
| `127.0.0.1:8080:8080` | Expose le port uniquement sur l'interface locale de l'hôte |

Cette configuration est un exemple de départ, pas une configuration universelle. Une application peut nécessiter des capacités, des répertoires inscriptibles ou une exposition réseau différente.

## 12. Sécurité dans Kubernetes

Dans Kubernetes, le durcissement doit également être appliqué au niveau des Pods et des conteneurs.

Exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-api
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 10001
    runAsGroup: 10001
    seccompProfile:
      type: RuntimeDefault

  containers:
    - name: api
      image: my-api:1.0.0

      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
```

### 12.1 Principales protections

- `runAsNonRoot` : interdit l'exécution en tant que root.
- `runAsUser` : définit l'UID du processus.
- `runAsGroup` : définit son groupe principal.
- `allowPrivilegeEscalation: false` : interdit l'élévation de privilèges.
- `readOnlyRootFilesystem` : protège le système de fichiers racine du conteneur.
- `capabilities.drop: ALL` : supprime les capacités Linux non nécessaires.
- `seccompProfile` : applique un profil de filtrage des appels système.

Il faut également prendre en compte :

- Les RBAC pour contrôler les droits sur l'API Kubernetes.
- Les NetworkPolicies pour contrôler les communications réseau.
- Les Secrets pour stocker les informations sensibles.
- Les Pod Security Standards pour définir le niveau de sécurité des Pods.
- Les politiques d'admission pour empêcher le déploiement de workloads non conformes.

## 13. Checklist de durcissement d'un serveur Linux

Voici une checklist synthétique à utiliser lors de la préparation d'un serveur.

### Comptes et privilèges

- [ ] Utiliser des comptes nominatifs.
- [ ] Éviter les connexions directes en root.
- [ ] Appliquer le principe du moindre privilège.
- [ ] Limiter les droits sudo.
- [ ] Désactiver ou verrouiller les comptes inutilisés.
- [ ] Utiliser des comptes dédiés pour les applications.

### Réseau

- [ ] Identifier les ports ouverts.
- [ ] Désactiver les services inutiles.
- [ ] Activer un pare-feu.
- [ ] N'exposer que les ports nécessaires.
- [ ] Restreindre l'accès aux bases de données.
- [ ] Vérifier les interfaces d'écoute des applications.

### Système

- [ ] Appliquer régulièrement les mises à jour de sécurité.
- [ ] Protéger les fichiers sensibles.
- [ ] Vérifier les permissions des répertoires.
- [ ] Activer AppArmor ou SELinux lorsque disponible.
- [ ] Éviter les services inutiles.
- [ ] Contrôler les tâches planifiées.

### Surveillance

- [ ] Centraliser les journaux.
- [ ] Surveiller les connexions.
- [ ] Contrôler les événements de sécurité.
- [ ] Mettre en place des règles d'audit adaptées.
- [ ] Surveiller les ressources et les comportements anormaux.

### Applications et conteneurs

- [ ] Ne pas exécuter les applications en root.
- [ ] Protéger les secrets.
- [ ] Limiter les droits d'écriture.
- [ ] Appliquer les options de sécurité systemd.
- [ ] Scanner les images Docker.
- [ ] Appliquer les Security Contexts Kubernetes.

## 14. Méthodologie de durcissement

Pour appliquer ces principes dans un contexte professionnel, il est préférable de suivre une démarche structurée.

1. **Inventaire** : identifier les services, utilisateurs, ports et applications.
2. **Analyse des risques** : déterminer les éléments les plus exposés.
3. **Définition de la politique** : établir les règles de sécurité attendues.
4. **Application** : configurer les permissions, le pare-feu, les mises à jour et les mécanismes de confinement.
5. **Validation** : vérifier que les applications fonctionnent toujours.
6. **Audit** : contrôler les configurations et les journaux.
7. **Maintenance** : réévaluer régulièrement les règles et les vulnérabilités.

Un durcissement doit être testé avant son application en production. Une règle trop restrictive peut provoquer une indisponibilité applicative.

## 15. Exercices pratiques

### Exercice 1 — Audit initial

Sur une machine Linux :

- Afficher les utilisateurs connectés.
- Lister les services actifs.
- Identifier les ports en écoute.
- Identifier les comptes disposant de privilèges sudo.

Commandes utiles :

```bash
who
systemctl list-units --type=service --state=running
sudo ss -tulpen
sudo -l
```

### Exercice 2 — Créer un compte applicatif

Créer un utilisateur système `springapp` sans shell de connexion.

Vérifier :

- Son UID et son GID.
- Son shell.
- Ses permissions sur le répertoire de l'application.

### Exercice 3 — Configurer un pare-feu

Avec UFW :

- Autoriser SSH.
- Autoriser HTTP et HTTPS.
- Refuser les connexions entrantes par défaut.
- Vérifier les règles actives.

### Exercice 4 — Sécuriser un service Spring Boot

Créer un service systemd qui :

- S'exécute avec `springapp`.
- N'a pas besoin des privilèges root.
- Dispose uniquement des droits d'écriture nécessaires.
- Utilise `NoNewPrivileges`.
- Utilise `ProtectSystem`.

Vérifier que l'application fonctionne après l'activation de ces restrictions.

### Exercice 5 — Audit de sécurité

Installer `auditd`, surveiller `/etc/passwd` et rechercher les événements générés après une modification contrôlée.

## 16. Commandes à retenir

| Commande | Utilité |
|---|---|
| `ss -tulpen` | Identifier les ports en écoute |
| `systemctl list-units` | Lister les services actifs |
| `sudo -l` | Vérifier les privilèges sudo |
| `passwd -l` | Verrouiller un compte |
| `ufw status` | Vérifier le pare-feu UFW |
| `getenforce` | Vérifier le mode SELinux |
| `aa-status` | Vérifier AppArmor |
| `journalctl` | Consulter les journaux systemd |
| `last` | Consulter l'historique des connexions |
| `auditctl -l` | Lister les règles auditd |
| `ausearch` | Rechercher des événements d'audit |

## 17. Ressources officielles

- Linux man pages : https://man7.org/linux/man-pages/
- Ubuntu Security : https://ubuntu.com/security
- Debian Security : https://www.debian.org/security/
- Red Hat Security : https://access.redhat.com/security/
- AppArmor : https://apparmor.net/
- SELinux : https://selinuxproject.org/
- systemd : https://www.freedesktop.org/software/systemd/man/
- Docker Security : https://docs.docker.com/engine/security/
- Kubernetes Security : https://kubernetes.io/docs/concepts/security/
- OWASP : https://owasp.org/

---

**À retenir :** la sécurité Linux repose sur plusieurs couches complémentaires. La gestion des privilèges, le filtrage réseau, les mises à jour, le confinement des processus et la surveillance doivent fonctionner ensemble pour réduire la surface d'attaque d'un serveur.