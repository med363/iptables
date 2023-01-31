### -L : liste les règles (--line-numbers : numéros de règles)
### -t : type (NAT...)
##### Tcpdump is a command line utility that allows you to capture and analyze network traffic going through your system.

Actions sur les chaines : INPUT / OUTPUT / FORWARD
en majuscules


-A : ajout de règle à une chaine (-A INPUT)


-D : suppression de règle (-D INPUT 1 - numéro de la règle dans la chaine INPUT )


-R : remplace la règle (-R INPUT)


-I : insertion d'une règle (sans chiffre au début de la chaine (ex: INPUT 1)


-F : flush les règles pour une chaine (-F INPUT)


-N : création de chaîne


-X : drop de chaine


-P : définition de la policy d'une chaine (par défaut - ex: -P INPUT DROP)

### la diffrence entre ajout et inserer => ajout regle à la du table or insere une regle n'importe quelle position du table
-> Caractéristiques  <-


-p : protocole (-p tcp)


-s : la source (ip, réseau)


-j : action à faire (DROP/ACCEPT)


-d : la destination (ip, réseau)


-i : interface d'entrée (eth0...)


-o : interface de sortie


--sport 80 : un port


-m multiport --sport 80,443 : plusieurs ports


-t : type (NAT...)

### gerer les ping ICMP
Empêcher les pings en entrée


  +------------+        +------------+
  | 172.17.0.1 +--------> 172.17.0.2 |
  +------------+        +------------+  
                        x 
                        |



ping = protocole icmp
on travaille sur 172.17.0.2

```bash 
iptables -I INPUT -p icmp --icmp-type 8 -j DROP
```
### Listons les règles:
```bash
iptables -L
```
### Rq :
#type 8 =  "Echo Request" pour le ping réseau
# -I : ajoute les règles au début de la table

### - suppression des règles
```bash 
iptables -F
```
### Identifying default ICMP types
The following table lists the default ICMP types:

ICMP types

ICMP Type

Message

0 Echo reply

3 Destination unreachable

4 Source quench

5 Redirect

8  Echo

9 Router advertisement

10 Router selection

11  Time exceeded

12 Parameter problem

13 Timestamp

14 Timestamp reply

15 Information request

16 Information reply

17 Address mask request

18  Address mask reply

30 Traceroute

Empêcher les pings en sortie


  +------------+        +------------+
  | 172.17.0.1 +--------> 172.17.0.2 |
  +------------+        +------------+  
               x 
               |



on travaille sur 172.17.0.1

```bash
iptables -I OUTPUT -p icmp -j DROP
```
==> -j : c'est quoi l'action 
```bash
iptables -L
```



## Supprimer une règle précise

```bash
iptables -L --line-numbers

iptables -D OUPUT "num_ligne"
```



### outils utiles pour faire des tests :
#### pour test iptable il faut avoir mini app ecout sur un port particulier pour pouvoir travail sur des ports et des services.
Mettre en place une socket écoutant sur le port 8000 :
### install socket mini serveur web python
```bash
python -m SimpleHTTPServer 8000
```


Lister les ports en écoute à distance :
```bash
nmap -PM ip_machine
```


Test ouverture :
### verifier si un port est ouvert Et avec netcat on peut fournit un fichier en entree
```bash
telnet ip_machine port

netcat ip_machine port < fichier
```





-> Travail sur les ip et ports <-
### refuser tt les trafic entrant du machine 172.7.0.3 sur le port 8000 mais accept le traffic entrant du machine 172.17.0.1


                   OK       serveur (8000)
                        +------------+
                +-------> 172.17.0.2 |
                |       +------------+
  +-------------+              ^
  | 172.17.0.1  |              |
  +-------------+              |
     client                KO  |
                        +------------+
                        | 172.17.0.3 |
                        +------------+
                            client



-> Mise en pratique <-

