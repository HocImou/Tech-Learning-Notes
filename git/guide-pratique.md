# Git — Guide pratique

Ce document regroupe les commandes Git les plus utiles dans un contexte professionnel avec **GitLab**, branches de fonctionnalités, Merge Requests, tags et livraisons CI/CD.

---

## 1. Commandes essentielles

| Besoin | Commande |
|---|---|
| Voir l'état du dépôt | `git status` |
| Voir les branches | `git branch` |
| Voir la branche courante | `git branch --show-current` |
| Voir l'historique | `git log --oneline --graph --decorate --all` |
| Récupérer les informations du remote | `git fetch origin` |
| Voir les remotes | `git remote -v` |
| Voir les modifications | `git diff` |
| Ajouter un fichier | `git add <fichier>` |
| Ajouter tous les fichiers | `git add .` |
| Créer un commit | `git commit -m "message"` |
| Envoyer vers GitLab | `git push` |
| Récupérer une branche | `git pull --rebase` |

---

# 2. Commencer une nouvelle fonctionnalité

Exemple : US `US_6133-22`.

Commencer par récupérer l'état du remote :

```bash
git fetch origin
```

Se positionner sur `develop` :

```bash
git switch develop
git pull --rebase origin develop
```

Créer la branche :

```bash
git switch -c US_6133-22
```

Vérifier :

```bash
git branch --show-current
```

Créer la branche distante:

```bash
git push -u origin US_6133-22
```

---

# 3. Enregistrer son travail

Voir l'état :

```bash
git status
```

Voir les modifications :

```bash
git diff
```

Ajouter les fichiers :

```bash
git add .
```

Créer le commit :

```bash
git commit -m "feat: implement US_6133-22"
```

Envoyer la branche :

```bash
git push -u origin US_6133-22
```

Après le premier push, on peut généralement utiliser simplement :

```bash
git push
```

---

# 4. Mettre sa branche à jour avec `develop`

Si `develop` a avancé pendant que tu travailles :

```bash
git fetch origin
git rebase origin/develop
```

Si le rebase réussit :

```bash
git push --force-with-lease
```

### Pourquoi `--force-with-lease` ?

Un rebase réécrit les commits de ta branche. Le push normal peut donc être refusé.

Préférer :

```bash
git push --force-with-lease
```

à :

```bash
git push --force
```

`--force-with-lease` protège mieux contre l'écrasement du travail poussé par quelqu'un d'autre.

---

# 5. Faire un merge de `develop`

Alternative au rebase :

```bash
git fetch origin
git merge origin/develop
```

Le merge conserve un commit de fusion.

Le rebase produit généralement un historique plus linéaire :

```text
Avant :

A---B---C  develop
     \
      D---E  feature


Après rebase :

A---B---C---D'---E'  feature
```

Pour une branche personnelle, `rebase` est souvent pratique pour maintenir un historique linéaire.

---

# 6. Résoudre un conflit pendant un rebase

Après :

```bash
git rebase origin/develop
```

Git peut signaler :

```text
CONFLICT (content)
```

Commencer par :

```bash
git status
```

Corriger les fichiers concernés.

Puis :

```bash
git add <fichier>
git rebase --continue
```

S'il y a plusieurs conflits, répéter :

```bash
git status
git add <fichier>
git rebase --continue
```

### Abandonner le rebase

Si tu veux revenir à l'état avant le rebase :

```bash
git rebase --abort
```

---

# 7. Le `git push` est refusé

Erreur typique :

```text
Updates were rejected because the remote contains work
that you do not have locally.
```

Ne pas utiliser immédiatement :

```bash
git push --force
```

Faire :

```bash
git fetch origin
git status
```

Puis généralement :

```bash
git pull --rebase origin <branche>
```

Et enfin :

```bash
git push
```

Exemple :

```bash
git pull --rebase origin US_6133-22
git push
```

---

# 8. Annuler une modification locale

Pour un fichier :

