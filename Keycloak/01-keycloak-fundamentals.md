# 01 — Comprendre Keycloak et les fondamentaux de l'IAM

## 1. Introduction

Dans une application moderne, la gestion des utilisateurs et de leur authentification représente un enjeu important.

Imaginons une entreprise qui possède plusieurs applications :

- Une application de gestion des commandes.
- Une application de gestion des collaborateurs.
- Une API REST utilisée par plusieurs clients.
- Une application de reporting.
- Un portail destiné aux partenaires.

Chaque application pourrait gérer indépendamment ses utilisateurs, ses mots de passe, ses sessions et ses mécanismes de sécurité.

Cette approche présente rapidement plusieurs inconvénients :

- Les utilisateurs doivent posséder plusieurs comptes.
- Les mots de passe sont stockés et gérés à plusieurs endroits.
- Chaque application doit implémenter sa propre logique d'authentification.
- Les règles de sécurité deviennent difficiles à maintenir.
- La gestion des utilisateurs et des permissions devient complexe.

Une solution consiste à centraliser la gestion des identités et des authentifications dans un système spécialisé.

C'est précisément le rôle d'un système IAM (*Identity and Access Management*).

Keycloak est une solution open source qui permet de mettre en place ce type d'architecture.

## 2. Comprendre l'IAM

### 2.1. Qu'est-ce que l'IAM ?

IAM signifie *Identity and Access Management*, ou gestion des identités et des accès.

Il s'agit d'un ensemble de mécanismes et de processus permettant de gérer :

- Les identités des utilisateurs.
- Leur authentification.
- Leurs droits d'accès.
- Leurs rôles et permissions.
- Leurs sessions.
- Leur cycle de vie (création, modification, désactivation et suppression).

L'IAM répond principalement à deux questions :

1. Qui êtes-vous ?
2. Qu'avez-vous le droit de faire ?

Ces deux questions correspondent à deux notions fondamentales : l'authentification et l'autorisation.

### 2.2. Authentification : Authentication

L'authentification consiste à vérifier l'identité d'un utilisateur.

Exemple :

Un collaborateur souhaite accéder à une application de gestion des commandes.

Il fournit :
- Son identifiant.
- Son mot de passe.

Le système vérifie que ces informations correspondent à un utilisateur connu.

Si les informations sont correctes, l'utilisateur est authentifié.

D'autres mécanismes peuvent être utilisés :

- Mot de passe.
- Authentification multifacteur (MFA).
- Certificat numérique.
- Authentification biométrique.
- Fournisseur d'identité externe.

**À retenir : l'authentification permet de vérifier l'identité d'un utilisateur.**

### 2.3. Autorisation : Authorization

L'autorisation intervient après l'authentification.

Elle consiste à déterminer quelles ressources et quelles opérations sont accessibles à un utilisateur.

Prenons trois utilisateurs :

| Utilisateur | Rôle | Permissions |
|---|---|---|
| Alice | ADMIN | Consulter, créer, modifier et supprimer |
| Bob | MANAGER | Consulter, créer et modifier |
| Charlie | USER | Consulter uniquement |

Les trois utilisateurs peuvent être authentifiés, mais ils ne possèdent pas les mêmes droits.

**À retenir : l'autorisation détermine ce qu'un utilisateur authentifié a le droit de faire.**

### 2.4. Authentification et autorisation : ne pas confondre

| Authentification | Autorisation |
|---|---|
| Qui êtes-vous ? | Que pouvez-vous faire ? |
| Vérifie l'identité | Vérifie les permissions |
| Login et mot de passe | Rôles et permissions |
| Intervient généralement en premier | Intervient après l'identification |

Une erreur fréquente consiste à considérer qu'un utilisateur authentifié peut accéder à toutes les ressources.

Ce n'est pas le cas.

Une application doit toujours vérifier les autorisations nécessaires à chaque opération protégée.

## 3. Qu'est-ce que Keycloak ?

Keycloak est un serveur de gestion des identités et des accès développé comme projet open source.

Il permet notamment de :

- Centraliser l'authentification.
- Gérer les utilisateurs et les groupes.
- Gérer les rôles et les permissions.
- Fournir un mécanisme de Single Sign-On.
- Délivrer des tokens.
- S'intégrer avec des applications utilisant OAuth 2.0 et OpenID Connect.
- Se connecter à des fournisseurs d'identité externes.
- Mettre en place une authentification multifacteur.

Keycloak joue le rôle de serveur d'identité et, dans les architectures OAuth 2.0, de serveur d'autorisation.

Il permet ainsi de déléguer la gestion de l'authentification à un composant spécialisé.

### 3.1. Le principe de centralisation

Imaginons une architecture composée de trois applications :

- Application de gestion des commandes.
- Application de gestion des collaborateurs.
- API de gestion des données.

Sans Keycloak, chacune pourrait gérer ses propres utilisateurs et mots de passe.

Avec Keycloak, les applications délèguent l'authentification au même serveur.

L'utilisateur peut ainsi disposer d'une identité centralisée.

### 3.2. Architecture générale

Voici une représentation simplifiée de cette architecture :

```text
                     +----------------------+
                     |       Utilisateur    |
                     +----------+-----------+
                                |
                                |
                         Accède aux applications
                                |
                 +--------------+--------------+
                 |              |              |
                 v              v              v
          +-------------+ +-------------+ +-------------+
          | Application | | Application | | Application |
          | Commandes   | | Collabor.   | | Reporting   |
          +------+------+ +------+------+ +------+------+
                 |              |              |
                 +--------------+--------------+
                                |
                       Authentification
                                |
                                v
                    +-----------------------+
                    |       KEYCLOAK        |
                    |                       |
                    |  Gestion identités    |
                    |  Authentification     |
                    |  Sessions             |
                    |  Rôles et groupes     |
                    |  Tokens               |
                    +-----------------------+
                                |
                                v
                    +-----------------------+
                    | Utilisateurs /        |
                    | Fournisseurs externes |
                    +-----------------------+
```

Ce schéma représente le principe de centralisation. Les communications réelles dépendent du protocole utilisé et du type d'application.

## 4. Comprendre les principaux concepts de Keycloak

Avant de configurer Keycloak, il faut connaître son vocabulaire.

### 4.1. Realm

Un Realm représente un espace logique isolé dans Keycloak.

Il permet de regrouper et d'administrer :

- Des utilisateurs.
- Des groupes.
- Des rôles.
- Des clients.
- Des sessions.
- Des paramètres de sécurité.

Prenons une entreprise fictive qui possède deux domaines fonctionnels :

- Le domaine interne destiné aux collaborateurs.
- Le domaine partenaire destiné aux fournisseurs.

Elle peut créer deux Realms :

```text
Keycloak
   |
   +--- internal-realm
   |       |
   |       +--- Users
   |       +--- Groups
   |       +--- Roles
   |       +--- Clients
   |
   +--- partner-realm
           |
           +--- Users
           +--- Groups
           +--- Roles
           +--- Clients
```

Chaque Realm possède ses propres configurations et ses propres utilisateurs.

Attention : un Realm n'est pas simplement un groupe d'utilisateurs. C'est un véritable espace d'administration et de sécurité.

Il faut donc réfléchir à son découpage avant de multiplier les Realms.

### 4.2. User

Un User représente une identité connue de Keycloak.

Un utilisateur peut posséder plusieurs informations :

- Username.
- Email.
- Prénom et nom.
- Mot de passe.
- Attributs personnalisés.
- Rôles.
- Appartenance à des groupes.

Exemple :

```text
Utilisateur : alice
Email       : alice@example.com
Prénom      : Alice
Nom         : Martin
```

Keycloak permet également de gérer l'état du compte :

- Activé ou désactivé.
- Email vérifié ou non.
- Mot de passe temporaire.
- Actions requises à la prochaine connexion.

### 4.3. Client

Un Client représente une application ou un service qui interagit avec Keycloak.

Exemples :

- Application Angular.
- Application Spring Boot.
- API REST.
- Service backend.

Un client possède une configuration spécifique :

- Client ID.
- Type de client.
- Redirect URIs.
- Scopes.
- Protocol Mappers.
- Paramètres d'authentification.

Il existe notamment deux grandes catégories de clients :

**Client public :**

Il ne peut pas conserver de secret confidentiel de manière sûre.

Exemple : une application Angular exécutée dans le navigateur.

**Client confidentiel :**

Il peut conserver un secret côté serveur.

Exemple : une application Spring Boot exécutée sur un serveur maîtrisé.

Un client confidentiel doit protéger son secret et ne jamais l'exposer dans le navigateur ou dans un dépôt Git.

Nous approfondirons ces notions dans le document 03.

### 4.4. Groups

Les Groups permettent de regrouper les utilisateurs selon leur organisation ou leur fonction.

Exemple :

```text
Entreprise
   |
   +--- Direction
   |       +--- Alice
   |
   +--- Comptabilité
   |       +--- Bob
   |
   +--- Informatique
           +--- Charlie
```

Les groupes facilitent l'administration des utilisateurs et permettent notamment d'associer des rôles à plusieurs utilisateurs.

### 4.5. Roles

Les Roles permettent de représenter des responsabilités ou des droits associés aux utilisateurs ou aux applications.

Exemples :

- ADMIN.
- MANAGER.
- USER.
- REPORT_READER.

Keycloak distingue notamment :

**Realm Roles :**

Rôles définis au niveau du Realm.

**Client Roles :**

Rôles définis spécifiquement pour un client.

Cette distinction permet de représenter des permissions globales ou propres à une application.

Il faut toutefois retenir qu'un rôle déclaré dans Keycloak ne protège pas automatiquement une API : l'application doit interpréter et appliquer les autorisations correspondantes.

## 5. Comprendre le Single Sign-On (SSO)

### 5.1. Qu'est-ce que le SSO ?

SSO signifie *Single Sign-On*.

Le principe est simple : un utilisateur s'authentifie une seule fois auprès d'un fournisseur d'identité et peut ensuite accéder à plusieurs applications compatibles sans ressaisir systématiquement ses identifiants.

Exemple :

1. Alice ouvre l'application de gestion des commandes.
2. L'application redirige Alice vers Keycloak.
3. Alice saisit ses identifiants.
4. Keycloak authentifie Alice.
5. Alice accède à l'application.
6. Alice ouvre ensuite l'application de reporting.
7. Cette application redirige également vers Keycloak.
8. Keycloak reconnaît la session existante et peut authentifier Alice sans lui demander à nouveau son mot de passe.

### 5.2. Schéma simplifié

```text
        Alice
          |
          v
   Application A
          |
          v
       Keycloak
          |
     Authentification
          |
          v
   Session Keycloak
          |
          +-------------------+
          |                   |
          v                   v
   Application A       Application B
          |                   |
          +---------+---------+
                    |
             Session SSO
```

### 5.3. Attention : SSO ne signifie pas partage des sessions applicatives

Chaque application conserve son propre contexte de sécurité.

Keycloak centralise la session d'authentification, mais cela ne signifie pas que toutes les applications partagent automatiquement leurs cookies ou leurs sessions HTTP.

Il faut distinguer :

- La session SSO gérée par Keycloak.
- La session applicative gérée par chaque application.
- Les tokens utilisés pour accéder aux ressources.

Nous reviendrons sur ces distinctions dans les documents consacrés à OAuth2, OIDC et aux tokens.

## 6. Comprendre le rôle de Keycloak dans une architecture moderne

Pour comprendre l'intérêt de Keycloak, il faut distinguer les différents acteurs.

### 6.1. Identity Provider (IdP)

Un Identity Provider est un système qui authentifie les utilisateurs et fournit des informations sur leur identité.

Keycloak peut jouer ce rôle.

Il peut également déléguer l'authentification à un autre fournisseur d'identité.

Exemple :

Une entreprise souhaite permettre à ses collaborateurs de se connecter avec leurs comptes Microsoft.

Keycloak peut être configuré pour utiliser Microsoft comme fournisseur d'identité externe.

On parle alors d'Identity Brokering.

### 6.2. Client

Le client est l'application qui demande une authentification ou des tokens.

Exemples :

- Une application web.
- Une application mobile.
- Un backend Spring Boot.

### 6.3. Resource Server

Un Resource Server est un serveur qui expose des ressources protégées.

Exemple :

Une API REST Spring Boot qui expose :

```text
GET /api/orders
POST /api/orders
DELETE /api/orders/{id}
```

Cette API peut exiger un access token valide avant d'autoriser l'accès à ses ressources.

Keycloak délivre les tokens, mais c'est l'API qui doit vérifier les tokens et appliquer les règles d'accès.

### 6.4. Authorization Server

Dans le cadre d'OAuth 2.0, l'Authorization Server est le composant chargé de délivrer les tokens d'accès selon les règles du protocole.

