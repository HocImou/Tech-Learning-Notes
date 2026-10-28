# Processus & services Linux
## Processus Linux  
Un processus est, Un programme en cours d’exécution, avec :  
 un PID (Process ID)    
 un parent (PPID)    
 un environnement  
 des droits  
 un état  
    
> Quand tu ouvres un terminal, un shell (bash) démarre, ce shell est un processus
> chaque commande lance un processus enfant. Un processus enfant hérite des variables d’environnement du parent.
    
Exemple :  
```batch
java -jar app.jar
```
> Linux :  
> 1. charge le binaire java  
> 2. crée un processus  
> 3. lui donne un PID  
> 4. lui transmet l’environnement (PATH, JAVA_HOME, etc.)  

<br>  
    
### Lister les processus  
```batch
# Affiche les processus du terminal courant uniquement.
ps
```
```batch
ps aux

# Résultats
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.2  0.2 225332  9052 ?        Ss   09:05   0:02 /sbin/init spla

Colonnes importantes :
> USER : propriétaire
> PID : identifiant du processus
> %CPU / %MEM : ressources
> COMMAND : commande lancée

```
> Explication :  
> a → processus de tous les utilisateurs  
> u → affichage détaillé (user, CPU, mémoire)   
> x → processus sans terminal (services, daemons)  

> > 👉 C’est la commande de base pour debugger un service.  

 <br>  

### Rechercher   
```batch
ps aux | grep java
```  
   <br>  
 <br>  
  
### En temp réel  
```batch
top
```
> Vue temps réel des processus :
>  - CPU
>  - RAM
>  - état (running, sleeping…)  
    
> > Très utile pour :  voir un process Java qui consomme trop et diagnostiquer un batch bloqué  

 <br>     
   
### États d’un processus
<!--StartFragment-->

État | Signification
-- | --
R | Running
S | Sleeping
D | Waiting IO
Z | Zombie (⚠️)
T | Stopped

> 👉 Un zombie = processus terminé mais non récupéré par le parent  
> 👉 Mauvaise gestion côté application ou script
<!--EndFragment-->  
<br>  
<br>  
    
### Hiérarchie des processus (parent / enfant)
```batch
ps -ef
```  
> -e → tous les processus  
> -f → format complet  
> Regarde : PID, PPID   

<br>  
<br>  
    
### Tuer un process  
```batch
kill <PID_parent>

```

> Si on tue le parent les enfants meurent (sauf exceptions).  

<br>  
<br>      

## Services Linux (daemons)

Processus vs service
<!--StartFragment-->
Processus | Service
-- | --
Manuel | Automatique
Lié à un terminal | En arrière-plan
Temporaire | Persistant
Utilisateur | Systèm
<!--EndFragment-->  
  
<br>  
  
* Exemples de services :  
ssh  
nginx  
docker  
postgresql  
backend Spring Boot  
<br>  
    
### systemd : le chef d’orchestre  
systemd est : Le gestionnaire de services et processus système  
Il :  
démarre les services au boot  
surveille leur état  
les redémarre si crash  
gère leur environnement  

<br>  
  
### La commande systemctl  
Interface principale pour parler à systemd.  
<br>  
  
### Vérifier l’état d’un service
```batch
systemctl status ssh
```
> Détail :  
> actif / inactif  
> PID du service  
> logs récents  
> erreurs éventuelles  

> > C’est LA commande réflexe quand “ça ne marche pas”.  

<br>  
   
### Démarrer/Arrêter un service  
```batch
# Démarrer nginx
systemctl start nginx

# Arrêter nginx
systemctl stop nginx
```  
<br>  
   
### Activer/ désactiver un service au démarrage  
```batch
systemctl enable nginx

systemctl disable nginx

```
<br>  
  
* Un service systemd = un environnement contrôlé
Un service n’hérite PAS de ton .bashrc ou .profile. Pas de shell interactif et un environnement minimal et sécurisé.

<br>  
  
### Voir l’environnement d’un service 
```batch
systemctl show my-service
# ou bien
systemctl show my-service | grep Environment
```

<br>  

### Kill proprement un processus
```batch
kill PID
# arrêt brutal
kill -9 PID
```

<br>  
  
### définir des variables pour un service  
Un service systemd :  
❌ n’utilise PAS ~/.bashrc  
❌ n’utilise PAS ~/.profile  
❌ n’hérite PAS de shell  

> 👉 Il démarre dans un environnement isolé et sécurisé  
> > Donc si une application a besoin de variables 👉 Il faudra les définir dans systemd  
<br>  
  
### Où définir les variables pour un service ?  
#### 1. Méthode 1 — Directement dans le fichier .service (la plus courante)  
Exemple : /etc/systemd/system/my-app.service  
```batch
Description=Application Spring Boot
After=network.target

[Service]
User=appuser
WorkingDirectory=/opt/my-app

# Variables d'environnement---------------------
Environment="JAVA_HOME=/usr/lib/jvm/java-17"    |
Environment="SPRING_PROFILES_ACTIVE=prod"       |
Environment="DB_HOST=localhost"                 |
#------------------------------------------------

ExecStart=/usr/bin/java -jar app.jar
Restart=always

[Install]
WantedBy=multi-user.target
```
<br>  
  
