🌞 Mettez en place une configuration réseau fonctionnelle entre les deux machines
```
PS C:\Users\lione> netsh interface ipv4 set address name="Ethernet" static 10.10.10.90 255.255.255.252
Mon ip : 10.10.10.186
ip machine 2 : 10.10.10.185
mask : 255.255.255.252
adresse réseaux : 00001010.00001010.00001010.10111000 (10.10.10.184)
adresse broadcast :00001010.00001010.00001010.10111011 (10.10.10.187)

```
🌞 Prouvez que la connexion est fonctionnelle entre les deux machines
```
PS C:\Users\lione> ping 10.10.10.185

Envoi d’une requête 'Ping'  10.10.10.185 avec 32 octets de données :
Réponse de 10.10.10.185 : octets=32 temps=3 ms TTL=128
Réponse de 10.10.10.185 : octets=32 temps=3 ms TTL=128
Réponse de 10.10.10.185 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.185 : octets=32 temps=1 ms TTL=128

Statistiques Ping pour 10.10.10.185:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 3ms, Moyenne = 2ms
```
🌞 Wireshark it
```
Fichier PingPong
```
🌞 Check the ARP table
```
PS C:\Users\lione> arp -a

mac de mon binome: 40-c2-ba-10-d5-74
mac du gateway reseaux : b2-aa-a0-76-3c-96
```
🌞 Manipuler la table ARP
```
PS C:\Users\lione> arp -d
PS C:\Users\lione> arp -a

Interface : 169.254.149.198 --- 0x6
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.206.1 --- 0x10
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.71.6 --- 0x11
  Adresse Internet      Adresse physique      Type
  10.33.79.254          7c-5a-1c-d3-d8-76     dynamique
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.153.1 --- 0x13
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
```
🌞 Wireshark it
```

```
