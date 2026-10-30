# Linux Networking — DNS & Résolution de noms (Partie 2)

## 1. Pourquoi le DNS est-il indispensable ?

Dans un réseau, les machines communiquent principalement à l'aide d'adresses IP.

Cependant, retenir des adresses IP pour accéder aux applications serait peu pratique.

Exemple :

```text
https://api.example.com
```

Derrière ce nom de domaine se trouve une ou plusieurs adresses IP.

C'est le rôle du **DNS (Domain Name System)** : permettre de traduire un nom de domaine en adresse IP.

Le DNS est utilisé quotidiennement dans de nombreux environnements :

- Applications backend (Spring Boot).
- Architectures microservices.
- Docker et Docker Compose.
- Kubernetes.
- Connexion aux bases de données.
- Communication entre serveurs.
- Accès aux services externes.

**À retenir :** le DNS permet aux applications de communiquer en utilisant des noms plutôt que des adresses IP.

## 2. Comment fonctionne le DNS ?

Le DNS repose sur un système hiérarchique et distribué.

Lorsqu'un utilisateur souhaite accéder à :

```text
https://api.example.com
```

Son système doit déterminer quelle adresse IP correspond à `api.example.com`.

De manière simplifiée, voici le processus :

1. L'application demande au système de résoudre le nom `api.example.com`.
2. Linux consulte ses mécanismes locaux de résolution.
3. Si aucune correspondance locale n'est trouvée, la requête est généralement transmise à un serveur DNS configuré.
4. Le serveur DNS recherche la réponse dans son cache ou interroge d'autres serveurs DNS.
5. Une adresse IP est retournée.
6. L'application peut ensuite établir une connexion vers cette adresse IP.

### Représentation simplifiée

```text
        Application
             |
             | api.example.com
             v
       Résolveur Linux
             |
             | Recherche locale
             v
       Serveur DNS configuré
             |
             | Recherche DNS
             v
        Adresse IP retournée
             |
             v
       Connexion au serveur
```

La résolution DNS et la connexion réseau sont deux opérations distinctes.

Une résolution DNS réussie ne garantit donc pas que le serveur est accessible : il faut également que le routage, le port, le pare-feu et le service soient opérationnels.

## 3. Les principaux types d'enregistrements DNS

Le DNS ne sert pas uniquement à trouver une adresse IP. Il permet de stocker différents types d'informations.

| Type | Rôle | Exemple |
|---|---|---|
| A | Associe un nom à une adresse IPv4 | `192.168.1.10` |
| AAAA | Associe un nom à une adresse IPv6 | `2001:db8::10` |
| CNAME | Définit un alias vers un autre nom DNS | `www.example.com` |
| MX | Indique les serveurs de messagerie | `mail.example.com` |
| NS | Indique les serveurs DNS faisant autorité | `ns1.example.com` |
| PTR | Permet une résolution inverse IP vers nom | `server.example.com` |
| TXT | Stocke des informations textuelles | SPF, vérifications de domaine |

### Exemple d'enregistrements

```text
api.example.com.    A       192.168.1.20
www.example.com.    CNAME   example.com.
example.com.        MX      mail.example.com.
```

### Le TTL : Time To Live

Chaque enregistrement DNS possède généralement un TTL.

Le TTL indique pendant combien de temps une réponse peut être conservée en cache avant de devoir être renouvelée.

Exemple :

```text
api.example.com.  300  IN  A  192.168.1.20
```

Ici, le TTL est de 300 secondes, soit 5 minutes.

**Pourquoi est-ce important ?**

Lorsqu'une adresse IP change, les anciens résultats peuvent rester temporairement présents dans les caches DNS.

Cela peut expliquer pourquoi un serveur fonctionne depuis une machine mais pas depuis une autre.

## 4. Comment Linux résout-il un nom de domaine ?

C'est une partie essentielle à comprendre pour diagnostiquer les problèmes DNS sous Linux.

