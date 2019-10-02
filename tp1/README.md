# Rendu Premier TP, Louis MAURY

## I. Gather informations

### Récupérer la liste des cartes réseau

Pour récupérer la liste des cartes réseau, il suffit de taper la commande suivante:

```shell
[centos8@localhost ~]$ ip a
enp0s3:link/ether 08:00:27:a8:84:87 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
enp0s8:link/ether 08:00:27:c8:7f:08 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.102/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s8
```
---
### Déterminer si les cartes ont récupéré une IP en DHCP ou non

Pour voir si la carte enp0s3 a une IP attribué par un DHCP il suffit de taper la commande suivante :
```shell
[centos8@localhost ~]$ sudo nmcli -f DHCP4 con show enp0s3
DHCP4.OPTION[1]:                        domain_name = auvence.co

DHCP4.OPTION[2]:                        domain_name_servers = 10.33.10.20 10.33.10.2 8.8.8.8

DHCP4.OPTION[3]:                        expiry = 1569590932
DHCP4.OPTION[4]:                        ip_address = 10.0.2.15
DHCP4.OPTION[5]:                        requested_broadcast_address = 1
DHCP4.OPTION[6]:                        requested_dhcp_server_identifier = 1
DHCP4.OPTION[7]:                        requested_domain_name = 1
DHCP4.OPTION[8]:                        requested_domain_name_servers = 1
DHCP4.OPTION[9]:                        requested_domain_search = 1
DHCP4.OPTION[10]:                       requested_host_name = 1
DHCP4.OPTION[11]:                       requested_interface_mtu = 1
DHCP4.OPTION[12]:                       requested_ms_classless_static_routes = 1
DHCP4.OPTION[13]:                       requested_nis_domain = 1
DHCP4.OPTION[14]:                       requested_nis_servers = 1
DHCP4.OPTION[15]:                       requested_ntp_servers = 1
DHCP4.OPTION[16]:                       requested_rfc3442_classless_static_routes = 1
DHCP4.OPTION[17]:                       requested_routers = 1
DHCP4.OPTION[18]:                       requested_static_routes = 1
DHCP4.OPTION[19]:                       requested_subnet_mask = 1
DHCP4.OPTION[20]:                       requested_time_offset = 1
DHCP4.OPTION[21]:                       requested_wpad = 1
DHCP4.OPTION[22]:                       routers = 10.0.2.2
DHCP4.OPTION[23]:                       subnet_mask = 255.255.255.0
```
On peut voir en *OPTION[2]* "`domain_name_servers = 10.33.10.20 10.33.10.2 8.8.8.8`" ce qui veut dire que la carte enp0s3 obtient son IP depuis un des serveurs DHCP dont les adresses sont renseigné.

---
### Afficher la table de routage et la table ARP

Pour afficher la table de routage il faut taper la commande suivante:
```shell
[centos8@localhost ~]$ netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG        0 0          0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U         0 0          0 enp0s3
192.168.56.0    0.0.0.0         255.255.255.0   U         0 0          0 enp0s8
```
Et pour la table ARP:
```shell
[centos8@localhost ~]$ arp -e
Address          HWtype  HWaddress           Flags Mask     Iface
192.168.56.1     ether   0a:00:27:00:00:00   C              enp0s8
10.0.2.2         ether   52:54:00:12:35:02   C              enp0s3
```
> Mais qu'est-ce que ça veut dire tout ça ?

#### Pour la table de routage:

`10.0.2.2` correspond à l'adresse du routeur qui permet à la carte enp0s3 de se connecter au réseau `10.0.2.0` et de recevoir son IP du DHCP.

#### pour la table ARP:
`192.168.56.1` correspond à l'adresse de la carte du reseau privé d'hote de VirtualBox et `10.0.2.2` correspond toujours à l'adresse du gateway (router).

---

### Récupérer la liste des ports en écoute (TCP et UDP)
Pour récupérer la liste des ports sur écoute il suffit de taper la commande `[centos8@15 ~]$ netstat -l`.

- Pour voir juste les ports qui écoutent en TCP il faut ajouter l'option `-lt`:
    ```shell
    [centos8@15 ~]$ netstat -lt
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State      
    tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
    tcp6       0      0 [::]:ssh                [::]:*                  LISTEN 
    ```

- Pour voir les ports qui écoutent en UDP il faut mettre l'option `-lu`:
    ```shell
    [centos8@15 ~]$ netstat -lu
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address               Foreign Address         State      
    udp        0      0 15.2.0.10.rev.sf:bootpc     0.0.0.0:*                          
    udp        0      0 15.2.0.10.rev.sf:bootpc     0.0.0.0:*                          
    udp        0      0 localhost:323               0.0.0.0:*                          
    udp6       0      0 localhost:323               [::]:*                             
    ```
---

### Récuperer la liste des DNS
Pour faire une requête DNS vers le site *www.reddit.com* il suffit de faire la commande suivante:
```shell
[centos8@15 ~]$ dig www.reddit.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-17.P2.el8_0.1 <<>> www.reddit.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10672
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.reddit.com.                        IN      A

;; ANSWER SECTION:
www.reddit.com.         3600    IN      CNAME   reddit.map.fastly.net.
reddit.map.fastly.net.  3600    IN      A       151.101.121.140

;; Query time: 18 msec
;; SERVER: 10.0.2.3#53(10.0.2.3)
;; WHEN: Fri Sep 27 13:42:38 EDT 2019
;; MSG SIZE  rcvd: 83
```

En tapant cette commande on voit que en faisant cette requete on passe par le server à l'adresse `10.0.2.3` qui est sur le meme réseau que notre VM (`10.0.2.15`).

---

### Afficher l'état du firewall
Pour afficher l'état du firewall il faut taper la commande suivante:
```shell
[centos8@localhost ~]$ systemctl status firewalld.service 
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2019-09-28 10:15:12 EDT; 6min ago
     Docs: man:firewalld(1)
 Main PID: 781 (firewalld)
    Tasks: 2 (limit: 17712)
   Memory: 29.5M
   CGroup: /system.slice/firewalld.service
           └─781 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --nopid

Sep 28 10:15:11 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
Sep 28 10:15:12 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
```
On peut voir que le firewall est en marche.

En effectuant la commande `firewall-cmd --list-all`, on peut voir toutes sortes de choses:
```shell
[centos8@localhost ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```
Comme par exemple que les interfaces *enp0s3* et *enp0s8* sont filtrées. On peut également voir que aucun port n'est autorisé/filtré.

---

## II. Edit configuration
Voici la configuration de la carte en réseau privé :
```
DEVICE=enp0s8
BOOTPROTO=none
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=192.168.56.10
```
et Voici celle de la nouvelle carte en réseau privé:
```
DEVICE=enp0s9
BOOTPROTO=none
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=192.168.57.20
```