Keycloak joue ce rôle lorsqu'il traite les demandes de tokens OAuth 2.0.

### 6.5. Récapitulatif des acteurs

| Acteur | Responsabilité |
|---|---|
| Resource Owner | Propriétaire des ressources, généralement l'utilisateur |
| Client | Application qui demande l'accès |
| Authorization Server | Délivre les tokens |
| Resource Server | Expose les ressources protégées |
| Identity Provider | Authentifie et fournit des informations sur l'identité |

Attention : les rôles Identity Provider et Authorization Server sont liés dans de nombreux scénarios, mais ils ne sont pas strictement synonymes.

## 7. Les protocoles utilisés par Keycloak

Keycloak s'appuie principalement sur des standards ouverts.

### 7.1. OAuth 2.0

OAuth 2.0 est un framework d'autorisation.

Il permet à une application d'obtenir un accès limité à des ressources protégées sans devoir récupérer directement le mot de passe de l'utilisateur.

Exemple :

Une application souhaite accéder à une API au nom d'un utilisateur.

Elle obtient un access token qui lui permet d'effectuer certaines opérations autorisées.

OAuth 2.0 traite principalement de délégation d'accès.

Il ne définit pas, à lui seul, un protocole standard complet d'authentification de l'utilisateur.

### 7.2. OpenID Connect (OIDC)

OpenID Connect est une couche d'identité construite au-dessus d'OAuth 2.0.

Il permet notamment de standardiser l'authentification et la transmission d'informations relatives à l'utilisateur.

OIDC introduit notamment :

- L'ID Token.
- Le scope `openid`.
- L'endpoint UserInfo.
- Des claims standardisés.
- Un mécanisme de découverte de configuration.

Il est particulièrement utilisé pour l'authentification des utilisateurs dans les applications web et mobiles.

### 7.3. SAML

SAML est un standard basé sur XML, utilisé pour l'échange d'informations d'authentification et d'autorisation entre des systèmes.

Il est notamment présent dans les architectures d'entreprise historiques et les systèmes de fédération d'identité.

Keycloak prend en charge SAML 2.0 en plus d'OIDC.

### 7.4. Comparaison

| Technologie | Objectif principal |
|---|---|
| OAuth 2.0 | Délégation d'autorisation |
| OpenID Connect | Authentification et informations d'identité |
| SAML 2.0 | Fédération d'identité et SSO |

Pour nos prochains exercices, nous utiliserons principalement OAuth 2.0 et OpenID Connect.

## 8. Comment se déroule une authentification avec Keycloak ?

Voici une première représentation simplifiée du processus.

Imaginons une application web qui utilise Keycloak pour authentifier ses utilisateurs.

```text
Utilisateur       Application       Keycloak
    |                  |                |
    | Accès application|                |
    |----------------->|                |
    |                  |                |
    |                  | Redirection    |
    |                  |--------------->|
    |                  |                |
    |                  | Page login     |
    |<----------------------------------|
    |                  |                |
    | Identifiants     |                |
    |---------------------------------->|
    |                  |                |
    |                  | Authentification
    |                  |                |
    |                  | Authorization |
    |                  | Code           |
    |                  |<---------------|
    |                  |                |
    |                  | Échange code   |
    |                  |--------------->|
    |                  |                |
    |                  | Access Token   |
    |                  | ID Token       |
    |                  |<---------------|
    |                  |                |
    | Accès autorisé   |                |
    |<-----------------|                |
```

Ce schéma est volontairement simplifié.

Dans une architecture réelle, les paramètres de sécurité, les cookies, les sessions, les redirections et les échanges HTTP doivent être pris en compte.

Nous étudierons en détail ce processus dans le document 05 consacré à Authorization Code et PKCE.

## 9. Ce que Keycloak ne fait pas automatiquement

Il est important de comprendre les limites du rôle de Keycloak.

Keycloak ne remplace pas :

- La logique métier des applications.
- Le contrôle d'accès métier.
- La validation des données.
- Les règles de gestion des ressources.
- La sécurisation de toutes les opérations métier.

Exemple :

Un utilisateur possède le rôle `MANAGER`.

Keycloak peut transmettre ce rôle dans un token.

Mais si l'application doit vérifier qu'un manager ne peut modifier que les commandes de son service, cette règle métier doit être appliquée par l'application ou par un système d'autorisation adapté.

