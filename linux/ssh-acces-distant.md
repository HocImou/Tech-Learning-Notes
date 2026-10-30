# SSH et accès distant

## 1. Introduction

SSH (*Secure Shell*) est un protocole réseau qui permet de se connecter de manière sécurisée à une machine distante.

Il est notamment utilisé pour :

- Administrer des serveurs Linux à distance.
- Exécuter des commandes sur une machine distante.
- Transférer des fichiers entre deux machines.
- Automatiser des opérations d'administration.
- Accéder à des machines virtuelles ou à des serveurs hébergeant des applications.

SSH chiffre les communications entre le client et le serveur. Les identifiants, commandes et données échangées ne circulent donc pas en clair sur le réseau.

SSH utilise généralement le port TCP **22** par défaut.

## 2. Architecture SSH

SSH repose sur deux composants principaux :

- **SSH Client** : programme utilisé depuis la machine locale pour établir une connexion.
- **SSH Server** : service installé sur la machine distante qui accepte les connexions.

Sur Linux, l'implémentation la plus courante est OpenSSH.

### Fonctionnement

1. Le client SSH initie une connexion vers le serveur.
2. Le serveur présente son identité à travers sa clé d'hôte (*host key*).
3. Le client vérifie l'identité du serveur.
4. Le client et le serveur négocient les paramètres cryptographiques.
5. Le client s'authentifie auprès du serveur.
6. Une session sécurisée est établie.

Exemple :

```bash
ssh utilisateur@192.168.1.100
```

Cette commande demande une connexion SSH avec l'utilisateur `utilisateur` sur la machine distante `192.168.1.100`.

## 3. Installation et vérification d'OpenSSH

### Sur Debian / Ubuntu

Installer le serveur SSH :

```bash
sudo apt update
sudo apt install openssh-server
```

Installer le client SSH :

```bash
sudo apt install openssh-client
```

### Sur Fedora / RHEL

```bash
sudo dnf install openssh-server
sudo dnf install openssh-clients
```

### Vérifier le service SSH

```bash
sudo systemctl status ssh
```

Sur certaines distributions, le service porte le nom `sshd` :

```bash
sudo systemctl status sshd
```

Démarrer le service :

```bash
sudo systemctl start ssh
```

Activer son démarrage automatique :

```bash
sudo systemctl enable ssh
```

Ou combiner les deux opérations :

```bash
sudo systemctl enable --now ssh
```

Vérifier que le serveur écoute sur le port SSH :

```bash
sudo ss -tlnp | grep :22
```

**À retenir :** installer le client SSH permet de se connecter à d'autres machines. Installer le serveur SSH permet à d'autres machines de se connecter à la tienne.

## 4. Connexion à une machine distante

### Connexion standard

```bash
ssh utilisateur@serveur
```

Exemple :

```bash
ssh hocine@192.168.1.100
```

Si l'authentification par mot de passe est autorisée, SSH demande le mot de passe de l'utilisateur distant.

### Utiliser un port différent

Si le serveur SSH écoute sur un port différent de 22 :

```bash
ssh -p 2222 utilisateur@serveur
```

### Exécuter une commande distante

Il est possible d'exécuter une commande sans ouvrir une session interactive :

```bash
ssh utilisateur@serveur "hostname"
```

Autres exemples :

```bash
ssh utilisateur@serveur "uptime"
ssh utilisateur@serveur "df -h"
ssh utilisateur@serveur "systemctl status nginx"
```

Cette possibilité est particulièrement utile pour l'automatisation et les scripts d'administration.

## 5. Authentification SSH par clés

L'authentification par clé SSH est généralement préférable à l'authentification par mot de passe.

Elle repose sur une paire de clés cryptographiques :

- **Clé privée** : conservée exclusivement sur la machine cliente. Elle doit rester secrète.
- **Clé publique** : copiée sur le serveur distant dans le fichier `~/.ssh/authorized_keys`.

Le serveur vérifie que le client possède bien la clé privée correspondant à la clé publique autorisée.

### 5.1 Générer une paire de clés

La méthode recommandée est d'utiliser Ed25519 :

```bash
ssh-keygen -t ed25519 -C "mon-pc"
```

Le programme propose un emplacement :

```text
~/.ssh/id_ed25519
```

