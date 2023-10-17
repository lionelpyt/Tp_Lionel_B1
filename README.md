🌞 Mettez en place une configuration réseau fonctionnelle entre les deux machines
```
PS C:\Users\lione> netsh interface ipv4 set address name="Ethernet" static 10.10.10.90 255.255.255.252
Mon ip : 10.10.10.186
ip machine 2 : 10.10.10.185
mask : 255.255.255.252


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