Lorsqu'une application demande la résolution d'un nom, Linux utilise notamment le mécanisme NSS (Name Service Switch).

Ce mécanisme est configuré dans :

```text
/etc/nsswitch.conf
```

### Consulter la configuration

```bash
cat /etc/nsswitch.conf
```

Rechercher la ligne suivante :

```text
hosts: files dns
```

Cette configuration indique généralement :

1. `files` : consulter les fichiers locaux, notamment `/etc/hosts`.
2. `dns` : interroger le DNS si aucune correspondance n'a été trouvée précédemment.

L'ordre est important : une correspondance trouvée dans `/etc/hosts` peut empêcher l'interrogation du serveur DNS.

D'autres sources peuvent également être configurées, comme `myhostname`, `resolve` ou des modules NSS spécifiques.

### 4.1 Le fichier /etc/hosts

Le fichier `/etc/hosts` permet d'associer manuellement un nom à une adresse IP.

Exemple :

```bash
cat /etc/hosts
```

Contenu :

```text
127.0.0.1       localhost
192.168.1.20    api.local
192.168.1.30    database.local
```

Grâce à cette configuration :

```bash
ping api.local
```

Linux peut résoudre `api.local` vers `192.168.1.20` sans interroger un serveur DNS, si la configuration NSS consulte bien `files` avant `dns`.

### Ajouter une entrée

```bash
sudo nano /etc/hosts
```

Ajouter :

```text
192.168.1.50    mon-api.local
```

Tester :

```bash
getent hosts mon-api.local
```

Résultat attendu :

```text
192.168.1.50    mon-api.local
```

**Attention :** `/etc/hosts` est une configuration locale. Une modification sur une machine ne se propage pas automatiquement aux autres machines.

### 4.2 Le fichier /etc/resolv.conf

Ce fichier contient les paramètres utilisés par le résolveur DNS traditionnel de Linux.

Consulter son contenu :

```bash
cat /etc/resolv.conf
```

Exemple :

```text
nameserver 192.168.1.1
search example.local
```

Signification :

- `nameserver` : adresse IP d'un serveur DNS à interroger.
- `search` : suffixe de recherche utilisé pour compléter certains noms non qualifiés.

Par exemple, avec :

```text
search example.local
```

Une recherche sur :

```text
database
```

peut également être tentée sous la forme :

```text
database.example.local
```

Le comportement exact dépend notamment des paramètres du résolveur et du nom recherché.

**Important :** sur certaines distributions, `/etc/resolv.conf` est géré automatiquement par NetworkManager, systemd-resolved ou un autre gestionnaire réseau.

Il peut même s'agir d'un lien symbolique.

Vérifier :

```bash
ls -l /etc/resolv.conf
```

Ne modifie donc pas ce fichier sans comprendre quel composant le gère : tes changements risquent d'être écrasés.

### 4.3 systemd-resolved

Sur de nombreuses distributions Linux utilisant systemd, `systemd-resolved` peut gérer la résolution DNS.

Vérifier son état :

```bash
systemctl status systemd-resolved
```

Afficher les informations DNS :

```bash
resolvectl status
```

Consulter les statistiques :

```bash
resolvectl statistics
```

Dans certaines configurations, `/etc/resolv.conf` pointe vers un résolveur DNS local :

```text
nameserver 127.0.0.53
```

Cela signifie que les applications envoient leurs requêtes DNS à un service local, qui se charge ensuite de contacter les serveurs DNS configurés.

**À retenir :**

- `/etc/hosts` contient des correspondances locales.
- `/etc/resolv.conf` expose des paramètres de résolution DNS.
- `/etc/nsswitch.conf` définit les sources et leur ordre de consultation.
- `systemd-resolved` peut gérer les requêtes DNS et leur cache.

## 5. Les commandes essentielles pour diagnostiquer le DNS

Il existe plusieurs commandes permettant de tester la résolution DNS.

### 5.1 getent

`getent` permet de consulter les bases de données configurées via NSS.