Il est également recommandé de définir une phrase secrète (*passphrase*) pour protéger la clé privée.

Les deux fichiers créés sont :

```text
~/.ssh/id_ed25519       # Clé privée
~/.ssh/id_ed25519.pub   # Clé publique
```

Ne partage jamais ta clé privée.

### 5.2 Copier la clé publique sur le serveur

La commande suivante permet d'ajouter la clé publique au serveur :

```bash
ssh-copy-id utilisateur@serveur
```

Pour un port spécifique :

```bash
ssh-copy-id -p 2222 utilisateur@serveur
```

Cette commande ajoute la clé publique au fichier :

```text
~/.ssh/authorized_keys
```

Si `ssh-copy-id` n'est pas disponible, il est possible d'ajouter manuellement le contenu de la clé publique dans ce fichier.

### 5.3 Se connecter avec la clé

```bash
ssh utilisateur@serveur
```

Si la clé est correctement installée, le serveur peut authentifier le client sans demander le mot de passe du compte distant.

Une passphrase peut toutefois être demandée pour déverrouiller la clé privée.

### 5.4 Utiliser ssh-agent

`ssh-agent` permet de conserver une clé déverrouillée en mémoire pendant une session.

Démarrer l'agent :

```bash
eval "$(ssh-agent -s)"
```

Ajouter la clé :

```bash
ssh-add ~/.ssh/id_ed25519
```

Lister les clés chargées :

```bash
ssh-add -l
```

Cela évite de saisir régulièrement la passphrase de la clé privée.

## 6. Gestion des clés et configuration SSH côté client

Le répertoire SSH de l'utilisateur est généralement :

```text
~/.ssh/
```

Il peut contenir :

| Fichier | Rôle |
|---|---|
| `id_ed25519` | Clé privée |
| `id_ed25519.pub` | Clé publique |
| `authorized_keys` | Clés publiques autorisées sur le serveur |
| `known_hosts` | Identités des serveurs déjà rencontrés |
| `config` | Configuration personnalisée du client SSH |

### 6.1 Permissions des fichiers

SSH est exigeant sur les permissions des fichiers contenant des clés.

Exemple :

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
```

La clé privée ne doit pas être accessible aux autres utilisateurs.

### 6.2 Configurer des alias SSH

Le fichier `~/.ssh/config` permet de définir des raccourcis pour les connexions.

Exemple :

```sshconfig
Host serveur-dev
    HostName 192.168.1.100
    User hocine
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host serveur-preprod
    HostName 192.168.1.101
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

Il devient possible de se connecter simplement avec :

```bash
ssh serveur-dev
```

Ou :

```bash
ssh serveur-preprod
```

Cette configuration évite de répéter les paramètres de connexion.

## 7. Vérification de l'identité du serveur : known_hosts

Lors de la première connexion à un serveur, SSH affiche généralement un message demandant de confirmer son empreinte cryptographique.

Exemple :

