# Raccourcis :
Maj+Ctrl+L : Clear console, ou bien écrire la commande : clear  
Maj+Ctrl+C : Copier  
Maj+Ctrl+V : Coller  
Maj+Ctrl+F  

# Répertoires essentiels
<!--StartFragment-->
Dossier | Rôle | Cas réel
-- | -- | --
/ | Racine | Point de départ de tout
/home | Dossiers utilisateurs | Ton code, scripts
/etc | Configurations système | Spring, Nginx, Docker
/var | Données variables | Logs, cache
/usr | Programmes système | Java, Git
/opt | Logiciels optionnels | Apps custom
/bin | Commandes essentielles | ls, cp, bash
/sbin | Admin système | systemctl
/tmp | Fichiers temporaires | Nettoyé automatiquement
<!--EndFragment-->

# Navigation dans le système > Commandes essentielles  
<!--StartFragment-->
Commande| Explication
-- | -- 
**pwd**| affiche le répertoire courant 
**ls -l**| Liste détaillée 
**ls -a**| Inclut les fichiers cachés
**cd /etc**|Change de repertoire
**cd ..**|Remonte d'un niveau
**cd ~**|Retour au Home
<!--EndFragment-->

# Créer et manipuler des fichiers / dossiers
```bash
touch file.txt          # créer un fichier vide
mkdir test              # créer un dossier
mkdir -p a/b/c          # créer arborescence
cp file.txt file2.txt   # copier
mv file2.txt test/      # déplacer / renommer
rm file.txt             # supprimer fichier
rm -r test/             # supprimer dossier
`````
# Lire le contenu des fichiers
```bash
cat file.txt        # affiche tout (petits fichiers)
less file.txt       # lecture interactive (recommandé)
head file.txt       # premières lignes
tail file.txt       # dernières lignes
tail -f app.log     # suivi en temps réel (logs)
`````

<!--StartFragment-->
# Les flux Linux  
Chaque commande Linux utilise 3 flux:  
Flux | Nom | Description
-- | -- | --
0 | stdin | entrée
1 | stdout | sortie normale
2 | stderr | erreurs
<!--EndFragment-->

> Par défaut :  
> * clavier → stdin  
> * écran → stdout / stderr  

# Redirections (>, >>, <)
```bash
ls > file.txt       # écrase le fichier

# Rediriger les résultats de la commande dans un fichier
command > error.txt 
cat app.log | grep ERROR > errors.log

ls >> file.txt      # ajoute à la fin
wc -l < file.txt    # Le nombre de lignes de fichier file.txt (Le fichier devient l’entrée de la commande)
`````
# Pipes  
Le pipe envoie la sortie d’une commande vers l’entrée de la suivante.  
```bash
commande1 | commande2
`````
```bash
ls | grep log              # Afficher les fichier qui ont un nom qui contient log
cat app.log | grep log     # Afficher les lignes qui contiennent le mot log
`````
# Modifier un fichier sous Linux  
* Modifier rapidement
```bash
echo "Nouvelle ligne" >> file.txt     #Pas interactif, mais rapide
`````
* nano
```bash
nano file.txt          # Modifier directement | CTRL + O → enregistrer | CTRL + X → quitter
`````
* Vi/Vim
```bash
vi file.txt
`````

> Modes :  
> i → insertion  
> ESC → commande  
> :w → enregistrer  
> :q → quitter  
> :wq → enregistrer & quitter  
> :q! → quitter sans sauvegarder  

* Modifier via redirection
```bash
echo "test" > file.txt
`````
# Consulter un fichier sous Linux
###  Affichage brut  
```bash
cat file.txt
`````
> À utiliser quand :  
> Fichier très petit  
> Besoin d’un affichage rapide    
   
###  Lecture interactive Less
```batch
less file.txt
`````
> À utiliser quand:  
> Fichier moyen ou gros  
> Logs de production  
> Besoin de rechercher / naviguer  
> lire une sortie de commande sans inonder le terminal  

* Navigation:  
<!--StartFragment-->
Touche | Action
-- | --
↓ / ↑ | ligne par ligne
PageDown / PageUp | page par page
Space | page suivante
G | aller à la fin
g | aller au début
q | Quitter
<!--EndFragment-->  

* Recherche:  
<!--StartFragment-->
Touche | Action
-- | --
/mot | chercher mot
n | occurrence suivante
N | occurrence précédente
<!--EndFragment-->

* Utilisation avancé :   
a) Afficher les numéros de ligne :  
```batch
less -N file.txt
`````

b) Suivre un fichier (comme tail -f)   
```batch
less +F file.txt

# CTRL + C → quitter le suivi
`````

c) less avec pipes
```batch
kubectl logs pod | less
ps aux | grep java | less
ps aux | less

# ps aux
# → affiche tous les processus du système
# |
# → envoie la sortie vers une autre commande
# less
# → permet de consulter cette sortie propremen
`````

### head – Début du fichier
```batch
head file.txt
head -n 20 file.txt
```
> À utiliser quand: 
> Vérifier le format  
> Lire les premières lignes (CSV, config)  

### tail – Fin du fichier
```batch
tail file.log
tail -n 20 file.log
`````
> À utiliser quand:  
> Lire les dernières lignes  
> Logs récents  

### tail -f – Suivi en temps réel (PROD)  
```batch
tail -f file.log
tail -f file.log | grep ERROR
`````
> À utiliser quand :  
> Application en cours d’exécution  
> Monitoring live  

###  Avec filtrage (grep)
```batch
grep ERROR file.log
`````
> À utiliser quand :  
> Chercher une info précise  
> Analyser des logs  