C'est particulièrement intéressant car cette commande suit le mécanisme de résolution utilisé par de nombreuses applications Linux.

```bash
# Résoudre un nom
getent hosts google.com

# Résoudre un nom interne
getent hosts api.example.com
```

Exemple :

```text
142.250.74.14    google.com
```

Si `getent` ne retourne rien, cela signifie qu'aucune correspondance n'a été trouvée via les sources NSS configurées.

### 5.2 nslookup

`nslookup` permet d'interroger un serveur DNS pour obtenir des informations sur un nom de domaine.

```bash
nslookup google.com
```

Exemple de résultat simplifié :

```text
Server:    192.168.1.1
Address:   192.168.1.1#53

Name:      google.com
Address:   142.250.74.14
```

On peut également interroger un serveur DNS précis :

```bash
nslookup google.com 8.8.8.8
```

Ici, la requête est adressée au serveur DNS `8.8.8.8`.

Attention : `nslookup` interroge directement le DNS et ne reproduit pas nécessairement l'ensemble du mécanisme NSS utilisé par les applications.

### 5.3 dig

`dig` est un outil très utile pour analyser précisément les réponses DNS.

```bash
# Interroger un domaine
dig google.com

# Récupérer uniquement les adresses IPv4
dig google.com A +short

# Récupérer les adresses IPv6
dig google.com AAAA +short

# Interroger un serveur DNS spécifique
dig @8.8.8.8 google.com

# Interroger un enregistrement MX
dig example.com MX

# Interroger un enregistrement TXT
dig example.com TXT
```

Exemple :

```text
$ dig google.com A +short

142.250.74.14
```

Quelques informations intéressantes dans une réponse `dig` :

- `ANSWER SECTION` : contient les enregistrements retournés.
- `TTL` : durée de conservation en cache.
- `SERVER` : serveur DNS interrogé.
- `Query time` : durée de la requête DNS.

**Différence entre les commandes :**

| Commande | Utilisation |
|---|---|
| `getent` | Tester la résolution via NSS |
| `nslookup` | Interroger un serveur DNS |
| `dig` | Analyser en détail les réponses DNS |
| `resolvectl` | Consulter et diagnostiquer systemd-resolved |

### Installer les outils si nécessaire

Sur Debian ou Ubuntu :

```bash
sudo apt update
sudo apt install dnsutils
```

Sur certaines distributions récentes, `dig` et `nslookup` sont fournis par le paquet `bind9-dnsutils`.

Pour `getent`, l'outil est généralement déjà disponible via les utilitaires système.

## 6. DNS : résolution directe et résolution inverse

### Résolution directe

La résolution directe consiste à retrouver une adresse IP à partir d'un nom de domaine.

Exemple :

```text
api.example.com → 192.168.1.20
```

Commande :

```bash
dig api.example.com A
```

### Résolution inverse

La résolution inverse consiste à retrouver un nom associé à une adresse IP.

Exemple :

```text
192.168.1.20 → server.example.com
```

Elle utilise des enregistrements PTR.

Commande :

```bash
dig -x 192.168.1.20
```

La résolution inverse nécessite qu'un enregistrement PTR existe dans la zone DNS correspondante. Elle n'est pas automatiquement déduite d'un enregistrement A.

## 7. DNS et applications Spring Boot

Prenons un exemple d'architecture composée de deux applications :

- `api-service` : application Spring Boot.
- `database-service` : base de données PostgreSQL.

La configuration de l'application pourrait être :

```yaml
spring:
  datasource:
    url: jdbc:postgresql://database-service:5432/application
    username: application
    password: ${DB_PASSWORD}
```

Ici, Spring Boot utilise le nom :

```text
database-service
```

Pour établir une connexion, le système doit résoudre ce nom dans l'environnement réseau concerné.

Si la résolution échoue, l'application peut rencontrer une erreur du type :

```text
UnknownHostException: database-service
```

Cette erreur signifie que le nom d'hôte n'a pas pu être résolu par le mécanisme utilisé par l'application.