```text
The authenticity of host '192.168.1.100' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Cette vérification permet de s'assurer que l'on se connecte au bon serveur.

Après acceptation, l'identité du serveur est enregistrée dans :

```text
~/.ssh/known_hosts
```

### Attention aux changements de clé

Si la clé du serveur change, SSH peut afficher une alerte :

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Cela peut être dû à une réinstallation du serveur, mais aussi à une tentative d'interception de connexion (*Man-in-the-Middle*).

Il ne faut pas ignorer cette alerte sans vérifier l'empreinte du serveur auprès d'une source fiable.

Après vérification, une ancienne entrée peut être supprimée avec :

```bash
ssh-keygen -R 192.168.1.100
```

## 8. Configuration du serveur SSH

La configuration du serveur SSH se trouve généralement dans :

```text
/etc/ssh/sshd_config
```

Il est recommandé de sauvegarder le fichier avant toute modification :

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

### Paramètres importants

| Paramètre | Description |
|---|---|
| `Port` | Port d'écoute du serveur SSH |
| `PermitRootLogin` | Autorisation ou interdiction de connexion directe en root |
| `PasswordAuthentication` | Autorisation de l'authentification par mot de passe |
| `PubkeyAuthentication` | Autorisation de l'authentification par clé publique |
| `AllowUsers` | Liste des utilisateurs autorisés |
| `MaxAuthTries` | Nombre maximal de tentatives d'authentification |

Exemple de configuration :

```sshconfig
Port 22
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
MaxAuthTries 3
```

Cette configuration désactive l'authentification par mot de passe et interdit la connexion directe en root.

**Attention :** avant de désactiver l'authentification par mot de passe, il faut vérifier que l'authentification par clé fonctionne réellement. Conserve une session SSH ouverte pendant les changements afin d'éviter de perdre l'accès au serveur.

### Vérifier la configuration

```bash
sudo sshd -t
```

Cette commande vérifie la syntaxe de la configuration.

Afficher la configuration effective :

```bash
sudo sshd -T
```

Redémarrer ou recharger le service après modification :

```bash
sudo systemctl reload ssh
```

Selon la distribution :

```bash
sudo systemctl reload sshd
```

## 9. Transfert de fichiers avec SCP

`scp` permet de copier des fichiers entre une machine locale et une machine distante en utilisant SSH.

### Envoyer un fichier vers un serveur

```bash
scp fichier.txt utilisateur@serveur:/home/utilisateur/
```

### Récupérer un fichier depuis un serveur

```bash
scp utilisateur@serveur:/home/utilisateur/fichier.txt .
```

### Copier un répertoire

```bash
scp -r dossier/ utilisateur@serveur:/home/utilisateur/
```

### Utiliser un port spécifique

```bash
scp -P 2222 fichier.txt utilisateur@serveur:/tmp/
```

Attention : pour `scp`, l'option du port s'écrit `-P` en majuscule, contrairement à `ssh` qui utilise `-p`.

## 10. Transfert de fichiers avec SFTP

SFTP (*SSH File Transfer Protocol*) permet de transférer et de gérer des fichiers à distance via SSH.

Connexion :

```bash
sftp utilisateur@serveur
```

Commandes courantes :

```text
ls              # Liste les fichiers distants
pwd             # Répertoire distant courant
cd dossier      # Changer de répertoire distant
lls             # Liste les fichiers locaux
lpwd            # Répertoire local courant
put fichier.txt # Envoyer un fichier
get fichier.txt # Télécharger un fichier
mkdir dossier   # Créer un répertoire distant
rm fichier.txt  # Supprimer un fichier distant
exit            # Quitter
```

SFTP est particulièrement intéressant lorsqu'on souhaite manipuler des fichiers de manière interactive.

## 11. Transfert de fichiers avec Rsync

`rsync` permet de synchroniser des fichiers et des répertoires entre deux machines.

Il est souvent plus adapté que `scp` pour les synchronisations répétées, car il peut ne transférer que les différences.

Exemple :

```bash
rsync -avz dossier/ utilisateur@serveur:/home/utilisateur/dossier/
```

Options :

| Option | Description |
|---|---|
| `-a` | Mode archive : conserve les attributs principaux et parcourt les répertoires |
| `-v` | Affiche les détails |
| `-z` | Compresse les données pendant le transfert |

Utiliser un port SSH spécifique :

```bash
rsync -avz -e "ssh -p 2222" dossier/ utilisateur@serveur:/home/utilisateur/dossier/
```

Faire une simulation avant le transfert :

```bash
rsync -avzn dossier/ utilisateur@serveur:/home/utilisateur/dossier/
```

L'option `-n` permet de simuler les opérations sans modifier la destination.

Attention à la différence entre :

```bash
rsync -av dossier/ utilisateur@serveur:/destination/
```

et :

```bash
rsync -av dossier utilisateur@serveur:/destination/
```

La présence du `/` final sur la source change le résultat : dans le premier cas, on synchronise le contenu de `dossier`, dans le second, le répertoire lui-même.

## 12. Tunnel SSH et port forwarding

SSH ne sert pas uniquement à ouvrir un terminal distant. Il permet également de créer des tunnels réseau sécurisés.

### 12.1 Local port forwarding

Le *local port forwarding* permet d'accéder à un service distant à travers un tunnel SSH.

Exemple : une base PostgreSQL écoute sur le port 5432 d'un serveur distant.

```bash
ssh -L 5433:localhost:5432 utilisateur@serveur
```

Le tunnel établit le chemin suivant :

```text
Machine locale                 Serveur distant
--------------                 ---------------
localhost:5433  ------------>  localhost:5432
       |                              |
       +---------- Tunnel SSH --------+