```bash
git restore <fichier>
```

Pour toutes les modifications non committées :

```bash
git restore .
```

> ⚠️ Ces modifications seront perdues.

---

# 9. Retirer un fichier du staging

Après :

```bash
git add .
```

si tu veux retirer un fichier du staging :

```bash
git restore --staged <fichier>
```

Pour tout retirer du staging :

```bash
git restore --staged .
```

Les modifications restent présentes dans les fichiers.

---

# 10. Modifier le dernier commit

Si tu as oublié d'ajouter un fichier :

```bash
git add <fichier>
git commit --amend --no-edit
```

Si tu veux également modifier le message :

```bash
git commit --amend -m "feat: implement US_6133-22"
```

Si le commit avait déjà été poussé :

```bash
git push --force-with-lease
```

---

# 11. Annuler un commit déjà poussé

Sur une branche partagée comme `develop`, préférer `git revert`.

Exemple :

```bash
git revert <commit>
```

Git crée un nouveau commit qui annule les changements du commit précédent.

Puis :

```bash
git push
```

### Pourquoi préférer `revert` ?

Parce que `revert` ne réécrit pas l'historique.

Éviter sur une branche partagée :

```bash
git reset --hard
git push --force
```

---

# 12. Supprimer une branche

## Branche locale

```bash
git branch -d US_6133-22
```

Si elle n'est pas mergée et que tu veux forcer :

```bash
git branch -D US_6133-22
```

## Branche distante

```bash
git push origin --delete US_6133-22
```

---

# 13. Consulter l'historique

Historique simple :

```bash
git log --oneline
```

Historique graphique :

```bash
git log --oneline --graph --decorate --all
```

Chercher un commit par message :

```bash
git log --oneline --grep="US_6133"
```

Voir le contenu d'un commit :

```bash
git show <commit>
```

Exemple :

```bash
git show a84f123
```

---

# 14. Trouver qui a modifié une ligne

```bash
git blame <fichier>
```

Exemple :

```bash
git blame src/main/java/.../MonService.java
```

Puis consulter le commit :

```bash
git show <commit>
```

---

# 15. Les tags

Les tags sont particulièrement utiles pour les livraisons.

## Voir les tags

```bash
git tag
```

## Créer un tag simple

```bash
git tag res-enri:1.10.1.2
```

## Créer un tag annoté

```bash
git tag -a res-enri:1.10.1.2 -m "Release res-enri 1.10.1.2"
```

## Envoyer un tag vers GitLab

```bash
git push origin res-enri:1.10.1.2
```

Pour un tag annoté portant le même nom :

```bash
git push origin res-enri:1.10.1.2
```

---

# 16. Vérifier sur quel commit pointe un tag

```bash
git show res-enri:1.10.1.2
```

Ou :

```bash
git rev-list -n 1 res-enri:1.10.1.2
```

---

# 17. Supprimer un tag

## Localement

```bash
git tag -d res-enri:1.10.1.2
```

## Sur le remote

```bash
git push origin --delete res-enri:1.10.1.2
```

Puis, si nécessaire, recréer le tag sur le bon commit.

---

# 18. Workflow GitLab + livraison Preprod

Dans un projet où le tag déclenche la livraison, un workflow typique est :

```bash
# 1. Vérifier la branche
git branch --show-current

# 2. Vérifier l'état
git status

# 3. Récupérer les changements distants
git fetch origin

# 4. Mettre à jour la branche
git pull --rebase origin <branche>

# 5. Développer et modifier les fichiers

# 6. Vérifier les modifications
git diff

# 7. Ajouter les fichiers
git add .

# 8. Créer le commit
git commit -m "fix: update dependency"

# 9. Envoyer vers GitLab
git push

# 10. Vérifier les derniers commits
git log --oneline -3

# 11. Créer le tag
git tag -a <tag> -m "Release <tag>"

# 12. Envoyer le tag
git push origin <tag>
```

Le pipeline GitLab peut ensuite utiliser ce tag pour construire et déployer l'application.

