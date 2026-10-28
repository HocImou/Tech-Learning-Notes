# PostgreSQL avec Docker : commandes pratiques

## 1. Objectif

Cette documentation rassemble les commandes Docker et PostgreSQL utiles pour administrer une base de données PostgreSQL exécutée dans un conteneur Docker.

Elle couvre notamment :

- Le démarrage et l'arrêt du conteneur PostgreSQL.
- La consultation des logs et l'état de santé du conteneur.
- La connexion à PostgreSQL avec `psql`.
- L'inspection des bases, tables, colonnes et données.
- L'exécution de requêtes SQL.
- Le diagnostic des problèmes de connexion.
- La sauvegarde et la restauration des données.

Les exemples utilisent le projet `spring-boot-architecture-lab`.

### Environnement de référence

| Élément | Valeur |
|---|---|
| Conteneur | `architecture-lab-postgres` |
| Base de données | `architecture_lab` |
| Utilisateur PostgreSQL | `app` |
| Port PostgreSQL | `5432` |
| Outil SQL | `psql` |

Ces valeurs sont propres à l'exemple. Il faut les adapter à son propre environnement.

## 2. Comprendre l'architecture

Dans notre environnement, PostgreSQL s'exécute dans un conteneur Docker, tandis que l'application Spring Boot s'exécute directement sur la machine.

```text
Machine de développement
│
├── Application Spring Boot
│   └── localhost:5432
│
└── Docker
    └── Conteneur PostgreSQL
        └── Base architecture_lab
```