```

Il est alors possible de configurer son client PostgreSQL avec :

```text
Host : localhost
Port : 5433
```

Les connexions au port local 5433 sont acheminées vers le port 5432 vu depuis le serveur SSH.

C'est particulièrement utile pour accéder à une base de données qui n'est pas directement exposée sur le réseau.

### 12.2 Remote port forwarding

Le *remote port forwarding* expose un port de la machine locale depuis le serveur distant.

Exemple :

```bash
ssh -R 8080:localhost:8080 utilisateur@serveur
```

Le serveur distant peut alors accéder au service local via son port 8080, sous réserve des règles de configuration SSH et réseau.

### 12.3 Dynamic port forwarding

SSH peut également créer un proxy SOCKS :

```bash
ssh -D 1080 utilisateur@serveur
```

Un logiciel configuré pour utiliser le proxy SOCKS local sur le port 1080 peut alors faire transiter ses connexions à travers le tunnel SSH.

## 13. Dépannage SSH

### Connexion refusée

Erreur :

```text
Connection refused
```

Causes possibles :

- Le serveur SSH n'est pas démarré.
- Le port SSH est incorrect.
- Le serveur n'écoute pas sur l'interface attendue.
- Un pare-feu bloque la connexion.

Vérifications :

```bash
sudo systemctl status ssh
sudo ss -tlnp | grep ssh
```

### Connexion expirée

Erreur :

```text
Connection timed out
```

Causes possibles :

- Serveur inaccessible.
- Problème réseau.
- Pare-feu ou filtrage réseau.
- Mauvaise adresse IP.

Vérifier la connectivité :

```bash
ping serveur
```

Attention : l'absence de réponse au ping ne prouve pas nécessairement que le serveur est indisponible, car ICMP peut être filtré.

### Permission denied

Erreur :

```text
Permission denied (publickey)
```

Causes possibles :

- Mauvais utilisateur.
- Mauvaise clé privée utilisée.
- Clé publique absente de `authorized_keys`.
- Permissions incorrectes.
- Authentification par clé désactivée côté serveur.

Lancer SSH en mode détaillé :

```bash
ssh -v utilisateur@serveur
```

Pour davantage de détails :

```bash
ssh -vvv utilisateur@serveur
```

Consulter les logs du serveur :

```bash
sudo journalctl -u ssh
```

Ou, selon la distribution :

```bash
sudo journalctl -u sshd
```

## 14. SSH dans un environnement Java et DevOps

SSH est très utilisé dans les environnements de développement et d'exploitation.

### 14.1 Déploiement d'une application Spring Boot

Dans un environnement simple, une application Spring Boot peut être transférée vers un serveur distant :

```bash
scp target/application.jar deploy@serveur:/opt/application/
```

Puis démarrée à distance :

```bash
ssh deploy@serveur "sudo systemctl restart application"
```

Dans un environnement professionnel, il est préférable d'automatiser ces opérations avec un outil de déploiement ou une pipeline CI/CD plutôt que de multiplier les connexions manuelles.

### 14.2 Git et SSH

Git utilise fréquemment SSH pour communiquer avec des dépôts distants.

Exemple :

```bash
git clone git@gitlab.com:organisation/projet.git
```

Ou avec GitHub :

```bash
git clone git@github.com:organisation/projet.git
```

Pour utiliser cette méthode, il faut ajouter sa clé publique SSH à son compte GitHub ou GitLab.

Tester la connexion à GitHub :

```bash
ssh -T git@github.com
```

Tester la connexion à GitLab :

```bash
ssh -T git@gitlab.com
```

### 14.3 SSH dans une pipeline CI/CD

Une pipeline GitLab CI/CD peut utiliser SSH pour se connecter à un serveur de déploiement.

Exemple simplifié :

```yaml
deploy:
  stage: deploy
  script:
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' > ~/.ssh/id_ed25519
    - chmod 600 ~/.ssh/id_ed25519
    - ssh-keyscan -H "$DEPLOY_HOST" >> ~/.ssh/known_hosts
    - scp target/application.jar "$DEPLOY_USER@$DEPLOY_HOST:/opt/application/"
    - ssh "$DEPLOY_USER@$DEPLOY_HOST" "sudo systemctl restart application"
