### -L : liste les règles (--line-numbers : numéros de règles)
### -t : type (NAT...)

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

### la diffrence entre ajout et inserer => ajout regle dés le debut du table or insere une regle n'importe quelle position du table
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

Lister les ports en écoute à distance :
```bash
nmap -PM ip_machine
```


