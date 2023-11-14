# TP5 : TCP, UDP et services réseau

Dans ce TP on va explorer un peu les protocoles TCP et UDP. 

**La première partie est détente**, vous explorez TCP et UDP un peu, en vous servant de votre PC.

La seconde partie se déroule en environnement virtuel, avec des VMs. Les VMs vont nous permettre en place des services réseau, qui reposent sur TCP et UDP.  
**Le but est donc de commencer à mettre les mains de plus en plus du côté administration, et pas simple client.**

Dans cette seconde partie, vous étudierez donc :

- le protocole SSH (contrôle de machine à distance)
- le protocole DNS (résolution de noms)
  - essentiel au fonctionnement des réseaux modernes

![TCP UDP](./img/tcp_udp.jpg)

# Sommaire

- [TP5 : TCP, UDP et services réseau](#tp5--tcp-udp-et-services-réseau)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
- [I. First steps](#i-first-steps)
- [II. Setup Virtuel](#ii-setup-virtuel)
  - [1. SSH](#1-ssh)
  - [2. Routage](#2-routage)
  - [3. Serveur Web](#3-serveur-web)

# 0. Prérequis

➜ **Pour ce TP, on va se servir de VMs Rocky Linux** quand des VMs seront nécessaires. On va en faire plusieurs, n'hésitez pas à diminuer la RAM (512Mo ou 1Go devraient suffire). Vous pouvez redescendre la mémoire vidéo aussi.  

➜ **Si vous voyez un 🦈** c'est qu'il y a une capture PCAP à produire et à mettre dans votre dépôt git de rendu

➜ **L'emoji 🖥️ indique une VM à créer**. Pour chaque VM, vous déroulerez la checklist suivante :

- [x] Créer la machine (avec une carte host-only)
- [ ] Définir une IP statique à la VM
- [ ] Donner un hostname à la machine
- [ ] Utiliser SSH pour administrer la machine
- [ ] Dès que le routeur est en place, n'oubliez pas d'ajouter une route par défaut aux autres VM pour qu'elles aient internet

> Toutes les commandes pour réaliser ces opérations sont dans [le mémo Rocky](../../cours/memo/rocky_network.md). Aucune de ces étapes ne doit figurer dan le rendu, c'est juste la mise en place de votre environnement de travail.

# I. First steps

Faites-vous un petit top 5 des applications que vous utilisez sur votre PC souvent, des applications qui utilisent le réseau : un site que vous visitez souvent, un jeu en ligne, Spotify, j'sais po moi, n'importe.

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

Mon ip 10:33.71.6


client League of legend : 104.17.166.5 Port: 443 TCP
mon port : 49680 

youtube :  77.136.192.84 port: 443 UDP
mon port : 64362

Drakensang Online : 8.209.84.94 port: 31610 UDP
mon port : 65203

Epic Game : 2.20.93.74 port: 443 TCP
mon port : 64298

Discord : 52.2.103.253 port: 443 TCP
mon port : 49932


> Rappel : quand on utilise un service en ligne, on se connecte sur un port d'un serveur. Pour ce faire, le client ouvre un port random, et se connecte à un port spécifique qu'il connaît sur le serveur. Par exemple 443 en TCP pour du trafic HTTPS. Le client doit savoir à l'avance à quelle IP et à quel port il se connecte.

🌞 **Demandez l'avis à votre OS**

- votre OS est responsable de l'ouverture des ports, et de placer un programme qui se "lie" à un port (on dit généralement qu'il se *bind* à un port)
- l'OS garde une trace en permanence de quelle programme utilise quel port
- utilisez la commande adaptée à votre OS pour repérer, dans la liste de toutes les connexions réseau établies, la connexion que vous voyez dans Wireshark, pour chacune des 5 applications
- **lancez votre terminal en admin pour avoir toutes les infos**

**Il faudra ajouter des options adaptées aux commandes pour y voir clair. Pour rappel, vous cherchez des connexions TCP ou UDP.**

```
# MacOS
$ netstat

# GNU/Linux
$ ss

# Windows
$ netstat
```

🦈🦈🦈🦈🦈 **Bah ouais, 5 captures Wireshark à l'appui évidemment.** Ce sera `tp5_service_1.pcapng` jusqu'à `tp5_service_5.pcapng`.

> Une capture pour chaque application, qui met bien en évidence le trafic en question. C'est à dire on ne doit voir QUE le trafic de l'application en question. Vous devez donc utiliser un filtre dans Wireshark pour isoler le trafic que vous souhaitez.

# II. Setup Virtuel

## 1. SSH

| Machine        | Réseau `10.5.1.0/24` |
| -------------- | -------------------- |
| `node1.tp5.b1` | `10.5.1.11`          |

🖥️ **Machine `node1.tp5.b1`**

- n'oubliez pas de dérouler la checklist (voir [les prérequis du TP](#0-prérequis))
- donnez lui l'adresse IP `10.5.1.11/24`

Connectez-vous en SSH à votre VM.

🌞 **Examinez le trafic dans Wireshark**

🦈 **`tp5_3_way.pcapng`

> **SUR WINDOWS, pour cette étape uniquement**, utilisez Git Bash et PAS Powershell. Avec Powershell il sera très difficile d'observer le FIN ACK.

🌞 **Demandez aux OS**

```
sur ma machine :
commande : netstat -i -n
TCP    10.5.1.1:54393         10.5.1.11:22           ESTABLISHED        414067
```
```
sur vm :
commande : ss -a
tcp                ESTAB               0                   0                                              10.5.1.11:ssh                                                10.5.1.1:54393
```

## 2. Routage

🌞 **Prouvez que**
```
[lionel@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=19.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=19.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=18.8 ms
```
```
[lionel@node1 ~]$ ping www.ynov.com
PING www.ynov.com (172.67.74.226) 56(84) bytes of data.
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=1 ttl=55 time=20.7 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=2 ttl=55 time=21.1 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=3 ttl=55 time=20.8 ms
```

## 3. Serveur Web

Dans cette section on va monter un p'tit serveur Web sur une VM. Le serveur web hébergera un unique site web, qui sera super nul super moche parce qu'on est pas en cours de dév web :D

Le nom du serveur web qu'on va utiliser c'est NGINX. On l'installe, on le configure, on le lance, et PAF un site web hébergé en local disponible.

| Machine         | Réseau `10.5.1.0/24` |
| --------------- | -------------------- |
| `node1.tp5.b1`  | `10.5.1.11`          |
| `router.tp5.b1` | `10.5.1.254`         |
| `web.tp5.b1`    | `10.5.1.12`          |

🖥️ **Machine `web.tp5.b1`**

🌞 **Installez le paquet `nginx`**

```
[lionel@web ~]$ sudo dnf install nginx
```

🌞 **Créer le site web**

🌞 **Donner les bonnes permissions**

```bash
[lionel@web ~]$sudo chown -R nginx:nginx /var/www/site_web_nul
```

🌞 **Créer un fichier de configuration NGINX pour notre site web**

🌞 **Démarrer le serveur web !**

- avec une commande `sudo systemctl start nginx`
- si rien ne s'affiche, c'est que tout va bien
  - `systemctl status nginx` pour vérifier
- si quelque chose s'affiche c'est qu'il y a un soucis
  - lisez et interprétez le message d'erreur
  - si ça suffit pas/si vous galérez à comprendre, vous m'appelez

🌞 **Ouvrir le port firewall**
```
[lionel@web conf.d]$ sudo firewall-cmd --permanent --zone=public --add-service=http
```
```
[lionel@web conf.d]$ sudo firewall-cmd --reload
```

🌞 **Visitez le serveur web !**
```
[lionel@node1 ~]$ sudo curl http://10.5.1.12
[sudo] password for lionel:
<h1>Coucou</h1>
```
🌞 **Visualiser le port en écoute**
```
[lionel@web conf.d]$ ss -nlt
State         Recv-Q        Send-Q               Local Address:Port               Peer Address:Port       Process
LISTEN        0             511                        0.0.0.0:80                      0.0.0.0:*
LISTEN        0             511                           [::]:80                         [::]:*
```

🌞 **Analyse trafic**

🦈 **`tp5_web.pcapng`**