```

Points de vigilance :

- La clé privée doit être stockée dans une variable CI/CD protégée et masquée, lorsque cela est possible.
- Ne jamais afficher la clé privée dans les logs.
- `ssh-keyscan` récupère une clé d'hôte, mais ne garantit pas à lui seul son authenticité. Il faut vérifier son empreinte par un canal fiable.
- Préférer une clé dédiée au déploiement, avec des droits limités.
- Éviter d'accorder des privilèges sudo trop larges à l'utilisateur de déploiement.

Dans une architecture Kubernetes avec GitOps et Argo CD, le déploiement des applications passe généralement par les mécanismes Kubernetes et GitOps. SSH n'est donc pas nécessairement le mécanisme de déploiement.

## 15. Bonnes pratiques de sécurité

- Utiliser de préférence l'authentification par clé.
- Protéger les clés privées avec des permissions strictes et une passphrase.
- Ne jamais partager une clé privée entre plusieurs personnes.
- Utiliser des clés dédiées pour les automatisations.
- Interdire la connexion directe en root lorsque cela est possible.
- Limiter les utilisateurs autorisés avec `AllowUsers` si le contexte le permet.
- Vérifier les empreintes des serveurs.
- Ne jamais ignorer une alerte de changement de clé sans vérification.
- Surveiller les logs d'authentification.
- Limiter les droits des comptes utilisés pour les déploiements.
- Ne pas considérer le simple changement du port 22 comme une mesure de sécurité suffisante.

## 16. Exercices pratiques

### Exercice 1 : installation et démarrage

1. Installer OpenSSH Server sur une VM Ubuntu.
2. Vérifier l'état du service.
3. Activer son démarrage automatique.
4. Vérifier que le port 22 est en écoute.

### Exercice 2 : connexion distante

1. Depuis une autre machine, se connecter à la VM.
2. Exécuter `hostname` à distance.
3. Exécuter `df -h` à distance.
4. Vérifier les informations de connexion avec `who`.

### Exercice 3 : authentification par clé

1. Générer une paire de clés Ed25519.
2. Installer la clé publique sur la VM.
3. Se connecter sans saisir le mot de passe du compte distant.
4. Vérifier les permissions des fichiers SSH.

### Exercice 4 : transfert de fichiers

1. Envoyer un fichier avec `scp`.
2. Le récupérer depuis le serveur.
3. Synchroniser un répertoire avec `rsync`.
4. Effectuer une simulation avec `rsync -n`.

### Exercice 5 : tunnel SSH

1. Installer PostgreSQL sur une VM.
2. Configurer PostgreSQL pour écouter sur son interface locale.
3. Créer un tunnel SSH local.
4. Se connecter à PostgreSQL depuis la machine cliente via le port local.

### Exercice 6 : automatisation

1. Créer un utilisateur dédié au déploiement.
2. Générer une clé SSH dédiée.
3. Configurer une connexion SSH par clé.
4. Simuler un transfert de fichier depuis une pipeline CI/CD.

## 17. Résumé des commandes essentielles

| Commande | Utilité |
|---|---|
| `ssh user@host` | Connexion distante |
| `ssh -p 2222 user@host` | Connexion sur un port spécifique |
| `ssh-keygen` | Génération de clés |
| `ssh-copy-id user@host` | Installation de la clé publique |
| `ssh-add` | Chargement d'une clé dans ssh-agent |
| `ssh -v user@host` | Diagnostic détaillé |
| `scp` | Copie de fichiers |
| `sftp` | Transfert interactif |
| `rsync` | Synchronisation de fichiers |
| `ssh -L` | Tunnel local |
| `ssh -R` | Tunnel distant |
| `ssh -D` | Proxy SOCKS |
| `ssh-keygen -R host` | Suppression d'une ancienne empreinte |
| `sshd -t` | Vérification de la configuration serveur |

## 18. Ressources officielles

- OpenSSH : https://www.openssh.com/
- OpenSSH Manual : https://man.openbsd.org/ssh
- SSH configuration : https://man.openbsd.org/ssh_config
- SSH Server configuration : https://man.openbsd.org/sshd_config
- Ubuntu - OpenSSH Server : https://ubuntu.com/server/docs/openssh-server
- GitHub - Connexion avec SSH : https://docs.github.com/en/authentication/connecting-to-github-with-ssh