Cela ne signifie pas nécessairement que PostgreSQL est arrêté.

Il faut distinguer :

- Problème DNS : le nom ne peut pas être résolu.
- Problème réseau : l'adresse IP est connue, mais la machine est inaccessible.
- Problème de port : la machine est joignable, mais le port n'accepte pas la connexion.
- Problème applicatif : le serveur répond, mais l'application ou le service rencontre une erreur.

### Exemple de diagnostic

```bash
# Vérifier la résolution du nom
getent hosts database-service

# Vérifier la résolution DNS
dig database-service

# Tester la connexion au port PostgreSQL
nc -vz database-service 5432
```

La commande `nc` nécessite que Netcat soit installé.

**À retenir :** lorsqu'une application Spring Boot ne parvient pas à joindre une base de données, il faut vérifier séparément la résolution DNS, la connectivité réseau et la disponibilité du service.

## 8. DNS et Docker

Docker dispose de mécanismes de résolution DNS permettant aux conteneurs de communiquer entre eux.

Dans un réseau Docker personnalisé, les conteneurs peuvent généralement se résoudre mutuellement à partir de leur nom ou de leur alias réseau.

### Exemple avec Docker Compose

```yaml
services:
  api:
    image: mon-api:latest
    ports:
      - "8080:8080"
    depends_on:
      - database

  database:
    image: postgres:16
    environment:
      POSTGRES_DB: application
      POSTGRES_USER: application
      POSTGRES_PASSWORD: secret
```

Dans cet exemple, les deux services appartiennent au réseau créé par Docker Compose.

Depuis le conteneur `api`, le nom :

```text
database
```

peut être utilisé pour joindre le conteneur PostgreSQL.

La configuration Spring Boot devient :

```yaml
spring:
  datasource:
    url: jdbc:postgresql://database:5432/application
```

Docker fournit un DNS embarqué pour les réseaux personnalisés. Dans les conteneurs concernés, il est généralement accessible via :

```text
127.0.0.11
```

Cette adresse correspond au résolveur DNS embarqué de Docker dans ce contexte.

### Tester depuis un conteneur

```bash
# Afficher les conteneurs
docker ps

# Entrer dans le conteneur API
docker exec -it mon-api sh

# Tester la résolution
getent hosts database

# Vérifier la configuration DNS
cat /etc/resolv.conf
```

**Attention :**

- La résolution par nom de service fonctionne entre conteneurs partageant un réseau Docker approprié.
- Le nom `database` n'est pas automatiquement résolvable depuis la machine hôte.
- `depends_on` ne garantit pas que PostgreSQL soit prêt à accepter les connexions.

**À retenir :** dans Docker Compose, il est préférable d'utiliser les noms de services pour les communications entre conteneurs plutôt que de coder en dur leurs adresses IP.

## 9. DNS et Kubernetes

Dans Kubernetes, le DNS est un composant essentiel de la communication entre les services.

Kubernetes utilise généralement CoreDNS comme serveur DNS interne du cluster.

CoreDNS permet aux applications de résoudre les noms des Services Kubernetes.

### Exemple d'architecture

Imaginons deux composants :

- Un Service Kubernetes nommé `api-service`.
- Un Service Kubernetes nommé `database-service`.

Ils se trouvent dans le namespace `application`.

Une application peut accéder à la base de données avec :

```text
database-service
```

si elle se trouve dans le même namespace et que la configuration DNS du cluster est standard.

Depuis un autre namespace, on peut utiliser :

```text
database-service.application
```

Le nom pleinement qualifié est généralement :

```text
database-service.application.svc.cluster.local
```

Décomposition :

| Partie | Signification |
|---|---|
| `database-service` | Nom du Service |
| `application` | Namespace |
| `svc` | Domaine des Services Kubernetes |
| `cluster.local` | Domaine du cluster, configurable |

### Vérifier CoreDNS

```bash
kubectl get pods -n kube-system
```

