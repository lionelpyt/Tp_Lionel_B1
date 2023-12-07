# II. SSH

## Sommaire
## 0. Setup

| Name            | LAN1 `10.7.1.0/24` |
| --------------- | ------------------ |
| `router.tp7.b1` | `10.7.1.254`       |
| `john.tp7.b1`   | `10.7.1.11`        |

## 1. Fingerprint

### A. Explications

💡 **Cette section explique comment vous pouvez vous assurer que vous vous connectez au bon serveur SSH.**

➜ **Lorsqu'on se connecte à une machine en SSH pour la première fois, apparaît un message comme celui-ci :**

➜ **Comment trust ?**

➜ **Le message n'apparaît plus aux connexions suivantes**

### B. Manips

🌞 **Effectuez une connexion SSH en vérifiant le fingerprint**

```
PS C:\Users\lione> ssh lionel@10.7.1.11
The authenticity of host '10.7.1.11 (10.7.1.11)' can't be established.
ED25519 key fingerprint is SHA256:LmKer88FXyBxoPfXWqSuD+6BD7j6fDNFQYR7JTiK8Ps.
This host key is known by the following other names/addresses:
    C:\Users\lione/.ssh/known_hosts:7: 10.3.1.12
    C:\Users\lione/.ssh/known_hosts:10: 10.3.1.11
    C:\Users\lione/.ssh/known_hosts:11: 10.3.1.254
    C:\Users\lione/.ssh/known_hosts:12: 10.4.1.12
    C:\Users\lione/.ssh/known_hosts:13: 10.4.1.253
    C:\Users\lione/.ssh/known_hosts:14: 10.4.1.254
    C:\Users\lione/.ssh/known_hosts:15: 10.4.1.200
    C:\Users\lione/.ssh/known_hosts:16: 10.4.1.137
    (12 additional names omitted)
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
```
[lionel@john ~]$ sudo ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key
[sudo] password for lionel:
256 SHA256:LmKer88FXyBxoPfXWqSuD+6BD7j6fDNFQYR7JTiK8Ps /etc/ssh/ssh_host_ed25519_key.pub (ED25519)
```

## 2. Conf serveur SSH

On va faire un truc très basique, pour manipuler un peu toujours les cartes réseau, les IP, les ports, toussa : on va choisir explicitement l'IP et le port où tourne notre serveur SSH sur `router`.

La configuration du serveur SSH se fait dans le fichier `/etc/ssh/sshd_config`, utilisez `nano` pour le modifier par exemple.

Il faut être admin pour modifier le fichier, il faudra donc préfixer votre commande avec `sudo`.

🌞 **Consulter l'état actuel**
```
[lionel@router ~]$ sudo ss -nltp
State  Recv-Q Send-Q Local Address:Port Peer Address:PortProcess
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=1473,fd=3))
```
🌞 **Modifier la configuration du serveur SSH**
```
[lionel@router ~]$ sudo cat /etc/ssh/sshd_config | grep 25000
Port 25000
[lionel@router ~]$ sudo cat /etc/ssh/sshd_config | grep 10.7
ListenAddress 10.7.1.254
[lionel@router ~]$ sudo systemctl restart sshd
```
🌞 **Prouvez que le changement a pris effet**

```
[lionel@router ~]$ sudo ss -nltp
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port   Process
LISTEN    0         128             10.7.1.254:25000            0.0.0.0:*       users:(("sshd",pid=1499,fd=3))
```

🌞 **N'oubliez pas d'ouvrir ce nouveau port dans le firewall**

```
[lionel@router ~]$ sudo firewall-cmd --add-port=22/tcp --permanent
success
[lionel@router ~]$ sudo firewall-cmd --reload
success
```

🌞 **Effectuer une connexion SSH sur le nouveau port**

```
PS C:\Users\lione> ssh lionel@router.tp7.b1 -p 25000
The authenticity of host '[router.tp7.b1]:25000 ([10.7.1.254]:25000)' can't be established.
ED25519 key fingerprint is SHA256:LmKer88FXyBxoPfXWqSuD+6BD7j6fDNFQYR7JTiK8Ps.
This host key is known by the following other names/addresses:
    C:\Users\lione/.ssh/known_hosts:7: 10.3.1.12
    C:\Users\lione/.ssh/known_hosts:10: 10.3.1.11
    C:\Users\lione/.ssh/known_hosts:11: 10.3.1.254
    C:\Users\lione/.ssh/known_hosts:12: 10.4.1.12
    C:\Users\lione/.ssh/known_hosts:13: 10.4.1.253
    C:\Users\lione/.ssh/known_hosts:14: 10.4.1.254
    C:\Users\lione/.ssh/known_hosts:15: 10.4.1.200
    C:\Users\lione/.ssh/known_hosts:16: 10.4.1.137
    (13 additional names omitted)
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

