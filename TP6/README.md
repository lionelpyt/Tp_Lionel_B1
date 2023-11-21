# TP6 : Un LAN maîtrisé meo ?

Dans ce TP on va mettre en oeuvre quelques services réseau, pour approfondir votre culture sur les services les plus répandus (comme DNS) et pour mettre en oeuvre le savoir sur TCP et UDP.

En vrai, après ça, vous aurez eu un bon aperçu de ce qui se fait dans la plupart des LANs du monde :

- **un routeur qui permet d'accéder à internet**
  - il agira donc comme la passerelle du réseau LAN
- **un switch/accès WiFi**
  - pour que tout le monde se connecte au routeur
  - c'est nos p'tits réseaux privé-hôte dans VirtualBox
- **un serveur DNS**
  - pour que les clients puissent résoudre des noms depuis le LAN
- **un serveur DHCP**
  - il file des IPs aux clients
  - et les informe de l'adresse IP de la passerelle du LAN
  - et aussi de l'adresse IP du serveur DNS du LAN
- **l'utilisation de TCP et UDP**
  - pour établir des connexions et faire des trucs utiles
  - une fois que le setup IP est terminé :
    - client a une IP
    - il connaît l'adresse de la passerelle
    - il connaît l'adresse d'un serveur DNS

---

![Not the network](./img/not_the_network.jpg)

## Sommaire

