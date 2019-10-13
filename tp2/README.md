Rendu Deuxième TP, Louis MAURY

# I. Simplest setup

## 1) Faire communiquer les deux PCs

> *Mais, je sais déjà comment les faire communiquer !* :confused:

Et quel est le protocole utilisé par un `ping` ?

> ...

Pour cette première partie nous allons utiliser la topologie ci-dessous:
```
+------+      e0 +------+ e1     +------+
| VPC1 +---------+ IOU1 +--------+ VPC2 |
+------+ e0      +------+     e0 +------+
```
Avec les adresses IP suivantes:
| Machine  |     IP     |
|----------|------------|
| VPC1     | 10.2.1.1/24|
| VPC2     | 10.2.1.2/24|
---

Comme on peut le constater, le **VPC1** est connecté de son interface `e0` à l'interface `e0` du Switch **IOU1**.
De même pour le **VPC2** qui est connecté du `e0` au `e1`.

Maintenant on va envoyer un `ping` de **VPC1** à **VPC2** en analysant le lien entre **VPC1** et **IOU1** (que nous appellerons **0->0**):
```
VPC1> ping 10.2.1.2
84 bytes from 10.2.1.2 icmp_seq=1 ttl=64 time=0.481 ms
84 bytes from 10.2.1.2 icmp_seq=2 ttl=64 time=1.177 ms
84 bytes from 10.2.1.2 icmp_seq=3 ttl=64 time=4.887 ms
```
Bon le ping fonctionne regardons maintenant du coté de Wireshark:
```
    Source    | Destination  |  Protocol
1|  10.2.1.1	10.2.1.2	    ICMP
2|  10.2.1.2	10.2.1.1	    ICMP
```
Nous pouvons remarquer qu'à la première ligne, le `ping` part de **`10.2.1.1`** (**VPC1**) et va à **`10.2.1.2`** (**VPC2**) mais on peut aussi voir que sur la ligne en dessous il a la réponse au `ping` de **VPC2** à **VPC1**

> *Mais, c'est quoi le truc à côté ?* :hushed:

Ça ? C'est tout simplement le nom du protocole `ICMP`

> *Et il sert à quoi ce protocole ?* :thinking:

***"*** **I**nternet **C**ontrole **M**essage **P**rotocol ou **ICMP** est un protocole qui permet de controler des erreurs de transmission ***"*** \
*Source: [Wikipedia](https://fr.wikipedia.org/wiki/Internet_Control_Message_Protocol)*

> *Ah c'est pour ça qu'il y a un ping d'aller et un de retour ?* :nerd_face:

Tout à fait, si la machine de départ reçoit la réponse sans erreur, alors le `ping` fonctionne.

---

Jetons un oeil à la table `ARP` de **VPC1** maintenant:
```
PC1> show arp

00:50:79:66:68:01  10.2.1.2 expires in 68 seconds
```
On voit l'addresse ip du **VPC2** mais aussi son addresse MAC.
On voit également que la table ARP va se vider dans 68 secondes mais ça on s'en occupe pas.

Du côté de wireshark:
```
    Source            | Destinatipon      | Protocol  |  Length  |  Info
1|  Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.1.2? Tell 10.2.1.1
2|  Private_66:68:01	Private_66:68:00	ARP	         64	        10.2.1.2 is at 00:50:79:66:68:01
```
Nous pouvons clairement remarquer que, lors du `ping` **VPC1** demande à tout le réseau (**Broadcast**) qui possède l'addresse **IP** **`10.2.1.2`** (`Who has 10.2.1.2?`) et le **VPC2** lui répond en disant que c'est bien son **IP** avec son addresse MAC **`00:50:79:66:68:01`**.

> *Mais il y a un problème ! Comment ça se fait qu'il y ai un switch au milieu des deux VPCs mais qu'il n'ait pas d'IP ?* :thinking:

Un Switch ne possède pas d'IP car il n'en a pas besoin, il ne fait que transmettre un message d'un poste au autres postes du réseau. Ici il ne sert pas a grand chose étant donné qu'il n'y à que deux PC branchés mais si il avait plus de deux VPCs branchés, le switch aurait été plus utile.

> *Donc si je comprends bien, les VPCs ont besoin d'une adresse **IP** pour pouvoir être identifié sur le réseau. Par exemple le **VPC1** utilise l'**IP** de **VPC2** pour pouvoir demander via le **switch** qu'elle VPC possède cette **IP**. C'est alors que le **VPC2** lui repond en donnant son adresse **MAC** afin qu'ils puissent communiquer tranquillement.*

Exactement ! Le **VPC1** envoie la demande "Est-ce que quelqu'un à l'addresse IP `10.2.1.2` ?" au switch, ce dernier l'envoie alors à tous les autres VPCs du réseau. Une fois que le **VPC2** à reçu la demande il renvoie la réponse au switch qui renvoie la demande à tous les VPCs du réseau. Les VPCs qui ne sont pas concernés par la demande ignorent le message.

---
---

# II. More switches

Pour cette partie, nous allons mettre en place la topologie suivante:
```
                              +-------+
                              |  VPC2 |
                              +---+---+
                                e0|
                                  |
                                  |
                                  |e0
                              +---+---+
                        +-----+ IOU2  +-----+
                        |   e1+-------+e2   |
                        |                   |
                      e1|                   |e1
+------+            +---+---+           +---+---+         e0+------+
| VPC1 +------------+ IOU1  +-----------+  IOU3 +-----------+ VPC3 |
+------+ e0       e0+-------+e2       e2+-------+e0         +------+
```
Avec les adresses **IP** suivantes:

|  Machines |      IPs      |
|-----------|---------------|
|   VPC1    | `10.2.2.1/24` |
|   VPC2    | `10.2.2.2/24` |
|   VPC3    | `10.2.2.3/24` |

- Faire communiquer les VPCs:\
    Pour ça il suffit juste de faire la commande `ping`:
    ```
    PC1> ping 10.2.2.2         
    84 bytes from 10.2.2.2 icmp_seq=1 ttl=64 time=4.213 ms
    84 bytes from 10.2.2.2 icmp_seq=2 ttl=64 time=2.575 ms

    PC1> ping 10.2.2.3
    84 bytes from 10.2.2.3 icmp_seq=1 ttl=64 time=2.137 ms
    84 bytes from 10.2.2.3 icmp_seq=2 ttl=64 time=2.028 ms
    ```
    Nos `ping` fonctionnent !
- Analyser le table MAC d'un switch:\
    Pour ça rien de bien compliqué, pour voir la table MAC il suffit de taper la commande suivante:
    ```
    IOU1#show mac address-table
          Mac Address Table
    -------------------------------------------

    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    1       0050.7966.6800    DYNAMIC     E0
    1       0050.7966.6801    DYNAMIC     E1
    1       0050.7966.6802    DYNAMIC     E2
    1       aabb.cc00.0210    DYNAMIC     E1
    1       aabb.cc00.0310    DYNAMIC     E1
    1       aabb.cc00.0320    DYNAMIC     E2
    Total Mac Addresses for this criterion: 6
    ```
    Sur les trois première lignes, le switch **IOU1** envoie le paquet sur les ports e0, e1 et e2 (qui sont branchés à **VPC1**, **IOU2** et **IOU3**).\
    Puis sur les trois dernière lignes ce sont les messages qu'il reçoit (les retours des deux switchs **IOU2** et **IOU3**).

    > *Mais pourquoi il reçoit deux fois le retour du IOU2 ?* :thinking:

    Il reçoit deux fois la réponse car le switch **IOU3** envoie la réponse à tout le monde. Comme **IOU2** reçoit deux message à transmettre et bien il l'envoie deux fois.
- > Je me suis un peu amusé avec Wireshark et j'ai lancé l'écoute sur le line qui relie les switches **IOU2** et **IOU3** et j'ai remarqué un protocole `CDP`... Mais c'est quoi ça ? :worried:

    ***"*** **C**isco **D**iscovery **P**rotocol ou **CDP** est un protocole propriétaire de Cisco qui permet de communiquer indépendament du réseau, c'est à dire, qu'il ne fait pas partie du modèle `OSI`. Il est utilisé la plupart du temps pour communiquer les informations sur les propriétés des ports, des connexions et des périphériques ***"***\
    Source: [Cisco](https://www.cisco.com/c/fr_ca/support/docs/smb/switches/cisco-small-business-200-series-smart-switches/smb982-cisco-discovery-protocol-cdp-properties-on-200-300-series-ma.html)

- Determiner les informations `STP`
    > *C'est quoi STP ?* :hushed:

    ***"*** `STP` est un protocole qui permet d'éviter les boucles réseau c'est à dire qu'il va éviter que les switch ne s'envoient indéfiniment des paquets et que ça ne bouche le réseau. ***"***\
    Source: [Ici](https://gitlab.com/it4lik/b2-reseau-2019/blob/master/cours/1.md#stp)

    Nous allons maintenant voir quel switch IOU est un `route bridge` et quels ports sont désactivés:

    ### Pour le **IOU1**:

    ```
    IOU1#show spanning-tree        

    VLAN0001
    Spanning tree enabled protocol rstp
    Root ID    Priority    32769
                Address     aabb.cc00.0100
                This bridge is the root
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

    Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                Address     aabb.cc00.0100
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec

    Interface           Role Sts Cost      Prio.Nbr Type
    ------------------- ---- --- --------- -------- --------------------------------
    Et0/0               Desg FWD 100       128.1    Shr 
    Et0/1               Desg FWD 100       128.2    Shr 
    Et0/2               Desg FWD 100       128.3    Shr 
    ```

    ### Pour le **IOU2**
    ```
    IOU2#show spanning-tree       

    VLAN0001
    Spanning tree enabled protocol rstp
    Root ID    Priority    32769
                Address     aabb.cc00.0100
                Cost        100
                Port        2 (Ethernet0/1)
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

    Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                Address     aabb.cc00.0200
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec

    Interface           Role Sts Cost      Prio.Nbr Type
    ------------------- ---- --- --------- -------- --------------------------------
    Et0/0               Desg FWD 100       128.1    Shr 
    Et0/1               Root FWD 100       128.2    Shr 
    Et0/2               Desg FWD 100       128.3    Shr 
    ```
    ### Pour le **IOU3**
    ```
    IOU3#show spanning-tree       

    VLAN0001
    Spanning tree enabled protocol rstp
    Root ID    Priority    32769
                Address     aabb.cc00.0100
                Cost        100
                Port        3 (Ethernet0/2)
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

    Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                Address     aabb.cc00.0300
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec

    Interface           Role Sts Cost      Prio.Nbr Type
    ------------------- ---- --- --------- -------- --------------------------------
    Et0/0               Desg FWD 100       128.1    Shr 
    Et0/1               Altn BLK 100       128.2    Shr 
    Et0/2               Root FWD 100       128.3    Shr
    ```

    > *C'est super tout ça mais qu'est-ce que je dois comprendre ?* :disappointed_relieved:

    Et bien on remarque que c'est le **IOU1** qui est le `route bridge` avec la ligne **`This bridge is the root`**
    mais aussi que **IOU3** a bloqué le port e1(**IOU2**). Il ne va donc recevoir que les paquets de **IOU1**

    Pour faire simple mettons tout ça dans un tableau:

    |    Switche    |    Role         |    Ports ouverts       |    Ports Fermés    |
    |---------------|-----------------|------------------------|--------------------|
    | **IOU1**      | `Route Bridge`  |`e0` ¦ `e1` ¦ `e2`      |     ***`X`***      |
    | **IOU2**      | "Passif"        |`e0` ¦ `e1` ¦ `e2`      |     ***`X`***      |
    | **IOU3**      | "Passif"        |`e0` ¦ ***`X`*** ¦ `e2` |       `e1`         |

    > *Donc le IOU3 ne peut pas communiquer directement avec le IOU2 ?* :open_mouth:

    Si c'est juste qu'il ne l'écoute pas mais pour vérifier ça nous allons effectuer un `ping` du **VPC1** vers le **VPC3**. Comme ça nous allons pouvoir analyser les trames
    qui passent entre les switches.

    ```
    PC1> ping 10.2.2.3
    84 bytes from 10.2.2.3 icmp_seq=1 ttl=64 time=3.870 ms
    84 bytes from 10.2.2.3 icmp_seq=2 ttl=64 time=2.565 ms
    84 bytes from 10.2.2.3 icmp_seq=3 ttl=64 time=2.445 ms
    84 bytes from 10.2.2.3 icmp_seq=4 ttl=64 time=3.052 ms
    84 bytes from 10.2.2.3 icmp_seq=5 ttl=64 time=2.170 ms
    ```
    Le `ping` s'effectue correctement, super !
    Regardons maintenant du coté du lien entre **IOU1** et **IOU2**:
    ```
    Source            | Destinatipon      | Protocol  |  Length  |  Info
    Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.2.3? Tell 10.2.2.1
    Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.2.3? Tell 10.2.2.1
    ```
    Il y a l'échange qui s'effectue deux fois mais c'est normal car c'est **IOU2** qui renvoie la question à **IOU1** mais ça normalement ça commence à être compris.

    Regardons maintenant le lien entre **IOU2** et **IOU3**:
    ```
    Source            | Destinatipon      | Protocol  |  Length  |  Info
    Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.2.3? Tell 10.2.2.1
    Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.2.3? Tell 10.2.2.1
    Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.2.3? Tell 10.2.2.1
    Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.2.3? Tell 10.2.2.1
    ```
    > *Oulalala mais il y en a le double ! Comment ça se fait ?!* :disappointed_relieved:
    
    Et oui, il en a beaucoup plus car **IOU3** reçois le paquet de **IOU1** et **IOU2**(mais ne l'écoute pas).
     Il va donc transmettre le paquet à **IOU2** et **IOU1**, etc... jusqu'à ce **IOU3** bloque tous les messages qui viennent de **IOU2**.

    Mais si on regarde le lien entre **IOU1** et **IOU3** on peut voir des choses bien différentes:
    ```
    Source            | Destinatipon      | Protocol  |  Length  |  Info
    Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.2.3? Tell 10.2.2.1
    Private_66:68:00	Broadcast	        ARP	         64	        Who has 10.2.2.3? Tell 10.2.2.1
    Private_66:68:02	Private_66:68:00	ARP	         64	        10.2.2.3 is at 00:50:79:66:68:02
    Private_66:68:02	Private_66:68:00	ARP	         64	        10.2.2.3 is at 00:50:79:66:68:02
    10.2.2.1	        10.2.2.3	        ICMP	     98	        Echo (ping) request  id=0xf9ee, seq=1/256, ttl=64 (reply in 35)
    10.2.2.3	        10.2.2.1	        ICMP	     98	        Echo (ping) reply    id=0xf9ee, seq=1/256, ttl=64 (request in 32)
    ```
    > *Ah on peut voir que le ping est passé par là !* :smiley:

    En effet le `ping` est bien passé par là, on peut voir également la réponse à la demande d'`IP` qui passe 
    deux fois comme on l'a vu précedemment pour le lien **IOU1**->**IOU2**.

    Mais pour clarifier tout ça il faut faire un schéma:

    ```
                                  +-------+
                                  |  VPC2 |
                                  +---+---+
                                    e0|
                                      |
                                      |
                                      |e0
                                  +---+---+
                            +-----+ IOU2  +-----+
                            |   e1+-------+e2   |
                            |                   |
                          e1|                   |e1
    +------+            +---+---+           +---+---+         e0+------+
    | VPC1 +------------+ IOU1  +-----------+  IOU3 +-----------+ VPC3 |
    +------+ e0       e0+-------+e2       e2+-------+e0         +------+
    ```
    Le paquet sera représenté par un `=>`\
    Le paquet part donc du **VPC1** vers le **IOU1** qui le partage aux autres switches:
    ```
                                  +-------+
                                  |  VPC2 |
                                  +---+---+
                                      |
                                      |
                                      |
                                      |
                                  +---+---+
                            +-----+ IOU2  +-----+
                            |     +-------+     |
                            ^                   |
                            ║                   |
                            |                   |  
    +------+            +---+---+           +---+---+           +------+
    | VPC1 +----=>------+ IOU1  +----=>-----+  IOU3 +-----------+ VPC3 |
    +------+            +-------+           +-------+           +------+
    ```
    Les switches peuvent maintenant partager cette information avec les autres machines:
    ```
                                  +-------+
                                  |  VPC2 |
                                  +---+---+
                                      |
                                      ^
                                      ║
                                      |
                                  +---+---+
                            +-----+ IOU2  +-----+
                            |     +-------+     |
                            |                   ^
                            ║                   ║
                            v                   v
                            |                   |  
    +------+            +---+---+           +---+---+           +------+
    | VPC1 +----<=------+ IOU1  +----<=-----+  IOU3 +----=>-----+ VPC3 |
    +------+            +-------+           +-------+           +------+
    ```
    Le **VPC3** va pouvoir lui répondre:
    ```
                                  +-------+
                                  |  VPC2 |
                                  +---+---+
                                      |
                                      ^
                                      ║
                                      |
                                  +---+---+
                            +-----+ IOU2  +-----+
                            |     +-------+     |
                            ^                   ^
                            ║                   ║
                            |                   v
                            |                   |  
    +------+            +---+---+           +---+---+           +------+
    | VPC1 +----<=------+ IOU1  +----<=-----+  IOU3 +----<=-----+ VPC3 |
    +------+            +-------+           +-------+           +------+
    ```