## 3. Connexion par clé

### A. Explications

💡 **Cette section explique comment vous pouvez remplacer l'utilisation d'un password (faible en terme de sécurité) par une clé (fort en terme de sécurité).**

### B. Manips

🌞 **Générer une paire de clés**
```
PS C:\Users\lione> ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
The key fingerprint is:
SHA256:4rrRcC4tnBo45rM8TqAo+QOXkolghmAQ1WYMV305gnw lione@Lionelp
```

🌞 **Déposer la clé publique sur une VM**
```
ssh-copy-id lionel@john
```
🌞 **Connectez-vous en SSH à la machine**

```
PS C:\Users\lione> ssh lionel@john
Last login: Fri Nov 24 15:30:50 2023 from 10.7.1.1
[lionel@john ~]$
```

### C. Changement de fingerprint

Si un jour, quand tu te connectes à un serveur auquel tu t'es déjà connecté avant (genre sur la même adresse IP) mais que le fingerprint a changé, alors la connexion va échouer.

En effet comme on a dit, grâce au fichier `known_hosts` votre PC enregistre les fingerprint que vous acceptez et compare automatiquement à chaque fois.

Vous mangez donc un beau message d'avertissement et la connexion échoue si ça arrive.

On va provoquer cette situation, pour que vous expérimentiez le truc. Ce qu'on va faire dans cette partie :

- supprimer les clés sur le serveur SSH
- regénérez des nouvelles clés (les empreintes seront donc différentes)
- faire une connexion SSH et manger un beau message d'avertissement avec une connexion échouée
- remédier à la situation

🌞 **Supprimer les clés sur la machine `router.tp7.b1`**
```
[lionel@router ~]$ sudo rm /etc/ssh/ssh_host_*
```

🌞 **Regénérez les clés sur la machine `router.tp7.b1`**

```
[lionel@router ~]$ sudo ssh-keygen -A
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519
[lionel@router ~]$ sudo systemctl restart sshd
```

