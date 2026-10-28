# Permissions & Utilisateurs
###  Définitions  
User (utilisateur)
 - Un **user** représente une personne ou un service
 - Chaque fichier a **un seul propriétaire**
 - Le propriétaire contrôle principalement le fichier

Group (groupe)
 - Un **group** est un **ensemble d’utilisateurs**
 - Un fichier a **un seul groupe associé**
 - Tous les membres du groupe partagent les mêmes droits  

### Comprendre les permissions
```batch
ls -l

# Résultat:
-rw-r--r-- 1 user dev 2456 fichier.txt

- rw- r-- r--
│  │   │   │
│  │   │   └── autres (others)
│  │   └────── groupe (group)
│  └────────── propriétaire (user)
└───────────── type de fichier
```
Types de fichiers
 - `-` : fichier classique
 - `d` : dossier
 - `l` : lien symbolique  

Droits et leur signification

| Droit | Fichier | Dossier |
|-----|--------|--------|
| r | lire le contenu | lister le dossier |
| w | modifier | créer / supprimer |
| x | exécuter | entrer dans le dossier |

⚠️ **Sans `x` sur un dossier, impossible d’y accéder**

### Modifier les permissions (`chmod`)
* Mode symbolique :
```batch
# Ajouter le droit de lecture au propriétaire
chemod u+r file.txt

#Ajouter le droit d'ecriture au groupe
chmod g+w file.txt

# Retirer le droit de lecture pour les autres utilisateurs (Others)
chmod o-r file.txt
```
* Mode numérique :  
Les permissions sont définies par 3 chiffres :  
            [user][group][others]  
Chaque chiffre est la somme des droits accordés.   
Valeur de chaque droit :  

<!--StartFragment-->
Droit | Lettre | Valeur
-- | -- | --
Lecture | r | 4
Écriture | w | 2
Exécution | x | 1
<!--EndFragment-->
👉 On additionne les valeurs pour obtenir le chiffre.   

Exemple 1 — chmod 755 script.sh
```batch
7   5   5
│   │   │
│   │   └── others
│   └────── group
└────────── user
```

👤 User = 7  
r (4) + w (2) + x (1) = 7
→ rwx

👥 Group = 5  
r (4) + x (1) = 5
→ r-x

🌍 Others = 5  
r (4) + x (1) = 5
→ r-x

📌 Résultat : 
rwx r-x r-x
👉 Script exécutable par tout le monde, mais modifiable uniquement par le propriétaire.

<!--StartFragment--> * Pourquoi ces valeurs sont des standards serveur

Permission | Usage
-- | --
755 | scripts, binaires
644 | fichiers config, code
700 | scripts sensibles
600 | secrets, clés
775 | dossiers partagés
777 | ❌ à éviter
<!--EndFragment-->


Astuce mentale:  

7 → tout  

6 → lire + écrire  

5 → lire + exécuter  

4 → lire  

0 → rien  

<br>  
    
* Cas développeur
```batch
# Rend un script exécutable
chmod +x script.sh

# Exécute le script
./script.sh
```
### Propriétaire & groupe (`chown`, `chgrp`)
* Changer le propriétaire  
```batch
# Changer le propriétaire d'un fichier
shown user file.txt

# Change le propriétaire ET le groupe
chown user:dev file.txt

# Change propriétaire et groupe sur tout un dossier (Changement Récursif)
chown -R user:dev dossier/
```
### Utilisateurs & groupes
```batch
# Affiche le nom de l’utilisateur courant
whoami

# Affiche l’UID, le GID et les groupes
id

# Affiche les groupes de l’utilisateur
groups

# Ajouter un utilisateur
sudo adduser userdev

# Crée un nouveau groupe nommé dev
sudo groupadd dev

# Vérifie que le groupe existe
getent group dev

# Ajoute l’utilisateur au groupe dev
sudo usermod -aG dev username
⚠️ `-aG` est **obligatoire** pour ne pas supprimer les groupes existants

# Affiche les membres du groupe docker
getent group docker

# Liste les groupes et utilisateurs associés
grep docker /etc/group
```
<br>  
    
* Réflexes Dev / DevOps

```batch
# Vérifier permissions et propriétaire
ls -l

# Vérifier son identité
id
> Le résultat : uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),999(docker)
   Interprétation :
     uid → identifiant utilisateur (user principal)
     gid → groupe principal de l’utilisateur
     groups → tous les groupes auxquels l’utilisateur appartient


# Donner accès à un groupe projet
chown -R user:dev /opt/app
chmod -R 775 /opt/app

# Donner accès Docker à un user
sudo usermod -aG docker $USER
```
# Variables d’environnement Linux
Une variable d’environnement est une clé = valeur, 
stockée en mémoire par le système
accessible par le shell, les scripts et les applications
utilisée pour configurer le comportement sans modifier le code  

👉 Exemple :
```batch
JAVA_HOME=/usr/lib/jvm/java-17
```
