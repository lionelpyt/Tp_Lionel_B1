# Tp_Lionel_B1

I.Exploration en solo

1. Affichage d'informations sur la pile TCP/IP locale

🌞 Affichez les infos des cartes réseau de votre PC
commande : ipconfig /all
WIFI :
Nom : MediaTek Wi-Fi 6 MT7921 Wireless LAN Card
Adresse MAC : CC-5E-F8-6E-4B-33
Adresse IP : 10.33.48.39
ETHERNET:
Nom : Realtek PCIe GbE Family Controller
Adresse MAC : 08-8F-C3-FE-EA-EE
Adresse IP : inexistante

🌞 Affichez votre gateway
commande : ipconfig
Adresse IP passerelle : 10.33.51.254

🌞 Déterminer la MAC de la passerelle
commande : arp -a
Adresse MAC : 00-50-56-e3-9f-8c

🌞 Trouvez comment afficher les informations sur une carte IP
Les étapes:
1) Parametre WIFI
2) Propriété du matériel

2. Modifications des informations

A. Modification d'adresse IP (part 1)

🌞 Utilisez l'interface graphique de votre OS pour changer d'adresse IP :

Carte réseau sans fil Wi-Fi :

   Suffixe DNS propre à la connexion. . . :
   Adresse IPv6 de liaison locale. . . . .: fe80::9afb:ea9a:fa9c:f7e6%16
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.48.10
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Passerelle par défaut. . . . . . . . . : 10.33.51.254

  🌞 Il est possible que vous perdiez l'accès internet.

  Je pense que notre adresse IP n'est pas reconnu par le wifi d'Ynov donc nous navons pas les droits d'y accéder.

  II. Exploration locale en duo

  1. Prérequis

  2. Câblage

  3. Modification d'adresse IP

  🌞 Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau

  🌞 Vérifier à l'aide d'une commande que votre IP a bien été changée

Carte Ethernet Ethernet :

   Suffixe DNS propre à la connexion. . . :

   Adresse IPv6 de liaison locale. . . . .: fe80::5613:62aa:2eda:9906%2

   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.90

   Masque de sous-réseau. . . . . . . . . : 255.255.255.0

   Passerelle par défaut. . . . . . . . . :
   