La configuration JDBC de l'application est :

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/architecture_lab
spring.datasource.username=app
```

Le port `5432` du conteneur est publié sur le port `5432` de la machine hôte.

## 3. Gérer le conteneur PostgreSQL

Les commandes suivantes sont à exécuter dans un terminal PowerShell ouvert à la racine du projet contenant `compose.yaml`.

### 3.1 Démarrer PostgreSQL

```powershell
docker compose up -d postgres
```

Explications :

- `docker compose` utilise la configuration Compose du projet.
- `up` crée et démarre les services nécessaires.
- `-d` lance les conteneurs en arrière-plan.
- `postgres` désigne le service à démarrer.

Si le service n'existe pas encore, Docker le crée à partir de la configuration.

### 3.2 Vérifier l'état des conteneurs

```powershell
docker compose ps
```

Cette commande affiche les conteneurs gérés par le projet Compose, leur état et les ports publiés.

Pour consulter tous les conteneurs, y compris ceux qui ne sont pas gérés par le projet courant :

```powershell
docker ps -a
```

Pour n'afficher que les conteneurs en cours d'exécution :

```powershell
docker ps
```

### 3.3 Arrêter PostgreSQL

```powershell
docker compose stop postgres
```

Le conteneur est arrêté, mais il n'est pas supprimé. Les données stockées dans le volume persistent.

Pour redémarrer le service :

```powershell
docker compose start postgres
```

### 3.4 Arrêter et supprimer les conteneurs du projet

```powershell
docker compose down
```

Cette commande arrête et supprime les conteneurs ainsi que les réseaux créés par Compose. Elle ne supprime pas les volumes nommés par défaut.

**Attention :** ne pas ajouter `-v` sans avoir vérifié les conséquences. `docker compose down -v` supprime également les volumes nommés du projet, ce qui peut entraîner la perte des données PostgreSQL qui y sont stockées.

## 4. Consulter les logs et l'état de santé

### 4.1 Afficher les logs PostgreSQL

```powershell
docker compose logs postgres
```

Pour suivre les nouveaux messages en temps réel :

```powershell
docker compose logs -f postgres
```

Pour afficher uniquement les 100 dernières lignes :

```powershell
docker compose logs --tail 100 postgres
```

Les logs sont utiles pour diagnostiquer :

- Une erreur de démarrage.
- Un problème de configuration.
- Un échec d'initialisation de la base.
- Une erreur de connexion ou d'authentification.

Pour quitter le suivi des logs, utiliser `Ctrl + C`.

### 4.2 Vérifier que PostgreSQL répond

```powershell
docker exec architecture-lab-postgres pg_isready -U app -d architecture_lab
```

`docker exec` exécute une commande dans un conteneur existant.

`pg_isready` vérifie si le serveur PostgreSQL accepte les connexions. Un résultat positif indique que le serveur est prêt à répondre, mais ne garantit pas que toutes les opérations SQL réussiront.

### 4.3 Inspecter le conteneur

```powershell
docker inspect architecture-lab-postgres
```

Pour afficher uniquement l'état de santé :

```powershell
docker inspect --format='{{json .State.Health}}' architecture-lab-postgres
```

Cette dernière commande nécessite qu'un healthcheck soit configuré dans le conteneur.

## 5. Se connecter à PostgreSQL avec `psql`

`psql` est le client en ligne de commande de PostgreSQL. Il permet d'exécuter des requêtes SQL et d'inspecter la base.

### 5.1 Ouvrir une session interactive

```powershell
docker exec -it architecture-lab-postgres psql -U app -d architecture_lab
```

Décomposition :

| Élément | Signification |
|---|---|
| `docker exec` | Exécute une commande dans un conteneur existant |
| `-i` | Maintient l'entrée standard ouverte |
| `-t` | Alloue un terminal |
| `architecture-lab-postgres` | Nom du conteneur |
| `psql` | Client PostgreSQL |
| `-U app` | Utilisateur PostgreSQL |
| `-d architecture_lab` | Base de données cible |

Une fois connecté, le prompt devient généralement :

```text
architecture_lab=>
```

Les commandes suivantes sont des méta-commandes propres à `psql`. Elles commencent généralement par une barre oblique inverse et ne nécessitent pas de point-virgule.

### 5.2 Lister les bases de données

```text
\l
```

Affiche les bases de données accessibles au serveur PostgreSQL.

### 5.3 Lister les tables

```text
\dt
```

Affiche les tables du schéma courant ou des schémas correspondant au filtre utilisé.

Pour afficher les tables de tous les schémas :

```text
\dt *.*
```

### 5.4 Décrire une table

```text
\d products
```

Affiche la structure de la table `products` : colonnes, types, contraintes et autres informations.

Pour obtenir davantage de détails :

```text
\d+ products
```

### 5.5 Lister les schémas

```text
\dn
```

Les schémas permettent d'organiser les objets d'une base PostgreSQL. Le schéma `public` est fréquemment utilisé par défaut.

### 5.6 Afficher les connexions actives

```sql
SELECT pid, usename, datname, state, query
FROM pg_stat_activity
WHERE datname = 'architecture_lab';
```

Cette requête permet de consulter les sessions visibles par l'utilisateur courant. La visibilité des requêtes et des autres sessions dépend des privilèges PostgreSQL.

### 5.7 Quitter `psql`

```text
\q
```

Ferme la session interactive.

## 6. Exécuter du SQL sans ouvrir une session interactive

Il est possible de lancer une requête directement depuis PowerShell.

### 6.1 Lister les tables

```powershell
docker exec architecture-lab-postgres psql -U app -d architecture_lab -c "\dt"
```

Le paramètre `-c` demande à `psql` d'exécuter une commande puis de terminer.

### 6.2 Consulter les données d'une table

```powershell
docker exec architecture-lab-postgres psql -U app -d architecture_lab -c "SELECT * FROM products;"
```

### 6.3 Compter les lignes

```powershell
docker exec architecture-lab-postgres psql -U app -d architecture_lab -c "SELECT COUNT(*) FROM products;"
```

### 6.4 Vérifier l'historique des migrations Flyway

Dans le projet Spring Boot, Flyway conserve l'historique des migrations dans la table `flyway_schema_history`, si cette table a été créée.

```powershell
docker exec architecture-lab-postgres psql -U app -d architecture_lab -c "SELECT version, description, success FROM flyway_schema_history ORDER BY installed_rank;"
```

Cette commande permet de vérifier quelles migrations ont été enregistrées et si elles ont réussi.

Si la table n'existe pas, vérifier que l'application a bien démarré avec Flyway et que les migrations sont configurées.

## 7. Commandes SQL utiles au quotidien

### 7.1 Lister les tables du schéma `public`

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
  AND table_type = 'BASE TABLE'
ORDER BY table_name;
```

### 7.2 Afficher les colonnes d'une table

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name = 'products'
ORDER BY ordinal_position;
```

### 7.3 Compter les lignes d'une table

```sql
SELECT COUNT(*) FROM products;
```

### 7.4 Consulter les index

```sql
SELECT indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'public'
  AND tablename = 'products';
