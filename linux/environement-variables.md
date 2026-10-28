
### Variables d’environnement les plus importantes 
* Variables système / utilisateur
<!--StartFragment--> 

Variable | Rôle
-- | --
$USER | utilisateur courant
$HOME | répertoire personnel
$PWD | dossier courant
$PATH | chemins des exécutables
$SHELL | shell utilisé
$LANG | langue / locale
$TERM | type de terminal
$HOSTNAME | nom de la machine
<!--EndFragment-->    

* Variables développeur / runtime
<!--StartFragment--> 
Variable | Usage
-- | --
$JAVA_HOME | JDK
$SPRING_PROFILES_ACTIVE | profil Spring
$NODE_ENV | environnement Node
$MAVEN_HOME | Maven
$KUBECONFIG | cluster Kubernetes
$EDITOR | éditeur par défaut
<!--EndFragment-->

### Lire et afficher les variables
```batch
#Liste toutes les variables d’environnement
printenv

# Variante
env
```

* Variable spécifique :
```batch
echo $HOME
echo $PATH

```
### Créer une variable d’environnement
* Temporaire (session courante)
```batch
export MY_VAR=ma_valeur

```
> Disponible:  
> dans le terminal courant  
> dans les processus enfants  
> ❌ Disparaît à la fermeture du terminal  

* Persistante (utilisateur)  
Dans les fichiers cachés qui se trouve dans home ~/.bashrc ou ~/.profile :  
```batch
# Ouvrir  ~/.bashrc
nano ~/.bashrc

# Ajouter une variable (Coller la ligne ci-dessous dans le nano)
export MY_VAR=ma_valeur

# Appliquer sans redémarrer le terminal
source ~/.bashrc

# Afficher la variable
echo $MY_VAR

```
>  Rôle de ~/.bashrc et ~/.profile :   
> > Ces fichiers sont des scripts exécutés automatiquement par Linux   
> >  
> > ~/.bashrc   
> > ➜ exécuté à chaque ouverture de terminal  
> > ➜ À mettre dedans: Variables globales utilisateur, Initialisation de l’environnement, PATH, JAVA_HOME, LANG   
> > Exemple:  
> > ```batch
> > export JAVA_HOME=/usr/lib/jvm/java-17  
> > export PATH=$PATH:$JAVA_HOME/bin   
> > ```
> > ~/.profile    
> > ➜ exécuté à la connexion utilisateur  
> > ➜ À mettre dedans: Aliases, Fonctions, Variables liées au shell, Prompt  
> > Exemple:  
> > ```batch
> > alias ll='ls -lh'  
> > export EDITOR=nano  
> > ```
> > 👉 En y mettant une variable, elle sera recréée automatiquement.  
> > .profile → environnement utilisateur
> > .bashrc → confort du shell  
  
* Variables système globales & $PATH  
 1. /etc/environment :  est un fichier système lu : au login, pour tous les utilisateurs indépendamment du shell (bash, sh, zsh…)  

👉 Ce n’est PAS un script, juste des paires clé=valeur.
```batch
JAVA_HOME=/usr/lib/jvm/java-17
SPRING_PROFILES_ACTIVE=prod
```
> Ces variables sont :  
> - disponibles pour tous les users  
> - accessibles par les applications  
> - persistantes après reboot  
  
> Quand l’utiliser ?  
> ✔️ Outils système  
> ✔️ Variables communes  
> ✔️ Serveurs multi-utilisateurs  
> ✔️ Services (Java, Docker, agents)   
  
 2. $PATH — la variable la plus critique  
Quand on tape une commande comme ls, Linux cherche un exécutable, son chemin se trouve dans $PATH.  
    
* Ajouter un outil au PATH :  
```batch 
Tu installes un outil dans :
/opt/mytool/bin/mytool

# Sans PATH
/opt/mytool/bin/mytool

#Avec PATH 
mytool

Ajout dans PATH
export PATH=$PATH:/opt/mytool/bin

# Attention risque d'ecraser la PHATH !!!!!!!!!
export PATH=/opt/mytool/bin

```

> Bonnes pratiques serveur  
> 1. Toujours ajouter au PATH ($PATH:...)  
> 2. Modifier le PATH :  
>  - .profile (user)  
>  - /etc/environment (global)  
>   
> Tester avant de fermer la session

* Cas DevOps réels
```batch
JAVA_HOME=/usr/lib/jvm/java-17
PATH=$PATH:$JAVA_HOME/bin
```