---

# 19. Exemple concret : correction d'une dépendance

Exemple avec une dépendance transitive npm.

Avant :

```text
@commitlint/cli
└── @commitlint/load
    └── cosmiconfig
        └── js-yaml@4.1.1
```

Mise à jour :

```bash
npm update js-yaml
```

Vérification :

```bash
npm ls js-yaml
```

Si la version corrigée est installée :

```bash
git diff -- package-lock.json
```

Puis :

```bash
git add package-lock.json
git commit -m "fix: update js-yaml"
git push
```

Avant une livraison par tag, vérifier que le lockfile fonctionne :

```bash
npm ci
npm ls js-yaml
```

Puis créer le tag :

```bash
git tag -a <tag> -m "Release <tag>"
git push origin <tag>
```

---

# 20. Commandes de secours

## Annuler un merge en cours

```bash
git merge --abort
```

## Annuler un rebase en cours

```bash
git rebase --abort
```

## Voir l'état exact du dépôt

```bash
git status
```

## Retrouver des commits ou états précédents

```bash
git reflog
```

`git reflog` est particulièrement utile lorsqu'une opération Git a déplacé ou supprimé une référence et que tu veux retrouver l'ancien état.

---

# 21. Les commandes à mémoriser en priorité

Pour un usage professionnel quotidien :

```bash
git status
git branch --show-current
git switch <branche>
git switch -c <nouvelle-branche>
git fetch origin
git pull --rebase
git add .
git commit -m "message"
git push
git diff
git log --oneline --graph --decorate --all
git restore
git restore --staged
git rebase
git rebase --abort
git revert
git tag
git push origin <tag>
git reflog
```

---

# 22. Workflow mental à retenir

```text
                         ┌──────────────┐
                         │    develop   │
                         └──────┬───────┘
                                │
                       switch -c US_xxx
                                │
                                ▼
                       ┌────────────────┐
                       │  Feature       │
                       │  Development   │
                       └───────┬────────┘
                               │
                         git add / commit
                               │
                               ▼
                           git push
                               │
                               ▼
                        ┌──────────────┐
                        │ Merge Request│
                        └──────┬───────┘
                               │
                               ▼
                            develop
                               │
                               ▼
                              TAG
                               │
                               ▼
                         GitLab CI/CD
                               │
                               ▼
                            PREPROD
```

---

# 23. Règles pratiques

### Avant de commencer à travailler

```bash
git fetch origin
git status
```

### Avant de pousser

```bash
git status
git diff
git log --oneline -3
```

### Si le push est refusé

```bash
git fetch origin
git pull --rebase origin <branche>
git push
```

### Si un conflit apparaît

```bash
git status
# Corriger les fichiers
git add <fichier>
git rebase --continue
```

### Si tu veux abandonner le rebase

```bash
git rebase --abort
```

### Sur une branche partagée

Préférer :

```bash
git revert <commit>
```

plutôt que de réécrire l'historique avec `reset` + `push --force`.

### Pour un rebase d'une branche personnelle

Préférer :

```bash
git push --force-with-lease
```

plutôt que :

```bash
git push --force
```

---

# Cheat Sheet

```bash
# État
git status
git branch --show-current

# Branches
git branch
git switch develop
git switch -c US_xxx

# Remote
git fetch origin
git pull --rebase origin <branche>

# Modifications
git diff
git add .
git restore <fichier>
git restore --staged <fichier>

# Commits
git commit -m "message"
git commit --amend --no-edit
git log --oneline --graph --decorate --all
git show <commit>

# Synchronisation
git push
git push -u origin <branche>
git push --force-with-lease

# Conflits
git status
git rebase --continue
git rebase --abort
git merge --abort

# Annulation
git revert <commit>

# Tags
git tag
git tag -a <tag> -m "Release <tag>"
git push origin <tag>
git tag -d <tag>
git push origin --delete <tag>

# Secours
git reflog
```