```

### 7.5 Vérifier la base et l'utilisateur courant

```sql
SELECT current_database(), current_user;
```

### 7.6 Vérifier les contraintes

```sql
SELECT conname, contype
FROM pg_constraint
WHERE conrelid = 'products'::regclass;
```

Cette dernière requête suppose que la table `products` existe et est accessible dans le schéma recherché par PostgreSQL.

## 8. Diagnostic des problèmes fréquents

### 8.1 Le conteneur n'existe pas

Erreur possible :

```text
No such container: architecture-lab-postgres
```

Vérifier les conteneurs et le service Compose :

```powershell
docker ps -a
docker compose ps
```

Le conteneur peut avoir un autre nom ou ne pas avoir encore été créé.

### 8.2 PostgreSQL n'est pas prêt

Vérifier :

```powershell
docker compose ps
docker compose logs --tail 100 postgres
docker exec architecture-lab-postgres pg_isready -U app -d architecture_lab
```

Si PostgreSQL est en cours d'initialisation, attendre qu'il soit prêt avant de relancer les commandes.

### 8.3 Erreur d'authentification

Exemple :

```text
password authentication failed for user "app"
```

Vérifier l'utilisateur et les paramètres de connexion.

Attention : modifier `POSTGRES_PASSWORD` dans `compose.yaml` ne change pas automatiquement le mot de passe d'un utilisateur dans un volume PostgreSQL déjà initialisé. Les variables d'initialisation de l'image officielle s'appliquent principalement lors de la création initiale de la base.

### 8.4 Erreur de connexion depuis Spring Boot

Exemple :

```text
Connection refused
```

Vérifier dans l'ordre :

1. Le conteneur PostgreSQL est démarré.
2. PostgreSQL est prêt à accepter les connexions.
3. Le port `5432` est correctement publié.
4. L'URL JDBC, le nom de la base et les identifiants sont corrects.

Si Spring Boot s'exécute sur la machine hôte, `localhost:5432` peut être utilisé lorsque le port est publié ainsi. Si Spring Boot s'exécute dans un autre conteneur du même réseau Compose, il faut généralement utiliser le nom du service, par exemple `postgres:5432`.

### 8.5 Le port est déjà utilisé

Si Docker ne peut pas publier le port `5432`, vérifier les conteneurs existants :

```powershell
docker ps
```

On peut aussi examiner les ports en écoute sous Windows :

```powershell
Get-NetTCPConnection -LocalPort 5432 -State Listen
```

Il faut ensuite identifier le processus qui utilise le port avant de modifier la configuration.

## 9. Sauvegarder et restaurer une base

Une sauvegarde est importante avant toute opération risquée.

### 9.1 Créer une sauvegarde au format PostgreSQL personnalisé

La commande suivante crée une sauvegarde dans le conteneur :

```powershell
docker exec architecture-lab-postgres pg_dump -U app -d architecture_lab -F c -f /tmp/architecture_lab.dump
```

Puis copier le fichier sur la machine hôte :

```powershell
docker cp architecture-lab-postgres:/tmp/architecture_lab.dump .\architecture_lab.dump
```

Le format personnalisé (`-F c`) est adapté à la restauration avec `pg_restore`.

Cette sauvegarde contient les objets et données accessibles au compte utilisé. Pour sauvegarder l'ensemble des objets globaux du serveur, comme certains rôles, il faut envisager une sauvegarde complémentaire avec `pg_dumpall` et les privilèges appropriés.

### 9.2 Restaurer une sauvegarde

**Attention :** restaurer une sauvegarde peut modifier ou écraser des objets existants. Vérifier la base cible et conserver une sauvegarde préalable.

Copier le fichier dans le conteneur :

```powershell
docker cp .\architecture_lab.dump architecture-lab-postgres:/tmp/architecture_lab.dump
```

Pour restaurer dans une base vide déjà créée :

```powershell
docker exec architecture-lab-postgres pg_restore -U app -d architecture_lab /tmp/architecture_lab.dump
```

Si les objets existent déjà, la restauration peut échouer en raison de conflits. Il faut alors choisir explicitement la stratégie de restauration adaptée au contexte.

## 10. Bonnes pratiques

- Utiliser `docker compose` pour gérer les services définis dans le fichier Compose.
- Consulter les logs avant de redémarrer ou de recréer un conteneur.
- Vérifier la disponibilité de PostgreSQL avant de diagnostiquer l'application Spring Boot.
- Utiliser un volume nommé pour conserver les données entre les recréations de conteneurs.
- Ne pas supprimer un volume sans vérifier les données qu'il contient.
- Ne pas versionner de vrais mots de passe ou secrets dans le dépôt Git.
- Utiliser des sauvegardes et tester régulièrement leur restauration.
- Éviter de lancer des commandes destructrices sur une base partagée ou de production sans validation.
- Distinguer les commandes Docker, les méta-commandes `psql` et les requêtes SQL.

## 11. Références officielles

- Docker Compose : https://docs.docker.com/compose/
- Docker `exec` : https://docs.docker.com/reference/cli/docker/container/exec/
- Image officielle PostgreSQL : https://hub.docker.com/_/postgres
- Documentation PostgreSQL : https://www.postgresql.org/docs/
- Client `psql` : https://www.postgresql.org/docs/current/app-psql.html
- `pg_dump` : https://www.postgresql.org/docs/current/app-pgdump.html
- `pg_restore` : https://www.postgresql.org/docs/current/app-pgrestore.html