Il ne faut donc pas confondre gestion des identités et gestion complète des règles métier.

## 10. Bonnes pratiques dès le départ

Quelques principes doivent guider toute implémentation :

1. Centraliser l'authentification lorsque l'architecture le justifie.
2. Utiliser des protocoles standards comme OIDC et OAuth 2.0.
3. Ne jamais exposer un secret client dans une application frontend.
4. Utiliser HTTPS pour les communications.
5. Définir clairement les responsabilités entre Keycloak et les applications.
6. Appliquer le principe du moindre privilège.
7. Ne pas confondre authentification et autorisation.
8. Ne pas considérer un JWT simplement décodé comme un token authentifié.
9. Séparer les configurations et les identités des différents environnements.
10. Prévoir une stratégie de gestion des utilisateurs, des sessions et des tokens.

## 11. Synthèse

Les notions essentielles à retenir :

| Notion | Définition |
|---|---|
| IAM | Gestion des identités et des accès |
| Authentication | Vérification de l'identité |
| Authorization | Vérification des droits |
| Keycloak | Solution de gestion des identités et des accès |
| Realm | Espace logique isolé dans Keycloak |
| User | Identité gérée par Keycloak |
| Client | Application ou service qui interagit avec Keycloak |
| Group | Regroupement d'utilisateurs |
| Role | Responsabilité ou droit associé à une identité ou une application |
| SSO | Authentification centralisée permettant l'accès à plusieurs applications |
| Identity Provider | Fournisseur d'identité |
| Authorization Server | Serveur chargé de délivrer les tokens |
| Resource Server | Serveur exposant des ressources protégées |
| OAuth 2.0 | Framework d'autorisation |
| OIDC | Couche d'identité construite sur OAuth 2.0 |
| SAML | Standard de fédération d'identité |

## 12. Exercices de compréhension

### Exercice 1 — Comprendre les responsabilités

Une entreprise possède trois applications :

- Une application de gestion des factures.
- Une application de gestion des collaborateurs.
- Une API REST utilisée par les deux applications.

Elle souhaite centraliser l'authentification.

Questions :

1. Quel est le rôle de Keycloak dans cette architecture ?
2. Les applications doivent-elles conserver les mots de passe des utilisateurs ?
3. Quel composant doit vérifier les tokens reçus par l'API ?
4. Keycloak doit-il contenir les règles métier concernant les factures ?
5. Quelle est la différence entre authentification et autorisation dans cet exemple ?

### Exercice 2 — Comprendre les concepts Keycloak

Une entreprise possède deux populations :

- Les collaborateurs internes.
- Les partenaires externes.

Elle souhaite créer des applications distinctes pour ces deux populations.

Questions :

1. Quel est le rôle d'un Realm ?
2. Faut-il obligatoirement créer deux Realms ?
3. Quelle est la différence entre un User et un Group ?
4. Quelle est la différence entre Realm Role et Client Role ?
5. Pourquoi faut-il réfléchir au découpage des Realms avant leur création ?

### Exercice 3 — Comprendre les protocoles

Questions :

1. Quelle est la différence fondamentale entre OAuth 2.0 et OpenID Connect ?
2. Quel est le rôle d'un access token ?
3. Quel est le rôle d'un ID Token ?
4. Pourquoi SAML est-il encore utilisé dans certaines entreprises ?
5. Pourquoi une API REST utilise-t-elle généralement un access token plutôt qu'un ID Token ?

## 13. Conclusion

Keycloak permet de centraliser la gestion des identités et des mécanismes d'authentification dans une architecture applicative.

Pour l'utiliser correctement, il faut comprendre les responsabilités de chaque composant et les protocoles sur lesquels il s'appuie.

La maîtrise de Keycloak ne consiste pas uniquement à savoir créer un utilisateur ou un client dans sa console d'administration.

Elle consiste surtout à comprendre :

- Comment une identité est authentifiée.
- Comment les droits sont représentés.
- Comment une application délègue son authentification.
- Comment les tokens sont délivrés et vérifiés.
- Comment les différentes applications collaborent dans un environnement sécurisé.

Dans le prochain document, nous approfondirons OAuth 2.0 et OpenID Connect afin de comprendre les mécanismes protocolaires qui permettent à Keycloak de communiquer avec les applications.