Rechercher les pods CoreDNS :

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

Vérifier le Service DNS :

```bash
kubectl get svc -n kube-system kube-dns
```

### Tester la résolution depuis un Pod

```bash
kubectl exec -it mon-pod -- nslookup database-service.application.svc.cluster.local
```

On peut également vérifier la configuration DNS du Pod :

```bash
kubectl exec -it mon-pod -- cat /etc/resolv.conf
```

Exemple simplifié :

```text
search application.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

Les valeurs sont données à titre d'exemple : elles dépendent de la configuration du cluster.

### Comprendre ndots:5

Le paramètre :

```text
options ndots:5
```

indique que le résolveur considère un nom comme pleinement qualifié lorsqu'il contient au moins cinq points.

Pour un nom comportant moins de cinq points, le résolveur peut essayer les suffixes de recherche avant de tenter la résolution absolue.

Cela peut entraîner plusieurs requêtes DNS pour un même nom.

C'est un élément intéressant à connaître lorsqu'on analyse des problèmes de latence DNS dans Kubernetes.

**À retenir :**

- CoreDNS fournit la résolution DNS interne du cluster.
- Les Services Kubernetes possèdent des noms DNS.
- Les Pods utilisent ces noms pour communiquer entre eux.
- Les noms DNS permettent d'éviter de dépendre directement des adresses IP des Pods, qui peuvent changer.

## 10. Diagnostiquer un problème DNS

Prenons un problème concret :

Une application Spring Boot n'arrive plus à joindre :

```text
database-service
```

Voici une démarche de diagnostic progressive.

### Étape 1 — Vérifier la résolution locale

```bash
getent hosts database-service
```

Si aucune réponse n'est retournée, vérifier les sources NSS et les paramètres DNS.

### Étape 2 — Vérifier la configuration DNS

```bash
cat /etc/resolv.conf
```

Puis, si systemd-resolved est utilisé :

```bash
resolvectl status
```

### Étape 3 — Interroger directement le DNS

```bash
dig database-service
```

On peut également interroger explicitement le serveur DNS :

```bash
dig @192.168.1.1 database-service
```

L'adresse `192.168.1.1` est un exemple : il faut utiliser le serveur DNS réellement configuré.

### Étape 4 — Vérifier la connectivité réseau

Si le nom est résolu, récupérer son adresse IP puis tester la connectivité.

```bash
ping 192.168.1.20
```

Attention : un serveur peut bloquer les requêtes ICMP. L'absence de réponse au ping ne prouve donc pas qu'il est inaccessible.

### Étape 5 — Vérifier le port

```bash
nc -vz 192.168.1.20 5432
```

Cette commande permet de vérifier si une connexion TCP peut être établie sur le port PostgreSQL.

### Étape 6 — Vérifier le service

Si la résolution et la connexion réseau fonctionnent, il faut vérifier que le service est bien démarré et accepte les connexions.

Pour PostgreSQL :

```bash
sudo systemctl status postgresql
```

### Synthèse du diagnostic

| Symptôme | Piste à vérifier |
|---|---|
| `UnknownHostException` | Résolution DNS |
| `Connection timed out` | Routage, filtrage réseau ou service inaccessible |
| `Connection refused` | Port fermé ou service non démarré |
| Nom résolu vers une mauvaise IP | Configuration DNS ou cache |
| Résolution différente selon les machines | Configuration DNS, `/etc/hosts`, cache ou contexte réseau |

Ces messages sont des indices, pas des preuves absolues. Il faut toujours les confronter aux résultats des commandes de diagnostic.

## 11. Bonnes pratiques

Quelques bonnes pratiques à retenir pour les environnements de développement et de production :

- Éviter de coder en dur les adresses IP des services lorsque leurs noms DNS sont disponibles.
- Utiliser des noms de services stables dans Docker Compose et Kubernetes.
- Comprendre qui gère `/etc/resolv.conf` avant de le modifier.
- Utiliser `getent` pour tester la résolution telle qu'elle est configurée via NSS.
- Utiliser `dig` pour analyser les réponses DNS.
- Ne pas confondre résolution DNS et connectivité réseau.
- Tenir compte du TTL lorsqu'une adresse IP change.
- Ne pas utiliser `/etc/hosts` comme solution de découverte de services à grande échelle.
- Surveiller les erreurs et les temps de résolution DNS dans les environnements de production.

## 12. Exercices pratiques

L'objectif est de manipuler les commandes et de comprendre les mécanismes de résolution DNS.

### Exercice 1 — Comprendre la résolution locale

```bash
cat /etc/hosts
cat /etc/nsswitch.conf
getent hosts localhost
getent hosts google.com
```

Questions :

- Quelle est la différence entre les entrées locales et les entrées DNS ?
- Dans quel ordre Linux consulte-t-il les sources de résolution ?
- Pourquoi `getent` est-il intéressant pour tester la résolution ?

### Exercice 2 — Analyser les serveurs DNS

```bash
cat /etc/resolv.conf
resolvectl status
```

Questions :

- Quel serveur DNS est configuré ?
- Utilises-tu `systemd-resolved` ?
- Que signifie `127.0.0.53` si cette adresse apparaît dans `/etc/resolv.conf` ?

### Exercice 3 — Manipuler dig

```bash
dig google.com
dig google.com A +short
dig google.com AAAA +short
dig google.com MX
dig -x 8.8.8.8
```

Questions :

- Quelle différence entre les enregistrements A et AAAA ?
- À quoi sert un enregistrement MX ?
- Que représente le TTL ?
- Quelle différence entre résolution directe et inverse ?

### Exercice 4 — Simuler une entrée locale

Ajouter temporairement dans `/etc/hosts` :

```text
127.0.0.1 test.local
```

Puis exécuter :

```bash
getent hosts test.local
```

Questions :

- Pourquoi la résolution fonctionne-t-elle sans serveur DNS ?
- Est-ce que `dig test.local` donnera nécessairement le même résultat ?
- Pourquoi cette configuration n'est-elle pas adaptée à une architecture distribuée ?

Après l'exercice, supprimer l'entrée ajoutée dans `/etc/hosts`.

### Exercice 5 — Comprendre Docker

Avec un projet Docker Compose comportant une API et PostgreSQL :

```bash
docker compose up -d
docker compose ps
```

Depuis le conteneur API, tester :

```bash
getent hosts database
```

Questions :

- Pourquoi le nom `database` est-il résolu ?
- Pourquoi l'adresse IP du conteneur peut-elle changer ?
- Pourquoi est-il préférable d'utiliser le nom du service dans la configuration Spring Boot ?

### Exercice 6 — Comprendre Kubernetes

Depuis un Pod Kubernetes :

```bash
kubectl exec -it mon-pod -- cat /etc/resolv.conf
```

Puis :

```bash
kubectl exec -it mon-pod -- nslookup kubernetes.default.svc.cluster.local
```

Questions :

- Quel serveur DNS est utilisé par le Pod ?
- Quel est le rôle de CoreDNS ?
- Pourquoi les Services Kubernetes disposent-ils de noms DNS ?

## 13. Ressources complémentaires

- [Documentation Linux — nsswitch.conf](https://man7.org/linux/man-pages/man5/nsswitch.conf.5.html)
- [Documentation Linux — resolv.conf](https://man7.org/linux/man-pages/man5/resolv.conf.5.html)
- [Documentation Linux — hosts](https://man7.org/linux/man-pages/man5/hosts.5.html)
- [Documentation Linux — getent](https://man7.org/linux/man-pages/man1/getent.1.html)
- [Documentation systemd-resolved](https://www.freedesktop.org/software/systemd/man/latest/systemd-resolved.service.html)
- [Documentation Docker — Networking](https://docs.docker.com/engine/network/)
- [Documentation Kubernetes — DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Documentation CoreDNS](https://coredns.io/manual/toc/)