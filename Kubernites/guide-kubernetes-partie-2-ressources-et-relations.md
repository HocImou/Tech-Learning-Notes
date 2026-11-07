# Kubernetes --- Guide de référence : ressources, relations et fonctionnement

> **Objectif :** conserver une référence pratique sur les mécanismes
> Kubernetes étudiés avant de construire une application Spring Boot
> fictive.\
> **Périmètre :** Deployments, ReplicaSets, Pods, labels, selectors,
> Services, endpoints, CRD, Operators, RBAC et StatefulSets.\
> **Contexte :** exemples inspirés d'une API Spring Boot, avec des noms
> fictifs. Les commandes et manifestes sont à adapter au namespace et
> aux conventions de chaque cluster.

------------------------------------------------------------------------

## Sommaire

1.  [Comprendre les ressources et les
    contrôleurs](#1-comprendre-les-ressources-et-les-contrôleurs)
2.  [Labels, selectors et template](#2-labels-selectors-et-template)
3.  [Services, ports et endpoints](#3-services-ports-et-endpoints)
4.  [ConfigMap et référence à une
    clé](#4-configmap-et-référence-à-une-clé)
5.  [CRD et Operators](#5-crd-et-operators)
6.  [RBAC : gérer les permissions](#6-rbac--gérer-les-permissions)
7.  [Deployment ou StatefulSet](#7-deployment-ou-statefulset)
8.  [Commandes kubectl à retenir](#8-commandes-kubectl-à-retenir)
9.  [Méthode de diagnostic](#9-méthode-de-diagnostic)
10. [Résumé des notions](#10-résumé-des-notions)

------------------------------------------------------------------------

# 1. Comprendre les ressources et les contrôleurs

Kubernetes est un système déclaratif : on décrit l'état souhaité dans
des manifests YAML, puis les contrôleurs Kubernetes cherchent
continuellement à rapprocher l'état réel de cet état souhaité.

Un manifest Kubernetes contient généralement :

-   `apiVersion` : version de l'API à laquelle appartient la ressource ;
-   `kind` : type de ressource (`Deployment`, `Service`, `ConfigMap`,
    etc.) ;
-   `metadata` : nom, namespace, labels et annotations ;
-   `spec` : état souhaité et configuration ;
-   `status` : état observé par Kubernetes, généralement renseigné par
    les contrôleurs.

Exemple minimal :

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-commandes
  namespace: demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-commandes
  template:
    metadata:
      labels:
        app: api-commandes
    spec:
      containers:
        - name: api
          image: exemple/api-commandes:1.0
          ports:
            - containerPort: 8080
```

## 1.1 La chaîne Deployment → ReplicaSet → Pods

On crée généralement un Deployment plutôt que des Pods directement pour
une application stateless.

-   Le **Deployment** décrit le nombre de réplicas, le template des Pods
    et la stratégie de mise à jour.
-   Le **ReplicaSet**, créé et piloté par le Deployment, veille à
    maintenir le nombre demandé de Pods correspondant à son selector.
-   Les **Pods** hébergent les conteneurs de l'application.

``` mermaid
flowchart TD
    D["Deployment<br/>api-commandes<br/>replicas: 2"]
    R["ReplicaSet<br/>api-commandes-xxxxx"]
    P1["Pod 1<br/>app=api-commandes"]
    P2["Pod 2<br/>app=api-commandes"]
    D -->|"crée et pilote"| R
    R -->|"crée depuis le template"| P1
    R -->|"crée depuis le template"| P2
```

Si un Pod est supprimé ou tombe en panne, le ReplicaSet constate qu'il
manque un Pod correspondant et tente d'en créer un autre. Le nom et
l'adresse IP du nouveau Pod peuvent être différents.

**À retenir :** un Pod créé directement, sans contrôleur comme
Deployment, ReplicaSet, StatefulSet ou Job, n'est pas automatiquement
recréé si on le supprime.

## 1.2 Le rôle des contrôleurs

Un contrôleur compare en continu :

-   l'état souhaité, défini dans `spec` ;
-   l'état réel, observé dans le cluster.

Il effectue des actions pour réduire l'écart entre les deux. C'est le
principe de la boucle de réconciliation.

``` mermaid
flowchart LR
    S["État souhaité<br/>spec"]
    C["Contrôleur Kubernetes"]
    A["État réel<br/>cluster"]
    C -->|"observe"| A
    S --> C
    C -->|"corrige l'écart"| A
```

------------------------------------------------------------------------

# 2. Labels, selectors et template

Les labels et selectors sont au cœur de Kubernetes. Ils permettent aux
contrôleurs et aux Services de retrouver les ressources qu'ils doivent
gérer ou cibler.

## 2.1 Un label, c'est quoi ?

Un label est une paire clé/valeur associée aux métadonnées d'une
ressource.

Exemple :

``` yaml
metadata:
  labels:
    app: api-commandes
    environment: dev
    component: backend
```

Les labels peuvent servir à :

-   identifier une application ;
-   distinguer les environnements ;
-   identifier un composant (`backend`, `frontend`, `worker`) ;
-   filtrer des ressources avec `kubectl` ;
-   permettre aux contrôleurs et Services de sélectionner les Pods.

Les labels ne sont pas des noms uniques : plusieurs Pods peuvent porter
exactement les mêmes labels.

## 2.2 Un selector, c'est quoi ?

Un selector est un critère de sélection. Par exemple :

``` yaml
selector:
  matchLabels:
    app: api-commandes
```

Il signifie : « sélectionner les ressources qui possèdent le label
`app=api-commandes` ».

Avec plusieurs critères :

``` yaml
selector:
  matchLabels:
    app: api-commandes
    environment: dev
```

Les deux conditions doivent être satisfaites.

## 2.3 Pourquoi le selector et le template doivent-ils correspondre ?

Dans un Deployment :

``` yaml
spec:
  selector:
    matchLabels:
      app: api-commandes

  template:
    metadata:
      labels:
        app: api-commandes
```

Les deux blocs ont des rôles différents :

-   `spec.selector.matchLabels` indique au Deployment/ReplicaSet quels
    Pods appartiennent à son ensemble ;
-   `spec.template.metadata.labels` définit les labels que recevront les
    Pods créés à partir du template.

Le ReplicaSet crée les Pods à partir du template et utilise son selector
pour identifier les Pods qu'il doit maintenir.

Dans un Deployment `apps/v1`, le selector est obligatoire, doit
correspondre aux labels du template et est immuable après création.

``` mermaid
flowchart TD
    D["Deployment"]
    SEL["selector<br/>app=api-commandes"]
    T["template.metadata.labels<br/>app=api-commandes"]
    R["ReplicaSet"]
    P1["Pod A<br/>label app=api-commandes"]
    P2["Pod B<br/>label app=api-commandes"]
    D --> SEL
    D --> T
    D -->|"crée/pilote"| R
    T -->|"labels copiés lors de la création"| P1
    T -->|"labels copiés lors de la création"| P2
    SEL -.->|"sélectionne les Pods correspondants"| P1
    SEL -.->|"sélectionne les Pods correspondants"| P2
```

### Que se passe-t-il si les labels ne correspondent pas ?

Exemple invalide :

``` yaml
selector:
  matchLabels:
    app: api-commandes

template:
  metadata:
    labels:
      app: autre-application
```

Kubernetes refuse normalement la création du Deployment car son selector
ne correspond pas aux labels du template.

Si un Pod géré par un ReplicaSet perd ensuite le label attendu, il ne
correspond plus au selector. Le ReplicaSet peut alors créer un autre Pod
pour retrouver le nombre demandé.

Éviter de modifier manuellement les labels des Pods gérés par un
Deployment : il vaut mieux modifier le manifest source et laisser le
contrôleur réconcilier l'état.

## 2.4 Vérifier les labels et selectors

``` bash
# Afficher les Pods et leurs labels
kubectl get pods --show-labels -n demo

# Afficher les Pods correspondant à un label
kubectl get pods -l app=api-commandes -n demo

# Afficher les labels d'un Pod
kubectl get pod NOM_DU_POD --show-labels -n demo

# Afficher le selector et les détails du Deployment
kubectl describe deployment api-commandes -n demo

# Afficher les ReplicaSets
kubectl get rs -n demo
```

Exemple de filtrage avec plusieurs labels :

``` bash
kubectl get pods -l app=api-commandes,environment=dev -n demo
```

------------------------------------------------------------------------

# 3. Services, ports et endpoints

Un Pod est éphémère : il peut être recréé avec une nouvelle adresse IP.
Il est donc déconseillé de configurer les clients pour appeler
directement l'IP d'un Pod.

Un **Service** fournit un point d'accès réseau stable et sélectionne les
Pods qui doivent recevoir le trafic.

## 3.1 Exemple de Service

Supposons que Spring Boot écoute sur le port `8083` :

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: api-commandes
  namespace: demo
spec:
  type: ClusterIP
  selector:
    app: api-commandes
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8083
```

Le selector du Service sélectionne les Pods qui possèdent
`app=api-commandes`.

Le Service ne contient pas une liste d'IP écrite manuellement :
Kubernetes découvre et maintient les destinations à partir du selector
et de l'état des Pods.

## 3.2 `port` et `targetPort`

Dans cet exemple :

-   `port: 80` : port exposé par le Service ;
-   `targetPort: 8083` : port du conteneur vers lequel le trafic est
    transmis ;
-   `protocol: TCP` : protocole réseau utilisé.

Chemin simplifié :

``` mermaid
flowchart TD
    C["Client dans le cluster"]
    S["Service api-commandes<br/>ClusterIP : port 80"]
    P1["Pod 1<br/>10.0.1.10:8083"]
    P2["Pod 2<br/>10.0.1.11:8083"]
    C -->|"api-commandes:80"| S
    S -->|"targetPort 8083"| P1
    S -->|"targetPort 8083"| P2
```

`containerPort: 8083` dans la définition du conteneur est une
déclaration descriptive du port. À lui seul, il n'ouvre pas le port et
ne crée pas le Service.

## 3.3 Que signifie endpoint ?

Dans ce contexte, un endpoint représente une destination réseau
utilisable par le Service, généralement une adresse IP de Pod et un
port.

Exemple conceptuel :

``` text
10.0.1.10:8083
10.0.1.11:8083
```

Kubernetes maintient les ressources **EndpointSlice** qui représentent
les destinations associées au Service. Un Service avec un selector est
suivi par les contrôleurs Kubernetes ; les EndpointSlices sont
actualisées lorsque les Pods sélectionnés ou leur état changent.

Un Pod non Ready n'est normalement pas considéré comme une destination
prête à recevoir le trafic standard du Service.

``` mermaid
flowchart TD
    S["Service<br/>selector: app=api-commandes"]
    E["EndpointSlices<br/>destinations réseau"]
    P1["Pod 1<br/>Ready"]
    P2["Pod 2<br/>Ready"]
    P3["Pod 3<br/>non Ready"]
    S -->|"selector"| E
    E --> P1
    E --> P2
    P3 -.->|"pas une destination prête standard"| E
```

## 3.4 Commandes de diagnostic d'un Service

``` bash
# Services du namespace
kubectl get svc -n demo

# Détails du Service
kubectl describe svc api-commandes -n demo

# EndpointSlices
kubectl get endpointslices -n demo

# YAML des EndpointSlices
kubectl get endpointslices -n demo -o yaml

# Vérifier les labels des Pods
kubectl get pods --show-labels -n demo
```

La commande historique suivante reste utile, mais EndpointSlices est la
représentation moderne à privilégier :

``` bash
kubectl get endpoints api-commandes -n demo
```

### Service créé, mais aucun endpoint ?

Vérifier dans cet ordre :

1.  Le selector du Service.
2.  Les labels réellement présents sur les Pods.
3.  L'état `Ready` des Pods.
4.  Le `targetPort` et le port réellement écouté par l'application.
5.  Les événements et les logs si les Pods ne deviennent pas Ready.

Un Service peut exister sans disposer de destinations prêtes.

------------------------------------------------------------------------

# 4. ConfigMap et référence à une clé

Une ConfigMap permet de stocker des données de configuration non
sensibles. Un Secret est destiné aux données sensibles, même si un
Secret Kubernetes n'est pas automatiquement chiffré au repos : cela
dépend notamment de la configuration du cluster.

## 4.1 Injecter une ConfigMap entière

Exemple de ConfigMap :

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
  namespace: demo
data:
  LOG_LEVEL: INFO
  FEATURE_ENABLED: "true"
```

Injection de toutes les entrées comme variables d'environnement :

``` yaml
containers:
  - name: api
    image: exemple/api:1.0
    envFrom:
      - configMapRef:
          name: api-config
```

## 4.2 Récupérer une seule clé avec `configMapKeyRef`

Il est aussi possible de récupérer une seule entrée de la ConfigMap :

``` yaml
containers:
  - name: api
    image: exemple/api:1.0
    env:
      - name: LOG_LEVEL
        valueFrom:
          configMapKeyRef:
            name: api-config
            key: LOG_LEVEL
```

Explication :

-   `env` : définit une variable d'environnement du conteneur ;
-   `name: LOG_LEVEL` : nom de la variable vue par l'application ;
-   `valueFrom` : la valeur provient d'une autre ressource ;
-   `configMapKeyRef` : la valeur provient d'une clé d'une ConfigMap ;
-   `name: api-config` : nom de la ConfigMap ;
-   `key: LOG_LEVEL` : clé précise à récupérer.

La variable d'environnement `LOG_LEVEL` du conteneur reçoit donc la
valeur `INFO`.

Pour un Secret, le mécanisme équivalent est `secretKeyRef` :

``` yaml
env:
  - name: CLIENT_SECRET
    valueFrom:
      secretKeyRef:
        name: client-secret
        key: client_secret
```

Attention à ne pas confondre le champ YAML `data` d'un objet Secret avec
une clé appelée `data` dans `secretKeyRef.key`. `key` désigne le nom
d'une entrée à l'intérieur du Secret.

------------------------------------------------------------------------

# 5. CRD et Operators

## 5.1 Qu'est-ce qu'une CRD ?

CRD signifie **Custom Resource Definition** (définition de ressource
personnalisée).

Kubernetes possède des types natifs comme `Pod`, `Deployment`,
`Service`, `ConfigMap` ou `Secret`.

Une CRD permet d'ajouter un nouveau type de ressource à l'API
Kubernetes. Par exemple, External Secrets Operator installe des CRD
permettant de déclarer des ressources telles que `ExternalSecret` et
`SecretStore`.

Exemple simplifié :

``` yaml
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: application-secret
  namespace: demo
spec:
  refreshInterval: 30s
  secretStoreRef:
    name: vault-demo
    kind: SecretStore
  target:
    name: application-secret
  data:
    - secretKey: client_secret
      remoteRef:
        key: chemin/du/secret/dans/vault
        property: client_secret
```

La CRD apprend à Kubernetes à reconnaître et valider le nouveau type de
ressource. Elle ne récupère pas elle-même le secret.

## 5.2 Différence entre CRD et Operator

-   La **CRD** définit le type de ressource et son schéma.
-   L'**Operator** est un programme qui surveille les objets de ce type
    et exécute la logique correspondante.

Exemple de synchronisation avec External Secrets Operator :

``` mermaid
flowchart TD
    U["Manifest ExternalSecret"]
    K["API Kubernetes<br/>CRD ExternalSecret"]
    O["External Secrets Operator<br/>contrôleur"]
    V["Coffre Vault"]
    S["Secret Kubernetes"]
    U --> K
    K -->|"objet observé"| O
    O -->|"récupère la valeur"| V
    V -->|"retourne les propriétés"| O
    O -->|"crée ou actualise"| S
```

Dans cet exemple, l'Operator doit lui-même disposer des autorisations
nécessaires pour lire les ressources concernées et créer ou actualiser
les Secrets. L'application consommatrice n'a pas automatiquement ces
permissions.

## 5.3 Commandes utiles pour les CRD

``` bash
# Lister les CRD installées
kubectl get crd

# Rechercher les CRD External Secrets
kubectl get crd | grep external-secrets

# Décrire une CRD
kubectl describe crd externalsecrets.external-secrets.io

# Lister les ExternalSecrets
kubectl get externalsecrets -n demo

# Décrire un ExternalSecret
kubectl describe externalsecret application-secret -n demo

# Découvrir le schéma et les champs disponibles
kubectl explain externalsecret
kubectl explain externalsecret.spec
kubectl explain externalsecret.spec.target
```

Une ressource personnalisée existe dans un cluster parce que sa CRD y a
été installée. Si la CRD n'existe pas, l'API Kubernetes ne reconnaît
généralement pas ce `kind`.

------------------------------------------------------------------------

# 6. RBAC : gérer les permissions

RBAC signifie **Role-Based Access Control**. Il permet de contrôler qui
peut effectuer quelles actions sur quelles ressources Kubernetes.

Il faut distinguer :

-   l'identité qui effectue l'action ;
-   la ressource ciblée ;
-   les actions autorisées ;
-   le périmètre (namespace ou cluster).

## 6.1 Les ressources RBAC principales

  -----------------------------------------------------------------------
  Ressource                           Fonction
  ----------------------------------- -----------------------------------
  `ServiceAccount`                    Identité utilisable par un Pod ou
                                      un processus dans le cluster

  `Role`                              Définit des permissions dans un
                                      namespace

  `RoleBinding`                       Associe un Role ou un ClusterRole à
                                      un sujet dans un namespace

  `ClusterRole`                       Définit des permissions au niveau
                                      cluster ou réutilisables

  `ClusterRoleBinding`                Associe un ClusterRole à un sujet à
                                      l'échelle du cluster
  -----------------------------------------------------------------------

Un `RoleBinding` peut associer un `ClusterRole` à un sujet dans un
namespace : les permissions ainsi accordées restent limitées à ce
namespace pour les ressources namespacées.

## 6.2 Exemple : autoriser un Pod à lire un Secret précis

### Étape 1 --- ServiceAccount

``` yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-sa
  namespace: demo
```

### Étape 2 --- Role

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: demo
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
    resourceNames: ["application-secret"]
```

Explications :

-   `apiGroups: [""]` : les Secrets appartiennent au groupe API core ;
-   `resources: ["secrets"]` : la règle cible les Secrets ;
-   `verbs: ["get"]` : elle autorise la lecture d'une ressource précise
    ;
-   `resourceNames` : elle limite l'accès au Secret
    `application-secret`.

### Étape 3 --- RoleBinding

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: api-secret-reader
  namespace: demo
subjects:
  - kind: ServiceAccount
    name: api-sa
    namespace: demo
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

Le RoleBinding associe le ServiceAccount au Role.

### Étape 4 --- Utiliser le ServiceAccount dans le Deployment

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-commandes
  namespace: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api-commandes
  template:
    metadata:
      labels:
        app: api-commandes
    spec:
      serviceAccountName: api-sa
      containers:
        - name: api
          image: exemple/api-commandes:1.0
```

Le Pod utilise maintenant l'identité `api-sa`.

### Schéma du fonctionnement

``` mermaid
flowchart TD
    P["Pod api-commandes"]
    SA["ServiceAccount<br/>api-sa"]
    RB["RoleBinding<br/>api-secret-reader"]
    R["Role<br/>secret-reader"]
    SEC["Secret<br/>application-secret"]
    P -->|"utilise"| SA
    SA -->|"identité associée par"| RB
    RB -->|"accorde les règles de"| R
    R -->|"autorise get sur"| SEC
```

## 6.3 Tester les permissions

``` bash
# L'identité peut-elle lire le Secret autorisé ?
kubectl auth can-i get secret/application-secret \
  --namespace demo \
  --as=system:serviceaccount:demo:api-sa

# Peut-elle lire un autre Secret ?
kubectl auth can-i get secret/autre-secret \
  --namespace demo \
  --as=system:serviceaccount:demo:api-sa
```

Résultat attendu avec les règles de l'exemple :

``` text
yes
no
```

La simulation avec `--as` dépend aussi des permissions de l'utilisateur
qui lance la commande.

## 6.4 Verbes RBAC fréquents

  Verbe      Signification
  ---------- --------------------------------------
  `get`      Lire une ressource précise
  `list`     Lister des ressources
  `watch`    Observer les changements
  `create`   Créer une ressource
  `update`   Remplacer/modifier une ressource
  `patch`    Modifier partiellement une ressource
  `delete`   Supprimer une ressource

N'accorder que les permissions nécessaires : c'est le principe du
**moindre privilège**.

## 6.5 Cas important : faut-il du RBAC pour consommer un Secret ?

**Non, pas lorsque Kubernetes injecte le Secret dans le conteneur.**

Exemple :

``` yaml
env:
  - name: CLIENT_SECRET
    valueFrom:
      secretKeyRef:
        name: application-secret
        key: client_secret
```

Kubernetes fournit la valeur au conteneur. L'application n'appelle pas
l'API Kubernetes pour récupérer le Secret : elle n'a donc pas besoin
d'un droit RBAC `get secrets` pour ce seul usage.

En revanche, si le code Java utilise directement l'API Kubernetes pour
lire des Secrets, il faut définir des permissions RBAC adaptées.

L'Operator qui synchronise les secrets possède ses propres permissions,
indépendantes de celles de l'application.

------------------------------------------------------------------------

# 7. Deployment ou StatefulSet

Le choix dépend principalement de la nature des instances de
l'application.

-   Un **Deployment** convient aux Pods interchangeables, généralement
    stateless.
-   Un **StatefulSet** convient aux Pods qui nécessitent une identité
    stable et, souvent, un stockage persistant individuel.

## 7.1 Deployment : Pods interchangeables

Exemple : une API Spring Boot stateless avec trois réplicas.

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-commandes
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-commandes
  template:
    metadata:
      labels:
        app: api-commandes
    spec:
      containers:
        - name: api
          image: exemple/api-commandes:1.0
          ports:
            - containerPort: 8080
```

Pods possibles :

``` text
api-commandes-abc
api-commandes-def
api-commandes-ghi
```

Si un Pod disparaît, le Deployment/ReplicaSet le remplace. Le nouveau
Pod peut avoir un autre nom et une autre IP. Cela convient si
l'application ne dépend pas de l'identité individuelle de chaque Pod.

Exemples d'utilisation :

-   API REST Spring Boot ;
-   frontend Angular servi par Nginx ;
-   microservices stateless ;
-   workers dont l'état durable est stocké ailleurs.

## 7.2 StatefulSet : identité stable

Un StatefulSet crée des Pods avec des identités ordinales stables :

``` text
postgres-0
postgres-1
postgres-2
```

Si `postgres-1` est recréé, il conserve ce nom. Son stockage associé
peut également être conservé.

Le StatefulSet permet notamment :

-   des identités réseau stables ;
-   des noms prévisibles ;
-   un ordre de création et de suppression ;
-   un volume persistant associé à chaque Pod avec
    `volumeClaimTemplates`.

## 7.3 Exemple pédagogique : PostgreSQL StatefulSet

Cet exemple illustre le fonctionnement du StatefulSet, mais **n'est pas
une configuration PostgreSQL prête pour la production**.

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - name: postgres
      port: 5432
      targetPort: 5432
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless
  replicas: 2
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:17
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: postgres-data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: postgres-data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 5Gi
```

### A. Service Headless

``` yaml
clusterIP: None
```

Un Service Headless ne fournit pas de ClusterIP virtuelle. Il permet
notamment la découverte DNS individuelle des Pods associés.

Le StatefulSet référence ce Service :

``` yaml
serviceName: postgres-headless
```

Les noms DNS peuvent prendre cette forme :

``` text
postgres-0.postgres-headless.demo.svc.cluster.local
postgres-1.postgres-headless.demo.svc.cluster.local
```

Le Service Headless ne configure pas à lui seul la réplication
PostgreSQL : il fournit un mécanisme de découverte réseau.

### B. Identité et ordre des Pods

Avec `replicas: 2`, les Pods sont :

``` text
postgres-0
postgres-1
```

Par défaut, la politique `OrderedReady` crée les Pods dans l'ordre et
attend que le précédent soit Ready avant de poursuivre.

Il existe aussi une politique `Parallel` qui permet une gestion
parallèle de la création et de la suppression, lorsque le système le
permet.

### C. Stockage individuel

`volumeClaimTemplates` permet de créer un PVC par Pod :

``` text
postgres-data-postgres-0
postgres-data-postgres-1
```

Chaque Pod dispose ainsi de son propre volume persistant.

``` mermaid
flowchart TD
    STS["StatefulSet postgres"]
    P0["Pod postgres-0"]
    P1["Pod postgres-1"]
    V0["PVC postgres-data-postgres-0"]
    V1["PVC postgres-data-postgres-1"]
    PV0["Volume persistant 0"]
    PV1["Volume persistant 1"]
    STS --> P0
    STS --> P1
    P0 --> V0 --> PV0
    P1 --> V1 --> PV1
```

Si un Pod est recréé, le PVC correspondant est normalement conservé et
peut être remonté sur le nouveau Pod. La suppression d'un StatefulSet ne
supprime pas automatiquement tous ses PVC. Le cycle de vie et la
politique de rétention doivent néanmoins être vérifiés et configurés
selon le besoin.

Le stockage nécessite une StorageClass ou une autre configuration de
provisionnement compatible avec le cluster.

## 7.4 Cas d'utilisation des StatefulSets

  -----------------------------------------------------------------------
  Système                             Pourquoi un StatefulSet peut être
                                      utile
  ----------------------------------- -----------------------------------
  PostgreSQL                          Identité et stockage propres à
                                      chaque instance

  Kafka                               Identité stable des brokers et
                                      stockage des données

  Elasticsearch                       Identité et stockage des nœuds de
                                      données

  Redis                               Certaines architectures distribuées
                                      avec identité stable et persistance
  -----------------------------------------------------------------------

Des opérateurs spécialisés sont souvent utilisés pour gérer la
configuration, la réplication, les mises à jour et les opérations
complexes de ces systèmes.

**Important :** un StatefulSet ne crée pas automatiquement un cluster
distribué ou une réplication de base de données. Il gère l'identité,
l'ordre et le stockage des Pods. La réplication, l'élection du primaire,
la cohérence, les sauvegardes et la restauration relèvent de la base de
données ou d'un Operator spécialisé.

## 7.5 Comparatif

  ------------------------------------------------------------------------
  Critère                 Deployment              StatefulSet
  ----------------------- ----------------------- ------------------------
  Identité des Pods       Interchangeable         Stable et ordonnée

  Nom des Pods            Suffixe généré          Ordinal (`app-0`,
                                                  `app-1`)

  Stockage individuel     Possible, non           Via
                          automatique             `volumeClaimTemplates`

  Ordre de création       Pas de garantie d'ordre Ordonné par défaut

  Usage courant           API, frontend,          Bases de données,
                          microservices           systèmes distribués
  ------------------------------------------------------------------------

Un Deployment peut aussi utiliser des volumes. Le StatefulSet n'est pas
réservé exclusivement aux bases de données. Le critère principal est le
besoin d'identité stable et de stockage associé à chaque instance.

------------------------------------------------------------------------

# 8. Commandes kubectl à retenir

Remplacer `demo` par le namespace utilisé.

## 8.1 Observer les ressources

``` bash
# Pods
kubectl get pods -n demo

# Pods avec labels
kubectl get pods --show-labels -n demo

# Deployments
kubectl get deployments -n demo

# ReplicaSets
kubectl get rs -n demo

# Services
kubectl get svc -n demo

# EndpointSlices
kubectl get endpointslices -n demo

# ConfigMaps et Secrets (ne pas afficher les valeurs sensibles)
kubectl get configmaps -n demo
kubectl get secrets -n demo

# StatefulSets et PVC
kubectl get statefulsets -n demo
kubectl get pvc -n demo
```

## 8.2 Examiner une ressource

``` bash
# Description détaillée
kubectl describe deployment api-commandes -n demo
kubectl describe pod NOM_DU_POD -n demo
kubectl describe svc api-commandes -n demo

# YAML de la ressource réellement présente dans le cluster
kubectl get deployment api-commandes -n demo -o yaml
kubectl get pod NOM_DU_POD -n demo -o yaml
```

`kubectl get -o yaml` montre l'objet enregistré dans le cluster, avec
ses champs et valeurs effectifs. Ce n'est pas nécessairement le fichier
source du dépôt Git ni le rendu Helm d'origine.

## 8.3 Examiner les logs et événements

``` bash
# Logs d'un Pod
kubectl logs NOM_DU_POD -n demo

# Logs d'un conteneur précis
kubectl logs NOM_DU_POD -c api -n demo

# Logs précédents si le conteneur a redémarré
kubectl logs NOM_DU_POD --previous -n demo

# Événements récents
kubectl get events -n demo --sort-by=.metadata.creationTimestamp
```

## 8.4 Vérifier RBAC

``` bash
# Tester une permission
kubectl auth can-i get secrets -n demo

# Tester une permission pour un ServiceAccount
kubectl auth can-i get secret/application-secret \
  -n demo \
  --as=system:serviceaccount:demo:api-sa
```

## 8.5 Découvrir les ressources

``` bash
# Lister les types de ressources connus
kubectl api-resources

# Lister les CRD installées
kubectl get crd

# Examiner les champs d'un type de ressource
kubectl explain deployment
kubectl explain deployment.spec
kubectl explain statefulset.spec
kubectl explain service.spec
```

------------------------------------------------------------------------

# 9. Méthode de diagnostic

Lorsqu'une application n'est pas accessible, éviter de commencer par
modifier plusieurs manifests. Procéder du Pod vers le réseau, en
vérifiant chaque relation.

## 9.1 L'application ne démarre pas

``` bash
kubectl get pods -n demo
kubectl describe pod NOM_DU_POD -n demo
kubectl logs NOM_DU_POD -n demo
kubectl get events -n demo --sort-by=.metadata.creationTimestamp
```

Vérifier notamment :

-   `ImagePullBackOff` : problème de récupération de l'image ;
-   `CrashLoopBackOff` : le conteneur démarre puis échoue de façon
    répétée ;
-   `CreateContainerConfigError` : configuration du conteneur invalide
    ou référence manquante ;
-   `Pending` : problème de planification, de ressources, de stockage ou
    de contraintes ;
-   `Readiness probe failed` : le conteneur n'est pas considéré comme
    prêt.

## 9.2 Le Deployment n'a pas le nombre de Pods attendu

``` bash
kubectl describe deployment api-commandes -n demo
kubectl get rs -n demo
kubectl get pods -l app=api-commandes -n demo
```

Vérifier :

-   le nombre de réplicas demandé ;
-   le selector et les labels du template ;
-   les ReplicaSets ;
-   les événements du Deployment et des Pods.

## 9.3 Le Service n'envoie pas le trafic

``` bash
kubectl describe svc api-commandes -n demo
kubectl get endpointslices -n demo
kubectl get pods --show-labels -n demo
```

Vérifier :

-   que le selector du Service correspond aux labels des Pods ;
-   que les Pods sont Ready ;
-   que le `targetPort` correspond au port écouté par l'application ;
-   que les règles réseau et politiques éventuelles autorisent le
    trafic.

## 9.4 Un Secret n'est pas disponible

``` bash
kubectl describe pod NOM_DU_POD -n demo
kubectl describe secret application-secret -n demo
```

`describe secret` permet de vérifier les noms des clés et leur taille
sans afficher les valeurs. Ne pas partager un
`kubectl get secret -o yaml` non expurgé : les valeurs sont encodées en
Base64, ce qui n'est pas un chiffrement.

Si le Secret est géré par External Secrets Operator :

``` bash
kubectl get externalsecrets -n demo
kubectl describe externalsecret application-secret -n demo
```

Vérifier séparément :

1.  L'état de synchronisation de l'ExternalSecret ;
2.  le nom du Secret créé ;
3.  les noms des clés réellement présentes dans ce Secret ;
4.  le nom de clé attendu dans `secretKeyRef.key`.

Un ExternalSecret `Ready=True` signifie que la synchronisation a réussi
selon l'Operator ; cela ne garantit pas que le Deployment référence le
bon nom de clé.

------------------------------------------------------------------------

# 10. Résumé des notions

  -----------------------------------------------------------------------
  Notion                              Question à se poser
  ----------------------------------- -----------------------------------
  Deployment                          Combien de réplicas de mon
                                      application dois-je maintenir ?

  ReplicaSet                          Comment maintenir le nombre de Pods
                                      correspondant au selector ?

  Pod template                        Quelle configuration et quels
                                      labels donner aux Pods créés ?

  Label                               Comment identifier une ressource ?

  Selector                            Quelles ressources doivent être
                                      sélectionnées ?

  Service                             Comment fournir un point d'accès
                                      stable à mon application ?

  EndpointSlice                       Vers quelles destinations réseau le
                                      Service peut-il envoyer le trafic ?

  ConfigMap                           Où stocker ma configuration non
                                      sensible ?

  Secret                              Où référencer mes données sensibles
                                      ?

  CRD                                 Comment ajouter un nouveau type de
                                      ressource à Kubernetes ?

  Operator                            Quel programme réconcilie cette
                                      ressource personnalisée ?

  ServiceAccount                      Quelle identité mon Pod
                                      utilise-t-il ?

  Role / RoleBinding                  Quelles permissions donner dans un
                                      namespace et à qui ?

  StatefulSet                         Mes Pods ont-ils besoin d'une
                                      identité et d'un stockage stables ?

  PVC                                 Quel volume persistant un Pod
                                      doit-il utiliser ?
  -----------------------------------------------------------------------

## Architecture cible de notre futur exercice Spring Boot

Nous construirons progressivement une petite application composée d'une
API Spring Boot et d'une base PostgreSQL. Les ressources seront ajoutées
une par une, puis gérées avec Helm et GitOps.

``` mermaid
flowchart TD
    C["Client HTTP"]
    I["Ingress"]
    S["Service API"]
    D["Deployment Spring Boot"]
    R["ReplicaSet"]
    P1["Pod API 1"]
    P2["Pod API 2"]
    DB["Service PostgreSQL"]
    STS["StatefulSet PostgreSQL"]
    PG["Pod PostgreSQL"]
    PVC["PVC / stockage persistant"]
    CM["ConfigMap"]
    SEC["Secret"]
    C --> I --> S --> D --> R
    R --> P1
    R --> P2
    P1 --> DB
    P2 --> DB
    DB --> STS --> PG --> PVC
    CM -.-> P1
    CM -.-> P2
    SEC -.-> P1
    SEC -.-> P2
```

> Ce schéma présente les relations conceptuelles. Les manifests
> définitifs préciseront notamment les ports, les probes, les
> ressources, les références aux secrets et le stockage.

## Principes à ne pas oublier

1.  **Déclarer l'état souhaité** plutôt que créer manuellement des Pods
    isolés.
2.  **Comprendre les labels et selectors** avant de diagnostiquer un
    Deployment ou un Service.
3.  **Ne jamais supposer qu'un Service possède des destinations** :
    vérifier les EndpointSlices.
4.  **Distinguer CRD et Operator** : l'un définit un type, l'autre
    exécute la logique.
5.  **Appliquer le moindre privilège** avec RBAC.
6.  **Choisir Deployment ou StatefulSet selon le besoin**, pas
    uniquement selon le type de technologie.
7.  **En environnement GitOps**, effectuer les changements durables dans
    Git et laisser l'outil GitOps réconcilier le cluster.

------------------------------------------------------------------------

*Document de référence évolutif --- à compléter au fil des exercices
pratiques Kubernetes, Helm et GitOps.*
