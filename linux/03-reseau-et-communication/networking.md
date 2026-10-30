# Linux Networking — Fondamentaux (Partie 1)

## 1. Pourquoi le réseau Linux est fondamental

Comprendre le réseau Linux est indispensable pour travailler sur des applications backend et des infrastructures modernes.

Tout ce qui suit dépend du réseau :

- Applications backend (Spring Boot)
- Architectures microservices
- Docker
- Kubernetes
- Diagnostic en production : timeouts, services injoignables, erreurs de connexion…

**À retenir :** Docker et Kubernetes ne réinventent pas le réseau. Ils s'appuient notamment sur les mécanismes réseau de Linux pour permettre aux applications et aux services de communiquer.

## 2. Les briques fondamentales du réseau

Pour comprendre les communications réseau sous Linux, il faut commencer par quatre notions :

1. Interface réseau
2. Adresse IP
3. Port
4. Processus (application)

Ces éléments permettent de comprendre comment une communication arrive jusqu'à une application.

## 3. Interface réseau

Une interface réseau est un point d'entrée et de sortie permettant à une machine de communiquer avec un réseau.

Elle peut être physique ou virtuelle.

Exemples :

| Interface | Rôle |
|---|---|
| `eth0` | Interface Ethernet |
| `wlan0` | Interface Wi-Fi |
| `lo` | Interface loopback (`localhost`) |
| `docker0` | Bridge réseau créé par Docker |

### Afficher les interfaces

```bash
# Afficher les interfaces et leurs adresses IP
ip addr

# Afficher les interfaces et leur état
ip link

# Afficher uniquement les interfaces actives
ip link show up
```

Exemple simplifié :

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.20/24
```

Quelques informations importantes :

- `eth0` : nom de l'interface.
- `UP` : interface administrativement activée.
- `LOWER_UP` : liaison réseau opérationnelle au niveau du lien.
- `inet` : adresse IPv4 configurée.

**À retenir :**

- Une interface réseau permet les communications.
- Une interface peut être physique ou virtuelle.
- Une interface `UP` ne garantit pas à elle seule que la communication avec une autre machine fonctionne : l'adressage, le routage et les règles de filtrage doivent également être corrects.
- Une interface réseau est différente d'une application.

## 4. Adresse IP

Une adresse IP identifie une interface réseau dans un réseau donné.

Le masque ou préfixe CIDR permet de déterminer quelle partie de l'adresse correspond au réseau et quelle partie identifie l'interface dans ce réseau.

Exemple :

```text
172.17.0.2/16
```

Cela signifie que :

- `172.17.0.2` est l'adresse IP.
- `/16` est le préfixe réseau.
- Le réseau correspondant est `172.17.0.0/16`.

Cette notation est notamment courante dans les réseaux Docker.

### Plusieurs adresses IP sur une interface

Une même interface peut posséder plusieurs adresses IP.

Exemple :

```text
eth0:
  192.168.1.10/24
  192.168.1.20/24
  192.168.1.30/24
```

Cela peut être utile dans certaines architectures serveur pour héberger plusieurs services ou utiliser plusieurs adresses réseau.

Attention : ces adresses ne représentent pas automatiquement des environnements distincts (dev, recette, production). La séparation des environnements dépend de l'architecture réseau mise en place.

### Ajouter une adresse IP

```bash
# Ajouter une adresse IP supplémentaire à eth0
sudo ip addr add 192.168.1.30/24 dev eth0

# Vérifier la configuration
ip addr show dev eth0
```

**Attention :** cette modification avec `ip addr` est généralement temporaire. Elle peut disparaître après un redémarrage ou une reconfiguration réseau. Pour la rendre persistante, il faut utiliser le gestionnaire réseau adapté à la distribution Linux.

## 5. Port réseau

Un port permet d'identifier un point de communication associé à un service sur une adresse IP.

On peut retenir cette distinction :

| Élément | Rôle |
|---|---|
| Adresse IP | Identifie une interface réseau |
| Port | Identifie un point de communication |
| Protocole | Définit les règles de communication (TCP ou UDP) |

Exemples de ports courants :

| Port | Utilisation |
|---|---|
| `80` | HTTP |
| `443` | HTTPS |
| `8080` | Port fréquemment utilisé par Spring Boot |
| `5432` | PostgreSQL |
| `27017` | MongoDB |
| `9092` | Kafka, selon la configuration |

Un port n'est pas obligatoirement associé à une seule application pour toute la machine, mais deux processus ne peuvent généralement pas écouter simultanément sur la même combinaison d'adresse, de port et de protocole, sauf configurations particulières.

## 6. Processus et applications

Une application exécutée sous Linux fonctionne généralement sous la forme d'un ou plusieurs processus.

Exemples :

- Java : application Spring Boot
- Nginx : serveur web
- PostgreSQL : serveur de base de données
- Kafka : service de messagerie

Lorsqu'une application souhaite recevoir des communications réseau, elle ouvre une socket et se met à écouter sur une adresse IP et un port.

### Identifier les ports en écoute

```bash
# Afficher les sockets TCP et UDP en écoute
ss -tuln

# Afficher également les processus associés
sudo ss -tulnp

