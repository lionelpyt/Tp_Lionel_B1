# Tp_Lionel_B1

I.Exploration en solo

1. Affichage d'informations sur la pile TCP/IP locale

🌞 Affichez les infos des cartes réseau de votre PC
```
commande : ipconfig /all
WIFI :
Nom : MediaTek Wi-Fi 6 MT7921 Wireless LAN Card
Adresse MAC : CC-5E-F8-6E-4B-33
Adresse IP : 10.33.48.39
ETHERNET:
Nom : Realtek PCIe GbE Family Controller
Adresse MAC : 08-8F-C3-FE-EA-EE
Adresse IP : inexistante
```

🌞 Affichez votre gateway
```
commande : ipconfig
Adresse IP passerelle : 10.33.51.254
```
🌞 Déterminer la MAC de la passerelle
```
commande : arp -a
Adresse MAC : 00-50-56-e3-9f-8c
```
🌞 Trouvez comment afficher les informations sur une carte
```
Les étapes:

1) Parametre WIFI
2) Propriété du matériel
```
2. Modifications des informations

A. Modification d'adresse IP (part 1)

🌞 Utilisez l'interface graphique de votre OS pour changer d'adresse IP :
```
Carte réseau sans fil Wi-Fi :

   Suffixe DNS propre à la connexion. . . :
   Adresse IPv6 de liaison locale. . . . .: fe80::9afb:ea9a:fa9c:f7e6%16
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.48.10
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Passerelle par défaut. . . . . . . . . : 10.33.51.254
```
  🌞 Il est possible que vous perdiez l'accès internet.
```
  Je pense que notre adresse IP n'est pas reconnu par le wifi d'Ynov donc nous navons pas les droits d'y accéder.
  ```
  II. Exploration locale en duo

  1. Prérequis

  2. Câblage

  3. Modification d'adresse IP

  🌞 Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau

  🌞 Vérifier à l'aide d'une commande que votre IP a bien été changée
  ```
Carte Ethernet Ethernet :

   Suffixe DNS propre à la connexion. . . :
   Adresse IPv6 de liaison locale. . . . .: fe80::5613:62aa:2eda:9906%2
   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.90
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Passerelle par défaut. . . . . . . . . :
```
   🌞 Vérifier que les deux machines se joignent
```
Envoi d’une requête 'Ping'  10.10.10.213 avec 32 octets de données :

Réponse de 10.10.10.213 : octets=32 temps=4 ms TTL=128
Réponse de 10.10.10.213 : octets=32 temps=4 ms TTL=128
Réponse de 10.10.10.213 : octets=32 temps=4 ms TTL=128
Réponse de 10.10.10.213 : octets=32 temps=5 ms TTL=128
```
🌞 Déterminer l'adresse MAC de votre correspondant
```
 40-c2-ba-10-d5-74
```
 4. Petit chat privé

 🌞 Visualiser la connexion en cours
```
 [nc.exe]
 
  TCP    10.33.48.39:139        0.0.0.0:0              LISTENING
```
  🌞 Pour aller un peu plus loin
  ```
  PS C:\Users\fayer> netstat -a -n -b
[nc64.exe]
  TCP    0.0.0.0:27036          0.0.0.0:0              LISTENING

PS C:\Users\fayer> netstat -a -n -b
 [nc64.exe]
  TCP    10.33.48.141:139       0.0.0.0:0              LISTENING
```
🌞 Activez et configurez votre firewall
```
PS C:\Users\lione\Downloads\netcat-win32-1.11\netcat-1.11> ping 192.168.137.1

Envoi d’une requête 'Ping'  192.168.137.1 avec 32 octets de données :
Réponse de 192.168.137.1 : octets=32 temps=8 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
```
🌞Tester l'accès internet
```
PS C:\Users\lione\Downloads\netcat-win32-1.11\netcat-1.11> ping 1.1.1.1

Envoi d’une requête 'Ping'  1.1.1.1 avec 32 octets de données :
Réponse de 192.168.137.2 : Impossible de joindre l’hôte de destination.
Délai d’attente de la demande dépassé.
Réponse de 192.168.137.2 : Impossible de joindre l’hôte de destination.
Réponse de 192.168.137.2 : Impossible de joindre l’hôte de destination.

Statistiques Ping pour 1.1.1.1:
    Paquets : envoyés = 4, reçus = 3, perdus = 1 (perte 25%)
```
🌞 Prouver que la connexion Internet passe bien par l'autre PC

III. Manipulations d'autres outils/protocoles côté client

1. DHCP

🌞Exploration du DHCP, depuis votre PC
```
ipconfig /all
l'adresse IP du serveur DHCP du réseau WiFi YNOV : 10.33.51.254
```

🌞 Trouver l'adresse IP du serveur DNS que connaît votre ordinateurre
```
nslookup
Serveurs DNS : 10.33.10.2
```
```
Pour google :

PS C:\Users\lione> nslookup google.com 8.8.8.8
Serveur :   dns.google
Address:  8.8.8.8

Réponse ne faisant pas autorité :
Nom :    google.com
Addresses:  2a00:1450:4007:80c::200e
          172.217.20.174

pour ynov.com :

  reverse lookup : 
  
  nslookup 231.31.113.12 8.8.8.8 :
  ne trouve pas
  nslookup 78.34.2.17 8.8.8.8
  78.34.2.17
```
🌞 Utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :

