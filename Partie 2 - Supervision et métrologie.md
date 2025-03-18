# 4. Partie 2 - Supervision et métrologie avec SNMP
Objectifs
- Mise en place d'un agent SNMPv3 et SNMPv2
- Sécurisation des trames SNMP
- Etude de la métrologie via SNMP

**************************************************
# 4.1 Configuration de SNMPv3 dans les routeurs
Après avoir configuré SNMPv3 sur les routeurs, nous avons installé les outils snmp sur les machines A et B afin de réaliser des tests.  
Nous avons par la suite testé de récupérer l'hostname du routeur 813-B10-R1 via la commande ci-dessous. Nous avons fait la même chose pour tester la configuration de 813-B10-R2
```
[etudiant@813-B10-B ~]$ snmpget -v3 -u snmpuser -l authPriv -a SHA -A auth_pass -x AES -X crypt_pass 10.200.2.254 sysName.0
```
Le résultat est correct : ``` SNMPv2-MIB::sysName.0 = STRING: 813-B10-R1 ```


### <u> Question 8 </u>
Afin de récupérer l'objet "syslocation" nous utilisons la commande snmpget suivante :  
```
snmpget -v3 -u snmpuser -l authPriv -a SHA -A auth_pass -x AES -X crypt_pass <IP_ROUTEUR> sysLocation.0
```
Par exemple, pour le routeur 813-B10-R2, nous avons le résultat suivant :  
```
SNMPv2-MIB::sysLocation.0 = STRING: "ETRS813 - Binôme10 Routeur 2 - Université Savoie Mont Blanc"
```
C'est bel et bien la valeur que nous avons configuré précèdemment. La configuration sur R2 est donc correcte et la commande également.

**************************************************
# 4.2 Configuration de SNMPv2 dans les routeurs
Sur A, testons le fonctionnement de notre configuration SNMPv2 en tentant de récupérer l'hostname du routeur R2, qui correspond à l'OID 1.3.6.1.2.1.1.5.0 (Recherché sur Internet).  
Pour cela on va utiliser la commande ci-dessous  
```
snmpget -v2c -c 123test123 10.200.2.253 1.3.6.1.2.1.1.5.0
```
Le résultat est correct : ``` SNMPv2-MIB::sysName.0 = STRING: 813-B10-R2 ```  
La configuration de SNMPv2 est donc correcte.  


### <u> Question 9 </u>
SNMP utilise l'encodage ASN.1 (définit des structures de données) avec le BER (Basic Encoding Rule) lors de l'émission de données.


### <u> Question 10 </u>
Afin de récupérer les informations sur le MTU de la 2e interface d'un de nos routeurs nous devons connaitre le OID associé à ce MTU. Suite à une recherche internet nous avons déterminé cet OID : 1.3.6.1.2.1.2.2.1.4.2  
Pour réaliser la capture nous avons :  
1) Préparer une capture de traffic dans un premier terminal sur la machine A avec la commande ``` sudo tshark -i enp0s8 -Y "snmp" -O snmp -x ```
2) Utiliser la commande ``` snmpget -v2c -c 123test123 10.200.2.254 1.3.6.1.2.1.2.2.1.4.2 ``` sur un deuxieme terminal sur A  

Nous capturons ainsi la requête SNMP-GET envoyé par A au routeur 813-B10-R1 et la réponse du routeur.  
Trame requête :  
```
0000  08 00 27 f4 4c 96 08 00 27 30 bb 08 08 00 45 00   ..'.L...'0....E.
0010  00 4d 4b 45 40 00 40 11 d4 c3 0a c8 02 0a 0a c8   .MKE@.@.........
0020  02 fe bc 13 00 a1 00 39 1a e2 30 2f 02 01 01 04   .......9..0/....
0030  0a 31 32 33 74 65 73 74 31 32 33 a0 1e 02 04 2d   .123test123....-
0040  ce 9b 1c 02 01 00 02 01 00 30 10 30 0e 06 0a 2b   .........0.0...+
0050  06 01 02 01 02 02 01 04 02 05 00                  ...........
```

Trame réponse :  
```
0000  08 00 27 30 bb 08 08 00 27 f4 4c 96 08 00 45 00   ..'0....'.L...E.
0010  00 4f 00 06 00 00 ff 11 a1 00 0a c8 02 fe 0a c8   .O..............
0020  02 0a 00 a1 bc 13 00 3b 42 74 30 31 02 01 01 04   .......;Bt01....
0030  0a 31 32 33 74 65 73 74 31 32 33 a2 20 02 04 2d   .123test123. ..-
0040  ce 9b 1c 02 01 00 02 01 00 30 12 30 10 06 0a 2b   .........0.0...+
0050  06 01 02 01 02 02 01 04 02 02 02 05 dc            .............
```

Analysons ces trames :
|Type de trame | En-tête IP  | En-tête UDP | Version | Communauté  | PDU Type  | Id requête | Status d'erreur | Index d'erreur | Nom (OID) | Valeur |
|--------------|-------------|-------------|---------|-------------|-----------|------------|-----------------|----------------|-----------|--------|
| Requête | @IP_src : 10.200.2.10 / @IP_dst : 10.200.2.254 | 48147 → 161 | 2 (SNMPv2) | test123 | GET (A0) | 2DCE9B1C | 0 | 0 | 1.3.6.1.2.1.2.2.1.4.2 | ? (Pas de valeur sur une requête) |
| Réponse | @IP_src : 10.200.2.254 / @IP_dst :  10.200.2.10 | 161 → 48147 | 2 (SNMPv2) | test123 | RESPONSE (A2) | 2DCE9B1C | 0 | 0 | 1.3.6.1.2.1.2.2.1.4.2 | 1500 |

On observe donc bien la valeur d'OID demander via la commande ci-dessus. La version SNMP est bien la version 2 et le port utilisé est UDP. Cela correspond à ce qu'on a vu en cours.

### <u> Question 11 </u>
Dans cette question nous nous intéressons à la MIB VRRP ainsi que le fichier contenant la description de cette dernière. Après avoir effectué des recherches sur Internet nous avons relevé la ligne indiquant l'OID relatif de la branche VRRP.
```vrrp OBJECT IDENTIFIER ::= { mib-2 68 }```

En en conclut l'OID complet : 1.3.6.1.2.1.68

### <u> Question 12 </u>
Nous avons exécuté la commande ```snmpwalk -v2c -c 123test123 10.100.2.254 vrrpMIB```, mais celle-ci a échoué avec l'erreur : "vrrpMIB: Unknown Object Identifier (Sub-id not found: (top) -> vrrpMIB)".  
En revanche, la commande ```snmpwalk -v2c -c 123test123 10.100.2.254 mib-2.68``` a fonctionné correctement.

L'échec de la première commande est dû au fait que l'agent SNMP ne reconnaît pas vrrpMIB comme un OID valide. Dans la seconde commande, l'OID numérique est utilisé directement, ce qui permet son identification par l'agent. Autrement dit, la correspondance entre vrrpMIB et la valeur 68 n'a pas été réalisée par l'agent SNMP.

### <u> Question 13 </u>
On s'intéresse ici à la table "vrrpOperTable". Son OID est mib-2.68.1.3 .Les 8 premières colonnes sont les suivantes :  
| OID                               | Valeur                 | Explication |
|---------------------------------------|----------------------------|-----------------|
| `mib-2.68.1.3.1.2.2.1`  | `00 00 5E 00 01 01`      | Adresse MAC VRRP |
| `mib-2.68.1.3.1.3.2.1`  | `3`                      | État du routeur VRRP (3 = `master`, 2 = `backup`, 1 = `init`) |
| `mib-2.68.1.3.1.4.2.1`  | `1`                      | Nombre d'adresses IP associées au routeur virtuel |
| `mib-2.68.1.3.1.5.2.1`  | `200`                    | Priorité du routeur |
| `mib-2.68.1.3.1.6.2.1`  | `1`                      | Mode de préemption (1 = activé, 2 = désactivé) |
| `mib-2.68.1.3.1.7.2.1`  | `10.100.2.252`           | Adresse IP du routeur maître actuel |
| `mib-2.68.1.3.1.8.2.1`  | `10.100.2.254`           | Adresse IP principale du routeur VRRP |
| `mib-2.68.1.3.1.9.2.1`  | `1`                      | Version VRRP utilisée (`1 = VRRPv2`, `2 = VRRPv3`) |


**************************************************
# 4.3 Métrologie
Dans cette partie nous nous intéressons aux possibilités offertes par SNMP pour observer les performances du réseau.  
Désactiver le processus faisant pare-feu sur B fût nécessaire afin de réaliser l'étude.  