🌞 **Tentez une nouvelle connexion au serveur**
```
changer avec la bonne KEY :
SHA256:++MHErHK7KgndNL3YJohoYD4ZGSUvRmFldV+aK5KLvA.
```
🌞 **Montrer sur quel port est disponible le serveur web**
```
[lionel@web ~]$ ss -nltp
State     Recv-Q    Send-Q       Local Address:Port       Peer Address:Port   Process
LISTEN    0         511                0.0.0.0:80              0.0.0.0:*
```
🌞 **Générer une clé et un certificat sur web.tp7.b1**
```
29  sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt
   30  sudo mv server.key /etc/pki/tls/private/web.tp7.b1.key
   31  sudo mv server.crt /etc/pki/tls/certs/web.tp7.b1.crt
   32  sudo chown nginx:nginx /etc/pki/tls/private/web.tp7.b1.key
   33  sudo chown nginx:nginx /etc/pki/tls/certs/web.tp7.b1.crt
   34  sudo chmod 0400 /etc/pki/tls/private/web.tp7.b1.key
   35  sudo chmod 0444 /etc/pki/tls/certs/web.tp7.b1.crt
```
🌞 **Modification de la conf de NGINX**
```
[lionel@web ~]$ sudo cat /etc/nginx/conf.d/site_web_nul.conf
server {
  # le port sur lequel on veut écouter
  listen 10.7.1.12:443 ssl;
  ssl_certificate /etc/pki/tls/certs/web.tp7.b1.crt;
  ssl_certificate_key /etc/pki/tls/private/web.tp7.b1.key;


  # le nom de la page d'accueil si le client de la précise pas
  index index.html;

  # un nom pour notre serveur (pas vraiment utile ici, mais bonne pratique)
  server_name www.site_web_nul.b1;

  # le dossier qui contient notre site web
  root /var/www/site_web_nul;
}
```
🌞 **Conf firewall**
```
38  sudo firewall-cmd --add-port=443/tcp --permanent
39  sudo firewall-cmd --reload
```
🌞 **Redémarrez NGINX**
```
40  sudo systemctl restart nginx
```
🌞 **Prouvez que NGINX écoute sur le port 443/tcp**
```
[lionel@web ~]$ sudo ss -ntlp
State      Recv-Q     Send-Q          Local Address:Port           Peer Address:Port     Process
LISTEN     0          511                 10.7.1.12:443                 0.0.0.0:*         users:(("nginx",pid=1518,fd=6),("nginx",pid=1517,fd=6))
```
🌞 **Visitez le site web en https**
```
[lionel@john ~]$ curl -k https://10.7.1.12
<h1>Coucou</h1>
```
🌞 **Installer le serveur Wireguard sur vpn.tp7.b1**
```
[lionel@vpn ~]$ sudo cat /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```
```
   21  sudo modprobe wireguard
   22  echo wireguard | sudo tee /etc/modules-load.d/wireguard.conf
   23  sudo nano /etc/sysctl.conf
   24  sudo sysctl -p
   26  sudo ip route add default via 10.7.1.254 dev enp0s3
   27  sudo dnf install wireguard-tools -y
   28  sudo cat /etc/sysctl.conf
```
🌞 **Générer la paire de clé du serveur sur vpn.tp7.b1**
```
[lionel@vpn ~]$ sudo cat /etc/wireguard/clients/john.key | wg pubkey | sudo tee /etc/wire
guard/clients/john.pub
+Ck26etZn7HYp9lHzy5mxqCRLjVuxJdyrEWfioHrDBw=
```
🌞 **Création du fichier de config du serveur sur vpn.tp7.b1**
```
[lionel@vpn ~]$ sudo cat /etc/wireguard/wg0.conf
[Interface]
Address = 10.107.1.0/24
SaveConfig = false
PostUp = firewall-cmd --zone=public --add-masquerade
PostUp = firewall-cmd --add-interface=wg0 --zone=public
PostDown = firewall-cmd --zone=public --remove-masquerade
PostDown = firewall-cmd --remove-interface=wg0 --zone=public
ListenPort = 13337
PrivateKey = yDtnbGkR6YmmoNiVPGYcWxnjmXD+gMpGO0ZrbIenp0U=

[Peer]
PublicKey = +Ck26etZn7HYp9lHzy5mxqCRLjVuxJdyrEWfioHrDBw=
AllowedIPs = 10.107.1.11/32
```
🌞 **Démarrez le serveur VPN sur vpn.tp7.b1**
```
[lionel@vpn ~]$ sudo wg show
interface: wg0
  public key: PdmK0flkvupsm6I5H/67gKGWfgaCX3nSEDHL3pKKSHA=
  private key: (hidden)
  listening port: 13337

peer: +Ck26etZn7HYp9lHzy5mxqCRLjVuxJdyrEWfioHrDBw=
  allowed ips: 10.107.1.11/32
```
🌞 **Ouvrir le bon port firewall**
```
[lionel@vpn ~]$ ss | grep 10.7.1.101
tcp   ESTAB 0      52                       10.7.1.101:ssh       10.7.1.1:61797
```
```
[lionel@vpn ~]$ sudo firewall-cmd --add-port=13337/tcp --permanent
success
[lionel@vpn ~]$ sudo firewall-cmd --reload
success
```
🌞 **On installe les outils Wireguard sur john.tp7.b1**
```
[lionel@john ~]$ sudo dnf install wireguard-tools
Complete!
```
🌞 **Supprimez votre route par défaut**
```
[lionel@john ~]$ ping 8.8.8.8
ping: connect: Network is unreachable
[lionel@john ~]$ ping www.google.com
ping: www.google.com: Name or service not known
```
🌞 **Configurer un fichier de conf sur john.tp7.b1**
```
[lionel@john ~]$ sudo cat wireguard/john.conf
[Interface]
Address = 10.107.1.11/24
PrivateKey = yDtnbGkR6YmmoNiVPGYcWxnjmXD+gMpGO0ZrbIenp0U=

[Peer]
PublicKey = +Ck26etZn7HYp9lHzy5mxqCRLjVuxJdyrEWfioHrDBw=
AllowedIPs = 0.0.0.0/0
Endpoint = 10.7.1.101:13337
```
🌞 **Lancer la connexion VPN**
```
[lionel@john wireguard]$ wg-quick up ./john.conf
Warning: `/home/lionel/wireguard/john.conf' is world accessible
[#] ip link add john type wireguard
[#] wg setconf john /dev/fd/63
[#] ip -4 address add 10.107.1.11/24 dev john
[#] ip link set mtu 1420 up dev john
[#] wg set john fwmark 51820
[#] ip -4 route add 0.0.0.0/0 dev john table 51820
[#] ip -4 rule add not fwmark 51820 table 51820
[#] ip -4 rule add table main suppress_prefixlength 0
[#] sysctl -q net.ipv4.conf.all.src_valid_mark=1
[#] nft -f /dev/fd/63
```
🌞 **Vérifier la connexion**
```

```
