

## 1. Echange ARP

🌞**Générer des requêtes ARP**
Jhon : 10.3.1.11
Marcel : 10.3.1.12

```
[lionel@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=1.91 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=1.52 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=1.26 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=64 time=0.998 ms

Table ARP :
[lionel@localhost ~]$ ip neigh show
10.3.1.11 dev enp0s3 lladdr 08:00:27:7e:27:fb STALE
10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:3c DELAY

[lionel@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:3c DELAY
10.3.1.12 dev enp0s3 lladdr 08:00:27:d5:5f:cc STALE

adresse mac de jhon : 08:00:27:7e:27:fb
adresse mac de Marcel : 08:00:27:d5:5f:cc
```

## 2. Analyse de trames

🌞**Analyse de trames**
```
commande : sudo tcpdump

15:27:06.485682 IP localhost.localdomain.ssh > 10.3.1.1.64427: Flags [P.], seq 2997075699:2997075759, ack 2398252934, win 501, length 60
```
🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**
```
[lionel@localhost ~]$ ip route show
10.3.1.0/24 dev enp0s3 proto kernel scope link src 10.3.1.11 metric 100
10.3.2.0/24 via 10.3.1.254 dev enp0s3
```
```
[lionel@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.43 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.61 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.45 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=1.26 ms
64 bytes from 10.3.2.12: icmp_seq=5 ttl=63 time=1.26 ms
```

## 2. Analyse de trames

🌞**Analyse des échanges ARP**
Marcel vers Jhon :

| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
| ----- | ----------- | --------- | ------------------------- | -------------- | -------------------------- |
| 1     | Requête ARP | x         | 08:00:27:e1:f8:9a         | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         | 08:00:27:7e:27:fb         | x              | 08:00:27:e1:f8:9a          |
| ...   | ...         | ...       | ...                       |                |                            |
| ?     | Ping        | 10.3.2.12 | 08:00:27:e1:f8:9a         | 10.3.2.11      | 08:00:27:7e:27:fb          |
| ?     | Pong        | 10.3.2.11 | 08:00:27:7e:27:fb         | 10.3.2.12      | 08:00:27:e1:f8:9a          |

## 3. Accès internet

🌞**Donnez un accès internet à vos machines** - config routeur

```
Depuis le routeur :
[lionel@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=17.5ms 
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=20.7ms 
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=18.8ms 
```

🌞**Donnez un accès internet à vos machines** - config clients
```
Depuis Jhon :
[lionel@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=18.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=21.2 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=19.1 ms
```
```
Depuis Jhon :
[lionel@localhost ~]$ ping google.com
PING google.com (142.250.179.110) 56(84) bytes of data.
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=1 ttl=115 time=22.4 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=2 ttl=115 time=18.4 ms
^C64 bytes from 142.250.179.110: icmp_seq=3 ttl=115 time=19.5 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 10097ms
rtt min/avg/max/mdev = 18.434/20.115/22.422/1.687 ms
```
🌞**Analyse de trames**
Marcel vers Jhon :
| ordre | type trame | IP source            | MAC source                | IP destination | MAC destination |     |
| ----- | ---------- | -------------------- | ------------------------- | -------------- | --------------- | --- |
| 1     | ping       | 10.3.1.254           | 08:00:27:e1:f8:9a         | 10.3.1.11      |08:00:27:7e:27:fb|     |
| 2     | pong       | 10.3.1.11            |08:00:27:7e:27:fb          | 10.3.1.254     |08:00:27:e1:f8:9a| ... |