empêcher d'entrée pour une machine
rq:en generalement refuse tt puis on accepte cas par cas [ mais la table l'ordre croisant particulier ==> global)
1. refuse tt trafic entrant sur le port de destination(--dport) 8000
2. precis une source (-s)
```bash
iptables -I INPUT -p tcp --dport 8000 -j DROP
iptables -I INPUT -s 172.17.0.1 -p tcp -dport 8000 -J ACCEPT
```

supprime regle
```bash
iptables -D INPUT <<nbre>>
```



empêcher d'aller vers une ip et un port

```bash
iptables -I OUTPUT -d 172.17.0.2 -p tcp --dport 8000 -j DROP
```



et on supprimer la règle

```bash
iptables -L --line-numbers

iptables -D OUTPUT num_ligne
```


supprimer les regles dans tt les tables
```bash
iptables -t filter -F
```


> Redéfinition des tables <-

Refuser toutes les connexions par défaut:

```bash
iptables -t filter -P INPUT DROP
iptables -t filter -P FORWARD DROP
iptables -t filter -P OUTPUT DROP
```

Rq : -P = par défaut

Si la machine établie une connexion on laisse passer la communication
on accepter la cnx dans les deux sens
```bash
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
```


Rq : -A = ajout de la règle à la fin (les règles sont prises dans l'ordre)


> Redéfinition des tables <-

on accepte les pings
pour le monitoring
```bash
iptables -t filter -A INPUT -p icmp -j ACCEPT
iptables -t filter -A OUTPUT -p icmp -j ACCEPT
```



Option : ssh sur le 2222 (conf ssh) et on autorise celui-ci (attention)

```bash
iptables -t filter -A INPUT -p tcp --dport 2222 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 2222 -j ACCEPT
```

-> Et bien d'autres règles <-


suivant le type de machine (type serveur/machine perso)


Pourront s'ajouter :

port 80 : http
port 443 : https
port 25 : smtp
port 587 : smtps
port 53 : dns
port 20 : ftp
port 990 : ftps
port 110 : pop3
port 995 : pop3s
port 143 : imap


> ipTables comme load-balancer ?!?! <-
iptables(faire routing || FORWARD ||
couche base : osi 4 (transport)


                                      +-----------+
                                    +-> 172.17.0.3|
 +-------------+    +-------------+ | +-----------+
 |  172.17.0.1 +-----> 172.17.0.2 +-+
 +-------------+    +-------------+ | +-----------+
    client             router       +-> 172.17.0.4|
                                      +-----------+



-> Au taff !!! <-

Requête de départ à acheminer sur 172.17.0 3-4

```bash
echo "toto" | nc 172.17.0.2 5000
```



Configuration du router

Pour gérer les paquets à l'aller:
etablir le 1ere route pour .3 
prerouting : avant fw
-t nat(table de nat)
-p (protocole)
-d (destination l'orgine de paquet)
DNAT (je vveut  modifier destinataire de ce paquet et le transmettre ailleurs)
```bash
iptables -A PREROUTING -t nat -p tcp -d 172.17.0.2 --dport 5000 -j DNAT --to-destination 172.17.0.3:5000
```
analyse 
```bash
tcpdup -i ens33 -n src port 5000
```


Pour prévoir le retour:
```bash
iptables -A POSTROUTING -t nat -p tcp -d 172.17.0.3 --dport 5000 -j SNAT --to-source 172.17.0.2
```



-> Et le load-balancing ??? <-
Pour simplifier on choisir le Round Robin (chacun son tour) :

- un paquet sur deux on skip


```bash
iptables -A PREROUTING -t nat -p tcp -d 172.17.0.2 -m statistic --mode nth --every 2 --packet 0 --dport 5000 -j DNAT --to-destination 172.17.0.3:5000
```



- l'autre route récupère les paquets restants


```bash
iptables -A PREROUTING -t nat -p tcp -d 172.17.0.2 --dport 5000 -j DNAT --to-destination     172.17.0.4:5000
```