# Rechercher les sockets utilisant le port 8080
sudo ss -tulnp | grep 8080
```

Options principales :

| Option | Signification |
|---|---|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | Sockets en écoute |
| `-n` | Affichage numérique des adresses et ports |
| `-p` | Informations sur les processus |

## 7. Comment une requête arrive à une application ?

Prenons un exemple concret avec une application Spring Boot.

Un client exécute :

```bash
curl http://192.168.1.20:8080
```

Voici une représentation simplifiée du chemin suivi :

1. Le client détermine comment joindre l'adresse IP `192.168.1.20`, notamment à l'aide de sa table de routage.
2. Le paquet arrive sur la machine destinataire via une interface réseau.
3. Linux traite le paquet selon le protocole réseau et le protocole de transport (TCP dans cet exemple).
4. Le port de destination `8080` permet d'identifier la socket correspondant à l'écoute de l'application.
5. Le processus Java reçoit la communication et Spring Boot traite la requête HTTP.
6. La réponse repart vers le client.

**À retenir :**

- L'interface réseau permet de recevoir ou d'émettre les paquets.
- L'adresse IP identifie le point réseau destinataire.
- Le port permet d'identifier le point de communication.
- La socket relie la communication au processus qui l'a ouverte.
- L'application traite la requête et produit une réponse.

## 8. Exemple concret : Spring Boot et réseau Linux

Par défaut, Spring Boot utilise généralement le port `8080` pour son serveur web embarqué.

Dans `application.yml` :

```yaml
server:
  port: 8080
```

Si aucune adresse d'écoute spécifique n'est configurée, le serveur écoute généralement sur toutes les interfaces IPv4 disponibles.

Cela correspond à :

```text
0.0.0.0:8080
```

**Important :** `0.0.0.0` est une adresse d'écoute particulière. Ce n'est pas l'adresse que les clients doivent utiliser pour accéder à l'application.

### Restreindre l'écoute à une adresse IP

On peut demander à Spring Boot d'écouter sur une adresse IP précise :

```yaml
server:
  address: 192.168.1.20
  port: 8080
```

L'application écoute alors sur cette adresse, à condition qu'elle soit effectivement configurée sur la machine.

### Vérifier l'écoute de l'application

```bash
sudo ss -tulnp | grep 8080
```

Exemple de résultat :

```text
LISTEN  0  100  192.168.1.20:8080  0.0.0.0:*  users:(("java",pid=1234,fd=42))
```

On peut identifier :

- Le protocole TCP.
- L'adresse locale `192.168.1.20`.
- Le port `8080`.
- Le processus Java associé.

### Que se passe-t-il si l'interface réseau tombe ?

Si l'on désactive l'interface :

```bash
sudo ip link set eth0 down
```

L'application Java peut continuer à fonctionner, mais elle risque de devenir injoignable depuis le réseau concerné.

Cela illustre une distinction importante : **le fonctionnement du processus applicatif et la disponibilité réseau de l'application sont deux choses différentes.**

Attention : ne désactive pas une interface réseau sur une machine distante sans disposer d'un accès de secours. Tu risques de perdre ta connexion.

## 9. Schéma récapitulatif

```text
             [ Client ]
                  |
                  | Requête réseau
                  v
          [ Interface réseau ]
                  |
                  v
              [ Adresse IP ]
                  |
                  v
           [ Protocole / Port ]
                  |
                  v
               [ Socket ]
                  |
                  v
             [ Processus ]
                  |
                  v
          [ Application Spring Boot ]
```

Ce schéma est volontairement simplifié : il représente les principaux éléments à comprendre pour suivre une communication réseau jusqu'à une application.

## 10. Points essentiels à retenir

- Une interface réseau peut posséder plusieurs adresses IP.
- Une interface `UP` ne garantit pas à elle seule la connectivité.
- Une adresse IP identifie une interface dans un réseau.
- Un port permet d'identifier un point de communication.
- Une application ouvre une socket pour recevoir des communications.
- `0.0.0.0` signifie que l'application écoute sur toutes les interfaces IPv4.
- Linux gère le traitement réseau et le routage des paquets.
- Docker et Kubernetes s'appuient sur ces mécanismes.

## 11. Exercices pratiques

L'objectif est de manipuler les commandes et de comprendre les informations retournées.

**Exercice 1 — Interfaces réseau**

```bash
ip addr
ip link
```

Questions :
- Quelles interfaces sont présentes ?
- Lesquelles sont actives ?
- Quelles adresses IP sont configurées ?

**Exercice 2 — Ports en écoute**

```bash
ss -tuln
sudo ss -tulnp
```

Questions :
- Quels ports sont actuellement en écoute ?
- Quels processus sont associés à ces ports ?
- Ton application Spring Boot écoute-t-elle sur `8080` ?

**Exercice 3 — Tester une application**

```bash
curl http://localhost:8080
```

Puis :

```bash
curl http://127.0.0.1:8080
```

Questions :
- Quelle différence y a-t-il entre `localhost` et `127.0.0.1` ?
- L'application répond-elle ?
- Que se passe-t-il si elle n'est pas démarrée ?

**Exercice 4 — Comprendre les interfaces d'écoute**

Compare les résultats suivants :

```text
127.0.0.1:8080
0.0.0.0:8080
192.168.1.20:8080
```

Questions :
- Quelle configuration limite l'accès à la machine locale ?
- Quelle configuration écoute sur toutes les interfaces IPv4 ?
- Quelle configuration écoute uniquement sur une adresse précise ?

## 12. Pour aller plus loin

- [Documentation officielle Linux — ip](https://man7.org/linux/man-pages/man8/ip.8.html)
- [Documentation officielle Linux — ss](https://man7.org/linux/man-pages/man8/ss.8.html)
- [Documentation Spring Boot — Application Properties](https://docs.spring.io/spring-boot/appendix/application-properties/index.html)
- [Documentation Docker — Networking](https://docs.docker.com/engine/network/)