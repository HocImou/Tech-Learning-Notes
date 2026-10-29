# Conventional Commits — Convention de messages Git

## 1. Introduction

Un commit Git représente un ensemble de modifications enregistrées dans l'historique d'un dépôt.

Le message associé à un commit doit permettre de comprendre rapidement :

- La nature de la modification.
- La partie du projet concernée.
- L'objectif du changement.

Dans un projet professionnel, adopter une convention commune permet de rendre l'historique Git plus lisible, de faciliter les revues de code et d'améliorer la collaboration entre développeurs.

La convention **Conventional Commits** propose un format standardisé pour rédiger les messages de commit.

Documentation officielle : [Conventional Commits](https://www.conventionalcommits.org/fr/v1.0.0/)

---

## 2. Structure d'un message Conventional Commit

La structure générale est la suivante :

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Chaque partie a un rôle précis.

| Élément | Description | Obligatoire |
|---|---|---|
| `type` | Nature de la modification | Oui |
| `scope` | Partie du projet concernée | Non |
| `description` | Résumé concis du changement | Oui |
| `body` | Explication détaillée du changement | Non |
| `footer` | Informations complémentaires, références ou breaking changes | Non |

### Exemple simple

```text
feat(auth): add JWT authentication
```

Dans cet exemple :

- `feat` : ajout d'une fonctionnalité.
- `auth` : périmètre concerné.
- `add JWT authentication` : description du changement.

### Exemple détaillé

```text
feat(auth): add JWT authentication

Implement JWT-based authentication for REST endpoints.
Add token validation and authorization filters.

Closes #42
```

Le message contient :
- Un en-tête qui résume le changement.
- Un corps qui explique les modifications.
- Un footer qui référence une issue GitHub.

---

## 3. Les principaux types de commit

Le type indique la nature du changement.

| Type | Utilisation | Exemple |
|---|---|---|
| `feat` | Ajout d'une fonctionnalité | `feat(user): add user registration` |
| `fix` | Correction d'un bug | `fix(auth): handle expired tokens` |
| `docs` | Modification de la documentation | `docs(readme): update installation guide` |
| `style` | Modifications de formatage sans changement fonctionnel | `style(api): format controller classes` |
| `refactor` | Réorganisation du code sans modification fonctionnelle attendue | `refactor(user): simplify user service` |
| `perf` | Amélioration des performances | `perf(batch): optimize database inserts` |
| `test` | Ajout ou modification de tests | `test(auth): add JWT validation tests` |
| `build` | Modification du système de build ou des dépendances | `build(maven): update spring boot version` |
| `ci` | Modification de la configuration CI/CD | `ci(gitlab): add Docker image build stage` |
| `chore` | Tâches diverses de maintenance | `chore(deps): update dependencies` |
| `revert` | Annulation d'un commit précédent | `revert: feat(auth): add JWT authentication` |

**Attention aux distinctions :**

- `feat` : le comportement ou les possibilités du produit évoluent avec une nouvelle fonctionnalité.
- `fix` : un comportement incorrect est corrigé.
- `refactor` : le code est amélioré sans changement fonctionnel intentionnel.
- `style` : le formatage est modifié sans impact sur la logique.
- `docs` : seule la documentation est concernée.

Les types autres que `feat` et `fix` sont des conventions d'équipe courantes. Conventional Commits n'impose pas une liste fermée de types.

---

## 4. Le scope : préciser la partie concernée

Le scope est une information facultative qui permet de préciser le périmètre du changement.

Il est placé entre parenthèses après le type.

```text
<type>(<scope>): <description>
```

Exemples :

```text
feat(auth): implement OAuth2 authentication

fix(batch): handle empty CSV files

refactor(user): extract validation logic

docs(git): add commit conventions

test(repository): add integration tests
```

Le scope doit être cohérent avec l'organisation du projet.

Dans une application Spring Boot, on pourrait utiliser :

- `auth`
- `user`
- `controller`
- `service`
- `repository`
- `batch`
- `security`
- `config`

Dans une application Angular :

- `auth`
- `components`
- `services`
- `routing`
- `state`

**Bonne pratique :** privilégier des scopes courts, compréhensibles et cohérents au sein d'une même équipe.

Il n'est pas nécessaire d'ajouter un scope lorsque le changement concerne l'ensemble du projet.

---

## 5. Rédiger une bonne description

La description doit être concise et expliquer clairement le changement.

### Bonnes pratiques

- Utiliser une formulation à l'impératif.
- Décrire ce que le commit apporte.
- Éviter les descriptions trop génériques.
- Ne pas détailler toute l'implémentation dans l'en-tête.
- Éviter les points finaux inutiles.
- Utiliser une formulation cohérente dans toute l'équipe.

### Exemples

| À éviter | Préférer |
|---|---|
| `fix: correction` | `fix(auth): handle invalid JWT tokens` |
| `feat: ajout` | `feat(user): add user registration` |
| `update: modification API` | `refactor(api): simplify response mapping` |
| `docs: update` | `docs(readme): document local setup` |
| `fix: problème SQL` | `fix(repository): correct pagination query` |

La description peut être rédigée en anglais ou en français. L'essentiel est de conserver une convention homogène dans le projet.

Dans un projet technique international, l'anglais est généralement un bon choix.

---

## 6. Le body : expliquer le pourquoi et le comment

Le body est facultatif. Il permet de compléter l'en-tête lorsque le changement nécessite davantage d'explications.

Il est particulièrement utile pour expliquer :

- Le problème rencontré.
- La raison du changement.
- Les choix techniques effectués.
- Les conséquences sur le fonctionnement de l'application.

Exemple :

```text
fix(batch): prevent duplicate customer imports

The batch could import the same customer multiple times
when the source file contained duplicate identifiers.

Add a duplicate detection step before database insertion.
```

L'en-tête résume la correction.

Le body explique le problème et la solution retenue.

**Bonne pratique :** expliquer pourquoi le changement est nécessaire plutôt que simplement répéter ce que le code fait.

---

## 7. Les footers : références et changements incompatibles

Les footers permettent d'ajouter des informations complémentaires à la fin du message.

### 7.1. Référencer une issue

Exemple avec GitHub :

```text
feat(user): add user export

Allow administrators to export user data.

Closes #42
```

Les mots-clés `Closes`, `Fixes` ou `Resolves` peuvent permettre de fermer automatiquement une issue GitHub lorsque le commit est intégré dans la branche par défaut, selon le contexte d'intégration.

### 7.2. Signaler un Breaking Change

Un *Breaking Change* est une modification qui introduit une incompatibilité avec une utilisation ou une interface existante.

Exemple :

```text
feat(api): change user response structure

BREAKING CHANGE: The user endpoint no longer returns
the legacy "fullname" field.

Clients must now use the "firstName" and "lastName" fields.
```

Le footer `BREAKING CHANGE:` indique explicitement cette incompatibilité.

Il est également possible d'utiliser `!` après le type ou le scope :

```text
feat(api)!: change user response structure
```

Les deux mécanismes peuvent être combinés, mais ce n'est pas obligatoire.

### Pourquoi signaler les Breaking Changes ?

Cela permet notamment :

- D'alerter les consommateurs de l'API.
- De faciliter la préparation des montées de version.
- D'identifier les modifications nécessitant une adaptation côté client.
- De permettre à certains outils de déterminer qu'une version majeure est nécessaire.

---

## 8. Exemples concrets dans un projet Java Spring Boot

### Ajout d'une fonctionnalité

```text
feat(user): add user creation endpoint
```

### Correction d'un bug

```text
fix(auth): reject expired JWT tokens
```

### Refactoring d'un service

```text
refactor(order): extract business validation logic
```

### Ajout de tests

```text
test(order): add integration tests for order creation
```

### Optimisation d'un batch

```text
perf(batch): optimize PostgreSQL batch inserts
```

### Modification de la configuration Maven

```text
build(maven): update compiler plugin
```

### Modification du pipeline GitLab

```text
ci(gitlab): add Docker image publishing stage
```

### Mise à jour de la documentation

```text
docs(architecture): document microservices communication
```

### Modification de la configuration Kubernetes

```text
chore(kubernetes): update deployment resource limits
```

---

## 9. Utilisation avec Git

La convention ne nécessite aucune configuration particulière de Git.

Elle s'applique directement lors de la création des commits.

### Commit simple

```bash
git add .
git commit -m "feat(user): add user registration"
```

### Commit avec un body

```bash
git commit
```

Git ouvre alors l'éditeur de texte configuré pour rédiger le message complet.

Exemple :

```text
fix(batch): handle empty input files

Prevent the batch from failing when the input CSV is empty.
Add validation before processing starts.
```

### Consulter l'historique

```bash
git log --oneline
```

Exemple de résultat :

```text
a1b2c3d feat(auth): add JWT authentication
b2c3d4e fix(batch): handle empty CSV files
c3d4e5f docs(readme): update installation guide
d4e5f6g refactor(user): simplify user service
```

Un historique standardisé est plus facile à parcourir et à comprendre.

---

## 10. Automatiser la vérification des commits

La convention peut être appliquée manuellement, mais il est également possible d'automatiser sa vérification.

Quelques outils courants :

| Outil | Rôle |
|---|---|
| Commitlint | Vérifie que les messages respectent les règles définies |
| Husky | Permet de déclencher des hooks Git |
| Commitizen | Aide à construire des messages de commit structurés |
| semantic-release | Automatise notamment les versions et les changelogs à partir des commits |

### Exemple d'utilisation de Commitlint

Commitlint permet de vérifier un message :

```bash
npx commitlint --edit .git/COMMIT_EDITMSG
```

La configuration dépend du projet et des règles retenues par l'équipe.

Dans un projet Java avec GitLab CI/CD, on peut également intégrer une vérification des messages de commit dans le pipeline.

**Attention :** Conventional Commits définit une convention de rédaction. Il ne configure pas automatiquement Commitlint, Husky ou semantic-release.

---

## 11. Conventional Commits et Semantic Versioning

Conventional Commits peut faciliter l'application du versionnement sémantique (*Semantic Versioning*).

Le Semantic Versioning utilise trois numéros :

```text
MAJOR.MINOR.PATCH
```

| Élément | Signification |
|---|---|
| `MAJOR` | Modification incompatible avec les versions précédentes |
| `MINOR` | Ajout de fonctionnalités compatibles |
| `PATCH` | Correction compatible |

Correspondance courante avec Conventional Commits :

| Type de commit | Impact habituel |
|---|---|
| `feat` | MINOR |
| `fix` | PATCH |
| `BREAKING CHANGE` | MAJOR |

Exemple :

```text
feat(user): add user export
```

Peut entraîner une augmentation de la version mineure.

```text
fix(auth): handle invalid tokens
```

Peut entraîner une augmentation de la version corrective.

```text
feat(api)!: remove legacy endpoint
```

Signale un changement incompatible qui peut entraîner une augmentation de la version majeure.

Cette automatisation dépend toutefois de l'outil et de sa configuration. Tous les commits ne déclenchent pas nécessairement une nouvelle version.

---

## 12. Convention recommandée pour les projets d'équipe

Pour garantir la cohérence des messages, il est recommandé de définir quelques règles communes :

1. Utiliser un type adapté à la nature du changement.
2. Ajouter un scope lorsque cela améliore la compréhension.
3. Rédiger une description courte et explicite.
4. Utiliser l'anglais si le projet est international.
5. Ajouter un body lorsque le changement nécessite des explications.
6. Référencer les issues lorsque cela est pertinent.
7. Signaler explicitement les Breaking Changes.
8. Éviter les commits regroupant des modifications sans rapport entre elles.

### Format recommandé

```text
<type>(<scope>): <description>

[Pourquoi le changement est nécessaire et détails utiles]

[Référence issue ou BREAKING CHANGE]
```

### Exemples de messages à privilégier

```text
feat(auth): implement refresh token mechanism

fix(batch): prevent duplicate records during import

refactor(repository): simplify customer query handling

test(api): add integration tests for authentication

docs(git): document conventional commit rules

ci(gitlab): add automated commit message validation
```

---

## 13. Ressources complémentaires

- [Conventional Commits — Documentation officielle](https://www.conventionalcommits.org/fr/v1.0.0/)
- [Semantic Versioning](https://semver.org/)
- [Commitlint](https://commitlint.js.org/)
- [Commitizen](https://commitizen-tools.github.io/commitizen/)
- [Semantic Release](https://semantic-release.gitbook.io/semantic-release/)

---

## Conclusion

Conventional Commits permet de structurer les messages Git afin de rendre l'historique plus compréhensible et exploitable.

Au-delà de la simple rédaction des messages, cette convention peut devenir un élément important de la qualité et de la gouvernance d'un projet.

Associée à des outils comme Commitlint et semantic-release, elle peut contribuer à automatiser certaines étapes du cycle de livraison logicielle.