- [TP6 : Un LAN maîtrisé meo ?](#tp6--un-lan-maîtrisé-meo-)
  - [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
- [I. Setup routeur](#i-setup-routeur)
- [II. Serveur DNS](#ii-serveur-dns)
  - [1. Présentation](#1-présentation)
  - [2. Setup](#2-setup)
  - [3. Test](#3-test)
- [III. Serveur DHCP](#iii-serveur-dhcp)
- [IV. Bonus : Serveur Web HTTPS](#iv-bonus--serveur-web-https)

# 0. Prérequis

➜ **Pour ce TP, on va se servir de VMs Rocky Linux** quand des VMs seront nécessaires. On va en faire plusieurs, n'hésitez pas à diminuer la RAM (512Mo ou 1Go devraient suffire). Vous pouvez redescendre la mémoire vidéo aussi.  

➜ **Si vous voyez un 🦈** c'est qu'il y a une capture PCAP à produire et à mettre dans votre dépôt git de rendu

➜ **L'emoji 🖥️ indique une VM à créer**. Pour chaque VM, vous déroulerez la checklist suivante :

- [x] Créer la machine (avec une carte host-only)
- [x] Définir une IP statique à la VM
- [x] Donner un hostname à la machine
- [x] Utiliser SSH pour administrer la machine
- [x] Remplir votre fichier `hosts`, celui de votre PC, pour accéder au VM avec un nom
- [x] Dès que le routeur est en place, n'oubliez pas d'ajouter une route par défaut aux autres VM pour qu'elles aient internet
- [x] Dès que le serveur DNS est en place, n'oubliez pas de l'ajouter à la conf des autres machines à la main si nécessaire

> Toutes les commandes pour réaliser ces opérations sont dans [le mémo Rocky](../../cours/memo/rocky_network.md). Aucune de ces étapes ne doit figurer dan le rendu, c'est juste la mise en place de votre environnement de travail.

# I. Setup routeur

![Topo 1](./img/topo1.svg)

| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `10.6.1.11`        |

🖥️ **Machine `router.tp6.b1`**

- ajoutez lui aussi une carte NAT en plus de la carte host-only (privé hôte en français) pour qu'il ait un accès internet
- toutes les autres VMs du TP devront utiliser ce routeur pour accéder à internet
- n'oubliez pas d'activer le routage vers internet sur cette machine :

```
$ sudo firewall-cmd --add-masquerade --permanent
$ sudo firewall-cmd --reload
```

🖥️ **Machine `john.tp6.b1`**

- une machine qui servira de client au sein du réseau pour effectuer des tests
- suivez-bien la checklist !
- testez tout de suite avec `john` que votre routeur fonctionne et que vous avez un accès internet

# II. Serveur DNS

![Haiku DNS](./img/haiku_dns.png)

## 1. Présentation

Un serveur DNS est un serveur qui est capable de répondre à des requêtes DNS.

Une requête DNS est la requête effectuée par une machine lorsqu'elle souhaite connaître l'adresse IP d'une machine, lorsqu'elle connaît son nom.

Par exemple, si vous ouvrez un navigateur web et saisissez `https://www.ynov.com` alors une requête DNS est automatiquement effectuée par votre PC pour déterminez à quelle adresse IP correspond le nom `www.ynov.com`.

> *La partie `https://` ne fait pas partie du nom de domaine, ça indique simplement au navigateur la méthode de connexion. Ici, c'est HTTPS.*

Dans cette partie, on va monter une VM qui porte un serveur DNS. Ce dernier répondra aux autres VMs du LAN quand elles auront besoin de connaître des noms. Ainsi, ce serveur pourra :

- résoudre des noms locaux
  - vous pourrez `ping john.tp6.b1` et ça fonctionnera
  - mais aussi `ping www.ynov.com` et votre serveur DNS sera capable de le résoudre aussi

*Dans la vraie vie, il n'est pas rare qu'une entreprise gère elle-même ses noms de domaine, voire gère elle-même son serveur DNS. C'est donc du savoir très proche de ce qu'on fait dans la vraie vie dont il est question.*

> En réalité, ce n'est pas votre serveur DNS qui pourra résoudre `www.ynov.com`, mais il sera capable de *forward* (faire passer) votre requête à un autre serveur DNS qui lui, connaît la réponse.

## 2. Setup

![Topo 2](./img/topo2.svg)

| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `10.6.1.11`        |
| `dns.tp6.b1`    | `10.6.1.101`       |

🖥️ **Machine `dns.tp6.b1`**

- n'oubliez pas de dérouler la checklist

> *Sur internet ça peut partir dans tous les sens, alors je vous ai écrit un petit tuto maison cette fois hehe. **Lisez attentivement toute cette partie en entier avant de commencer les manips.** Pour mieux capter ce que vous faites.*

Installation du serveur DNS :

```bash
# installation du serveur DNS, son p'tit nom c'est BIND9
$ sudo dnf install -y bind bind-utils
```

La configuration du serveur DNS va se faire dans 3 fichiers essentiellement :

- **un fichier de configuration principal**
  - `/etc/named.conf`
  - on définit les trucs généraux, comme les adresses IP et le port où on veu écouter
  - on définit aussi un chemin vers les autres fichiers, les fichiers de zone
- **un fichier de zone**
  - `/var/named/tp6.b1.db`
  - je vous préviens, la syntaxe fait mal
  - on peut y définir des correspondances `nom ---> IP`
- **un fichier de zone inverse**
  - `/var/named/tp6.b1.rev`
  - on peut y définir des correspondances `IP ---> nom`

➜ **Allooooons-y, fichier de conf principal**

- je ne vous mets que les lignes les plus importantes, et celles qu'on modifie
- les `[...]` indiquent qu'il faut laisser la conf par défaut, que j'ai enlevé ici pour que ce soit + lisible
  - il ne faut donc pas les mettre dans vos fichiers

```bash
# éditez le fichier de config principal pour qu'il ressemble à :
$ sudo cat /etc/named.conf
options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
[...]
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        recursion yes;
[...]
# référence vers notre fichier de zone
zone "tp6.b1" IN {
     type master;
     file "tp6.b1.db";
     allow-update { none; };
     allow-query {any; };
};
# référence vers notre fichier de zone inverse
zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp6.b1.rev";
     allow-update { none; };
     allow-query { any; };
};
```

➜ **Et pour les fichiers de zone**

- dans ces fichiers c'est le caractère `;` pour les commentaires
- hésitez pas à virer mes commentaires de façon générale
- c'juste pour que vous captiez mais ça fait dégueu dans un fichier de conf

```bash
# Fichier de zone pour nom -> IP

$ sudo cat /var/named/tp6.b1.db

$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns       IN A 10.6.1.101
john      IN A 10.6.1.11
```

```bash
# Fichier de zone inverse pour IP -> nom

$ sudo cat /var/named/tp6.b1.rev

$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Reverse lookup
101 IN PTR dns.tp6.b1.
11 IN PTR john.tp6.b1.
```

➜ **Une fois ces 3 fichiers en place, démarrez le service DNS**

```bash
# Démarrez le service tout de suite
$ sudo systemctl start named

# Faire en sorte que le service démarre tout seul quand la VM s'allume
$ sudo systemctl enable named

# Obtenir des infos sur le service
$ sudo systemctl status named

# Obtenir des logs en cas de probème
$ sudo journalctl -xe -u named
```

🌞 **Dans le rendu, je veux**
```
[lionel@dns etc]$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };
        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

# référence vers notre fichier de zone
zone "tp6.b1" IN {
     type master;
     file "tp6.b1.db";
     allow-update { none; };
     allow-query {any; };
};
# référence vers notre fichier de zone inverse
zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp6.b1.rev";
     allow-update { none; };
     allow-query { any; };
};
```
```
[lionel@dns etc]$ sudo cat /var/named/tp6.b1.db
[sudo] password for lionel:
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns       IN A 10.6.1.101
john      IN A 10.6.1.11
```
```
[lionel@dns etc]$ sudo cat /var/named/tp6.b1.rev
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Reverse lookup
101 IN PTR dns.tp6.b1.
11 IN PTR john.tp6.b1.
```
```
[lionel@dns etc]$ systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: disabled)
     Active: active (running) since Fri 2023-11-17 16:21:15 CET; 12min ago
   Main PID: 2151 (named)
      Tasks: 4 (limit: 4674)
     Memory: 16.9M
        CPU: 68ms
     CGroup: /system.slice/named.service
             └─2151 /usr/sbin/named -u named -c /etc/named.conf

Nov 17 16:21:15 dns named[2151]: configuring command channel from '/etc/rndc.key'
Nov 17 16:21:15 dns named[2151]: command channel listening on 127.0.0.1#953
Nov 17 16:21:15 dns named[2151]: configuring command channel from '/etc/rndc.key'
Nov 17 16:21:15 dns named[2151]: command channel listening on ::1#953
Nov 17 16:21:15 dns named[2151]: managed-keys-zone: loaded serial 0
Nov 17 16:21:15 dns named[2151]: zone 1.4.10.in-addr.arpa/IN: loaded serial 2019061800
Nov 17 16:21:15 dns named[2151]: zone tp6.b1/IN: loaded serial 2019061800
Nov 17 16:21:15 dns named[2151]: all zones loaded
Nov 17 16:21:15 dns systemd[1]: Started Berkeley Internet Name Domain (DNS).
Nov 17 16:21:15 dns named[2151]: running
```
```
[lionel@dns etc]$ ss -nlt
State     Recv-Q    Send-Q       Local Address:Port       Peer Address:Port   Process
LISTEN    0         10              10.6.1.101:53              0.0.0.0:*
```

🌞 **Ouvrez le bon port dans le firewall**
```
[lionel@dns etc]$ ss -nlt
State     Recv-Q    Send-Q       Local Address:Port       Peer Address:Port   Process
LISTEN    0         10              10.6.1.101:53              0.0.0.0:*
```
 ```
 [lionel@dns ~]$ sudo firewall-cmd --add-port=53/udp --permanent
success
[lionel@dns ~]$ sudo firewall-cmd --reload
success
 ```

## 3. Test

🌞 **Sur la machine `john.tp6.b1`**

```
[lionel@jhon ~]$ ping www.ynov.com
PING www.ynov.com (104.26.10.233) 56(84) bytes of data.
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=1 ttl=55 time=22.7 ms
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=2 ttl=55 time=20.6 ms
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=3 ttl=55 time=22.4 ms
```
```
[lionel@jhon ~]$ ping dns.tp6.b1
PING dns.tp6.b1 (10.6.1.101) 56(84) bytes of data.
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=1 ttl=64 time=0.568 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=2 ttl=64 time=1.37 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=3 ttl=64 time=1.26 ms
```
```
[lionel@jhon ~]$ ping john.tp6.b1
PING john.tp6.b1 (10.6.1.11) 56(84) bytes of data.
64 bytes from jhon (10.6.1.11): icmp_seq=1 ttl=64 time=0.146 ms
64 bytes from jhon (10.6.1.11): icmp_seq=2 ttl=64 time=0.098 ms
64 bytes from jhon (10.6.1.11): icmp_seq=3 ttl=64 time=0.064 ms
```

🌞 **Sur votre PC**

```
PS C:\Users\lione> nslookup john.tp6.b1 dns.tp6.b1
Serveur :   UnKnown
Address:  10.6.1.101

Nom :    john.tp6.b1
Address:  10.6.1.11
```

# III. Serveur DHCP

> *On prend les mêmes et on r'commence.*

![Topo 3](./img/topo3.svg)

| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `DHCP`             |
| `dns.tp6.b1`    | `10.6.1.101`       |
| `dhcp.tp6.b1`   | `10.6.1.102`       |

🖥️ **Machine `dhcp.tp6.b1`**

- déroulez la checklisteuh
- indiquez votre propre serveur DNS pour la résolution de noms

🌞 **Installer un serveur DHCP**

```
[lionel@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf

# create new
# specify domain name
option domain-name     "TP6_DHCP";
# specify DNS server's hostname or IP address
option domain-name-servers     10.6.1.101;
# default lease time
default-lease-time 21600;
# max lease time
max-lease-time 30000;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.6.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.6.1.13 10.6.1.37;
    # specify broadcast address
    option broadcast-address 10.6.1.255;
    # specify gateway
    option routers 10.6.1.254;
}
```

🌞 **Test avec `john.tp6.b1`**

```
 enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fb:03:28 brd ff:ff:ff:ff:ff:ff
    inet 10.6.1.13/24 brd 10.6.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 21365sec preferred_lft 21365sec
    inet6 fe80::a00:27ff:fefb:328/64 scope link
       valid_lft forever preferred_lft forever
```
 ```
 [lionel@jhon ~]$ ip r s
default via 10.6.1.254 dev enp0s3
default via 10.6.1.254 dev enp0s3 proto dhcp src 10.6.1.13 metric 100
10.6.1.0/24 dev enp0s3 proto kernel scope link src 10.6.1.13 metric 100
 ```
 ```
 [lionel@jhon ~]$ ping www.google.com
PING www.google.com (172.217.20.196) 56(84) bytes of data.
64 bytes from par10s50-in-f4.1e100.net (172.217.20.196): icmp_seq=1 ttl=115 time=13.1 ms
64 bytes from waw02s08-in-f196.1e100.net (172.217.20.196): icmp_seq=2 ttl=115 time=23.5 m
 ```

🌞 **Requête web avec `john.tp6.b1`**

🦈 **Capture `tp6_web.pcapng`**

# IV. Bonus : Serveur Web HTTPS

Cette partie est un **bonus** : juste pour les téméraires donc.

L'objectif est simple : monter un serveur web comme dans le TP précédent, mais le servir avec HTTPS et pas juste HTTP, afin de fournir une connexion de confiance aux visiteurs du site.

| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `DHCP`             |
| `dns.tp6.b1`    | `10.6.1.101`       |
| `dhcp.tp6.b1`   | `10.6.1.102`       |
| `web.tp6.b1`    | `10.6.1.103`       |

Pour passer de HTTP à HTTPS avec un serveur NGINX ça peut se résumer à :

- taper une commande pour générer une clé et un certificat
- ajouter 3 lignes de conf dans la conf existante

C'est si peu que je vous laisse google "simple nginx https configuration" par exemple ! Hésitez-pas à me demander si besoin d'aide, mais c'est un exercice classique et formateur :)

➜ **On doit donc pouvoir visiter votre site web localement, depuis les VMs, en tapant `https://web.tp6.b1`.**
