### l'interet de redireget les paquets , modifier les en-tete et filtrage par ip ,par ports ,par protocole(ICMP,UDP,TCP),NAT et Net filter
```bash
sudo apt-get install iptables
```
### il faut savoir que iptables fonction par table => il trois tables de fonctionnement et dans ces tables on ajoute des regles de gestion sont ordonnées (son prises dans un ordre preçis) ==> vous allez filtré un traffi entrant soit pour votre machine local ,soit pour redireget ce traffic à ailleur .
### la 1ere table c'est le NAT il s'occupe de translation de port et du IP . on a le DNAT (IPd) SNAT(IPs) MASQUERADE (simule une gateway)
### le 2eme table c'est le filter comopse de trois chaines ==>INPUT ,OUTPUT et FORWARD.Et quatre targets DROP(refus brut des paquets sans notifier),ACCEPT (accepte les paquets) ,REJECT(rejet avec notifier) et finallement DENY et puis on a log (mettre un trace sur les paquets sortant et entrant )
### le 3eme table c'est le mangle (fait la modification des paquets) compose de cing targets TOS (TypeOfService) ,TTL(TimeToLive) MARK(des etiquettes sur chaque paquets),SECMARK(etiquettes de securite) et CONNSECMARK(copie en cas de securite pour detecte les comportents anormaux) 