### <u> Question 14 </u>
Le protocole de transport utilisé pour le test de débit entre les deux machines A et B est ``` TCP ``` (utilisé par défaut). Le protocole UDP peut-être utilisé avec l'option -u. (iperf3 -c <@IP_B> -u)  
Iperf3 réalise 10 mesures, une toutes les secondes, soit 10s pour la durée de la mesure.  
On observe un débit moyen de 2,62 Gbits/s entre nos 2 machines.  
```
[root@813-B10-A etudiant]# iperf3 -c 10.200.2.11
Connecting to host 10.200.2.11, port 5201
[  5] local 10.200.2.10 port 41148 connected to 10.200.2.11 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   268 MBytes  2.25 Gbits/sec  906    247 KBytes       
[  5]   1.00-2.00   sec   319 MBytes  2.68 Gbits/sec  990    189 KBytes       
[  5]   2.00-3.00   sec   282 MBytes  2.36 Gbits/sec  1035    245 KBytes       
[  5]   3.00-4.00   sec   339 MBytes  2.85 Gbits/sec  647    209 KBytes       
[  5]   4.00-5.00   sec   286 MBytes  2.40 Gbits/sec  857    182 KBytes       
[  5]   5.00-6.00   sec   297 MBytes  2.50 Gbits/sec  1215    182 KBytes       
[  5]   6.00-7.00   sec   313 MBytes  2.63 Gbits/sec  1395    215 KBytes       
[  5]   7.00-8.00   sec   340 MBytes  2.86 Gbits/sec  1358    170 KBytes       
[  5]   8.00-9.00   sec   313 MBytes  2.62 Gbits/sec  1157    300 KBytes       
[  5]   9.00-10.00  sec   365 MBytes  3.06 Gbits/sec  1575    199 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  3.05 GBytes  2.62 Gbits/sec  11135             sender
[  5]   0.00-10.00  sec  3.05 GBytes  2.62 Gbits/sec                  receiver
```

### <u> Question 15 </u>
Sur B nous réalisons une capture de trame sur l'interface LAN (réseau 10.200.2.0/24) via la commande : ```tshark -w /tmp/capture-500k.pcap -i enp0s8 "udp port 5201"```
Sur A nous lançons la commande iperf en précisant le protocole UDP comme protocole à utilisé (option -u) et on fixe la bande passante (option -b) à 500k.
Après visualisé la capture avec la commande ```capinfos``` on observe un débit de 515kbits/s, ce qui est légérement plus élevé que le résultat annoncé par iperf3.

Le débit calculé par capinfos est plus élevé que le débit généré par iperf3 car capinfos réalise un calcule de débit brut, c'est à dire que les en-têtes (Ethernet, IP, TCP) sont comprises dans le calcul.  

### <u> Question 16 </u> 
Si nous réalisons l'observation avec les compteurs 32 bits il nous faudra utiliser les OID suivants :  
- ifInOctets : 1.3.6.1.2.1.2.2.1.10  
- ifOutOctets : 1.3.6.1.2.1.2.2.1.16  
On utilisera ces OID pour observer un faible débit (max 4 Go). Dans notre cas le traffic sur notre réseau est très faible, nous allons donc utiliser des compteurs 32 bits.

Et en 64 bits nous utiliserons les OID suivants :  
- ifHCInOctets : 1.3.6.1.2.1.31.1.1.1.6  
- ifHCOutOctets : 1.3.6.1.2.1.31.1.1.1.10  
A contrario, ces compteurs sont recommandés lorsque le volument de trafic est élevé.  

### <u> Question 17 </u>
Manipulation simple permmettant de trouver le débit entrant :  
1) Lancer la commande ``` iperf3 -c 192.168.141.87 -t 30 -b 1M ``` sur A (Bien évidemment la commande iperf3 -s a été lancée sur B). Elle permet, via l'option "-t", de générer un flux via iperf mais plus longtemps que par défaut. 192.168.141.87 est l'adresse IP que B a obtenu sur le VLAN140
3) Lancer le script sur A, mesurant le débit sortant (nommé 813-Q17-debit_out.sh).  
   Le .3 à la fin de l'OID fait référence à l'interface GigabitEthernet3 du routeur (interface qui est au niveau du réseau 10.250.0.0/24)  
   ```
    TX1=$(snmpget -v2c -c 123test123 -Oqv 10.200.2.254 1.3.6.1.2.1.2.2.1.16.3) # Mesure du nombre d'octets envoyés
    sleep 10
    TX2=$(snmpget -v2c -c 123test123 -Oqv 10.200.2.254 1.3.6.1.2.1.2.2.1.16.3) # Mesure du nombre d'octets envoyés après 10 secondes
    echo "Débit sortant: $(( (TX2 - TX1) * 8 / 10 )) bits/s" # Calcul du débit sortant en bits par seconde
   ```
On obtient presque la même valeur via la commande snmpget et la commande iperf3.  
Script -> Débit sortant: 1096766 bits/s, soit 1,10 Mbits/s.  
Iperf3 -> Débit sortant : 1,01 Mbits/s.  



