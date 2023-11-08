# TP4 : DHCP

# I. DHCP Client

🌞 **Déterminer**

- l'adresse du serveur DHCP : 10.33.79.254
- bail DHCP obtenu : 13:57:48
- expiration bail DHCP : 13:57:40

🌞 **Capturer un échange DHCP**
``` 
commande : ipconfig /release
```
DORA
🌞 **Analyser la capture Wireshark**

PaquetClient

## II. Serveur DHCP


### 1. Topologie

### 2. Tableau d'adressage

### 3. Setup topologie

➜ **Mettez en place la topologie**

🌞 **Preuve de mise en place**
```
[lionel@dhcp ~]$ ping google.com
PING google.com (142.250.179.110) 56(84) bytes of data.
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=1 ttl=115 time=19.7 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=2 ttl=115 time=21.7 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=3 ttl=115 time=21.1 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=4 ttl=115 time=17.9 ms
```
```
[lionel@node2 ~]$ ping google.com
PING google.com (142.250.179.110) 56(84) bytes of data.
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=1 ttl=115 time=21.5 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=2 ttl=115 time=20.7 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=3 ttl=115 time=18.5 ms
```

```
[lionel@node2 ~]$ traceroute 1
traceroute to 1 (0.0.0.1), 30 hops max, 60 byte packets
 1  _gateway (10.4.1.254)  11.326 ms  9.845 ms  9.322 ms
 2  10.0.3.2 (10.0.3.2)  8.902 ms  8.343 ms  8.005 ms
 3  10.0.3.2 (10.0.3.2)  7.144 ms !N  6.808 ms !N  3.438 ms !N
 ```

### 4. Serveur DHCP

🌞 **Rendu**

```
dnf -y install dhcp-server
sudo nano /etc/dhcp/dhcpd.conf
systemctl enable --now dhcpd
```

`systemctl status dhcpd` 
```
[lionel@dhcp ~]$ systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Wed 2023-11-08 20:56:40 CET; 1min 14s ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 11505 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 4674)
     Memory: 4.5M
        CPU: 8ms
     CGroup: /system.slice/dhcpd.service
             └─11505 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid
```
```
[lionel@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf

# create new
# specify domain name
option domain-name     "srv.world";
# specify DNS server's hostname or IP address
option domain-name-servers     dlp.srv.world;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # specify broadcast address
    option broadcast-address 10.4.1.255;
    # specify gateway
    option routers 10.4.1.254;
}
```

### 5. Client DHCP

➜ **Petite astuce**

- pour avoir un peu plus de détails sur l'interaction entre le client et votre serveur DHCP, vous pouvez lancer la commande `sudo journalctl -xe -u dhcpd -f` sur `dhcp.tp4.b1`
- cette commande permet de suivre en temps réel l'arrivée de nouveaux logs
- si vous laissez tourner cette commande pendant l'étape qui suit, vous allez voir arriver en temps réel les requêtes DHCP du client dnas les logs

🌞 **Test !**

```
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

🌞 **Prouvez que**
```
[lionel@node1 ~]$ ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:31:7a:f8 brd ff:ff:ff:ff:ff:ff
    inet 10.4.1.200/24 brd 10.4.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 527sec preferred_lft 527sec
    inet6 fe80::a00:27ff:fe31:7af8/64 scope link
       valid_lft forever preferred_lft forever
```
```
[lionel@node1 ~]$ journalctl | grep -Ei 'dhcp'
Nov 08 21:22:10 node1 NetworkManager[1348]: <info>  [1699474930.1452] dhcp4 (enp0s3): state changed new lease, address=10.4.1.200
```
```
[lionel@node1 ~]$ ping 10.4.1.254
PING 10.4.1.254 (10.4.1.254) 56(84) bytes of data.
64 bytes from 10.4.1.254: icmp_seq=1 ttl=64 time=0.969 ms
64 bytes from 10.4.1.254: icmp_seq=2 ttl=64 time=0.613 ms
64 bytes from 10.4.1.254: icmp_seq=3 ttl=64 time=0.955 ms
64 bytes from 10.4.1.254: icmp_seq=4 ttl=64 time=0.758 ms
```

🌞 **Bail DHCP serveur**

```
[lionel@dhcp ~]$  cat /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.2b1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

lease 10.4.1.200 {
  starts 3 2023/11/08 20:57:10;
  ends 3 2023/11/08 21:07:10;
  cltt 3 2023/11/08 20:57:10;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:31:7a:f8;
  uid "\001\010\000'1z\370";
}
```

### 6. Options DHCP

🌞 **Nouvelle conf !**

```
[lionel@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
[sudo] password for lionel:

# create new
# specify domain name
option domain-name     "TP4_DHCP";
# specify DNS server's hostname or IP address
option domain-name-servers     8.8.8.8;
# default lease time
default-lease-time 21600;
# max lease time
max-lease-time 30000;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # specify broadcast address
    option broadcast-address 10.4.1.255;
    # specify gateway
    option routers 10.4.1.254;
}
```

🌞 **Test !**
```
dhclient -r -v enp0s3
dhclient -v enp0s3
```
```
[lionel@node1 ~]$ sudo cat /etc/resolv.conf
[sudo] password for lionel:
; generated by /usr/sbin/dhclient-script
search TP4_DHCP
nameserver 8.8.8.8
```
```
[lionel@node1 ~]$ ping google.com
PING google.com (142.250.179.110) 56(84) bytes of data.
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=1 ttl=115 time=17.5 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=2 ttl=115 time=20.4 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=3 ttl=115 time=20.2 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=4 ttl=115 time=19.7 ms
```


🌞 **Capture Wireshark**

🦈 **`tp4_dhcp_server1.pcapng`**