* Appliquer les changements  
```batch
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart my-app
```
> 📌 Obligatoire sinon systemd ignore les changements  

<br>  
  
#### 2. Méthode 2 — Fichier d’environnement externe (best practice prod)  — EnvironmentFile  
[ 1 service = 1 fichier .service + 1 fichier .env ]  
- fichier .service:   
décrit comment démarrer le service, il a une structure stable   
Ce fichier est : versionné, auditable, stable dans le temps  
Exemple : 
```batch
[Service]
User=appuser

# EnvironmentFile = injecter une configuration dans un service, pas dans le système
EnvironmentFile=/etc/my-app/my-app.env
EnvironmentFile=/etc/my-app/my-app-secrets.env
WorkingDirectory=/opt/my-app
ExecStart=/usr/bin/java -jar app.jar
Restart=always
```
- fichier .env:   
contient la configuration métier et technique, change selon l’environnement  
Ce fichier est : différent en dev / recette / prod, souvent non versionné, géré par DevOps / CI  
Exemple :   
```batch
SPRING_PROFILES_ACTIVE=prod
DB_HOST=db-prod
DB_PORT=5432
JAVA_OPTS=-Xms512m -Xmx1024m

```
> Règles importantes :
> > VAR=valeur  
> > pas de export  
> > pas de quotes nécessaires  
> > pas d’espaces autour du =  
```batch

 APP_ENV = production # ❌ Mauvais

 APP_ENV=production  # ✅ Bon

```
📌 Remarque  
On pourrais tout à fait avoir plusieurs fichiers env :  
```batch
EnvironmentFile=/etc/my-app/base.env
EnvironmentFile=/etc/my-app/prod.env
EnvironmentFile=/etc/my-app/secrets.env
```
> ➡️ systemd les lit dans l’ordre  
> ➡️ une variable peut être écrasée  
<br>  
  

Cas réel:  
/etc/my-app/  
 ├── my-app.service  
 ├── my-app.env  
 └── secrets.env (permissions strictes)  

> 🔐 Sécurité (très important)  
> > Les fichiers .env peuvent avoir des droits :  chmod 600 my-app.env
> > Le .service est souvent lisible par tous  
> > Meilleure gestion des secrets  

> 4️⃣ Audit & conformité  
> > Le même .service sur tous les environnements  
> > seules les variables changent  
> > traçabilité claire  

<br>  
  
#### 3. Méthode 3 — Override systemd (sans toucher au fichier):  
paquets installés via apt  
```batch
sudo systemctl edit docker
# Cela ouvre un fichier override :
[Service]
Environment="HTTP_PROXY=http://proxy:8080"
```
> ✔️ Très utilisé en infra / ops
> ❌ Rarement utilisé par les devs seuls
> 📌 systemd fusionne avec le fichier original. 👉 Très important en environnement contrôlé.
  
<br>
  
* PATH et services (point critique sécurité)
```batch
# Par défaut :
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin

# ❌ Risque :mauvais binaire, fail au démarrage
ExecStart=java -jar app.jar

#Toujours :
ExecStart=/usr/bin/java -jar app.jar
```

#### Différence Environment= vs EnvironmentFile= 
- Environment=
```batch
Environment="JAVA_HOME=/usr/lib/jvm/java-17"

```
> Caractéristiques :  
> > défini directement dans le .service  
> > lisible par tous  
> > rapide  
> > peu flexible  

> Usage recommandé :  
> > variables techniques  
> > stables  
> > non sensibles  
- EnvironmentFile=  
```batch
EnvironmentFile=/etc/my-app/my-app.env

```
> Caractéristiques :  
> > fichier externe  
> > permissions contrôlées  
> > modifiable sans toucher au service  
> > compatible CI/CD  

> Usage recommandé :  
> > configuration métier  
> > secrets  
> > profils  
> > ports  
<br>  
  
### Comment debugger une variable absente  
* Étape 1 — Vérifier le fichier : 
```batch
sudo cat /etc/my-app/my-app.env

``` 
> 👉 Attention à :  
> espaces  
> fautes de frappe  
> = mal placé  
* Étape 2 — Vérifier que systemd la voit  
```batch
systemctl show my-app | grep Environment

``` 
* Étape 3 — Vérifier dans le processus  
```batch
#Récuperer le MainPID
sudo systemctl show my-app --property=MainPID

# Afficher l'environnement réel
sudo cat /proc/<PID>/environ | tr '\0' '\n'

``` 
* Étape 4 — Vérifier les logs  
```batch
journalctl -u my-app -xe
```
### systemd vs Docker vs Kubernetes
<!--StartFragment-->
systemd | Docker | Kubernetes
-- | -- | --
.service | Dockerfile | Deployment
Environment | ENV | env
EnvironmentFile | --env-file | ConfigMap
secrets.env | --secret | Secret

<!--EndFragment-->

* Exemple Docker :  
```batch
docker run --env-file my-app.env my-image
# identique à 
EnvironmentFile=my-app.env
```
* Exemple Kubernetes  
```batch
envFrom:
  - configMapRef:
      name: my-app-config
  - secretRef:
      name: my-app-secrets

# Identique à plusieurs EnvironmentFile
```