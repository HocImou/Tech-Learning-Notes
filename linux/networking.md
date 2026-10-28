# Linux Networking — Fondamentaux (Partie 1)  

🎯 Pourquoi le réseau Linux est fondamental. Tout ce qui suit dépend du réseau :  

- applications backend (Spring Boot)  
- microservices  
- Docker  
- Kubernetes  
- debugging prod (timeouts, services injoignables)  

> 👉 Docker et Kubernetes n’inventent rien  
> 👉 Ils exploitent le réseau Linux  

## Les briques fondamentales du réseau  

Dans Linux, tout repose sur 4 concepts :  

1-Interface réseau  
2-Adresse IP  
3-Port  
4-Processus (application)  

### 1. Interface réseau  

Une interface réseau est le point d’entrée/sortie réseau d’une machine.  

Exemples :  

* eth0 → carte réseau Ethernet  

* wlan0 → Wi-Fi  

* lo → loopback (localhost)  

* docker0 → réseau Docker  

Voir les interfaces :  
```batch
# Liste toutes les interfaces réseau
ip addr

# Ou bien
ip link

# Exemple de résultat:
2: eth0: <UP>
    inet 192.168.1.20/24
```
> 🧠 À retenir :  
> sans interface UP, aucune communication possible  
> une interface ≠ une application  

<br> 
  
### 2. Adresse IP  

L’adresse IP identifie une machine ou une interface sur un réseau, et le masque détermine quelle partie représente le réseau et quelle partie la machine. Par exemple, 172.17.0.2/16 indique qu’un conteneur appartient à un réseau 172.17.0.0/16..  

Une interface peut avoir plusieurs IP  
```text
eth0:
  192.168.1.10 → dev
  192.168.1.20 → preprod
  192.168.1.30 → prod

# 👉 Très courant en environnement serveur
# 👉 Une seule interface physique, plusieurs contextes logiques
```
Ajouter une IP (exemple) : 
```batch
# Ajoute une IP supplémentaire à eth0
sudo ip addr add 192.168.1.30/24 dev eth0
```
<br>
  
### 3. Port
<html>
<body>
<!--StartFragment-->
<p data-start="1988" data-end="2036">Un <strong data-start="1991" data-end="1999">port</strong> identifie un <strong data-start="2013" data-end="2024">service</strong> sur une IP.</p>


Élément | Rôle  
-- | --
IP | Machine
Port | Application

</div></div>
<p data-start="2112" data-end="2122">Exemples :</p>
<ul data-start="2123" data-end="2197">
<li data-start="2123" data-end="2136">
<p data-start="2125" data-end="2136"><code data-start="2125" data-end="2129">80</code> → HTTP</p>
</li>
<li data-start="2137" data-end="2152">
<p data-start="2139" data-end="2152"><code data-start="2139" data-end="2144">443</code> → HTTPS</p>
</li>
<li data-start="2153" data-end="2175">
<p data-start="2155" data-end="2175"><code data-start="2155" data-end="2161">8080</code> → Spring Boot</p>
</li>
<li data-start="2176" data-end="2197">
<p data-start="2178" data-end="2197"><code data-start="2178" data-end="2184">5432</code> → PostgreSQL</p>
</li>
</ul>
<!--EndFragment-->
</body>
</html>
<br>
  
### 4. Processus (application)  
Une application est un processus Linux.  

Exemples :  

Java → Spring Boot  

Nginx  

PostgreSQL  

Voir les processus réseau :  
```batch
# Voir qui écoute sur quels ports
ss -tuln
```
<br>
  
### Comment une requête arrive à l'application  
```batch
curl http://192.168.1.20:8080
```
Le chemin est :  

1. Le réseau trouve l’IP 192.168.1.20  

2. Le paquet arrive sur l’interface eth0  

3. Linux voit le port 8080  

4. Linux transmet au processus Java  

5. Spring Boot répond  

> 🧠 L’interface reçoit  
> 🧠 Le port trie  
> 🧠 L’application traite  

<br>
  
Exemple concret : Spring Boot + Interface :  
```text
# Configuration Spring Boot
server:
  port: 8080
```
Par défaut :  
```text
# 👉 L’application écoute sur toutes les interfaces
0.0.0.0:8080
```
### Restreindre une application à une IP  
```text
server:
  address: 192.168.1.20
  port: 8080
```
Vérification :  
```batch
ss -tuln | grep 8080
```
Résultat :
```batch
192.168.1.20:8080
```
> ⚠️ Si l’interface tombe, l’application tourne toujours mais elle devient injoignable
> ```batch
> sudo ip link set eth0 down
> ```

<br>  
Résumé :   

```batch  
[ Client ]  
    |  
[ Interface ]  
    |  
[ IP ]  
    |  
[ Port ]  
    |  
[ Processus ]  
```

> ✅ À retenir absolument:  
> * Une interface peut avoir plusieurs IP  
> * Une application écoute sur une IP + port  
> * Linux fait le routage  
> * Docker / Kubernetes reposent sur ce modèle  

# Linux Networking — DNS & Résolution de noms (Partie 2)