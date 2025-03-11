# 3. Partie 2 - Supervision et métrologie avec SNMP
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
SNMPv2-MIB::sysLocation.0 = STRING: "ETRS813 - BinC4me10 Routeur 2 - UniversitC) Savoie Mont Blanc"
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
On s'intéresse ici à la table "vrrpOperTable". Son OID est mib-2.68.1.3
Les 8 premières colonnes sont les suivantes :
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
### <u> Question 15 </u>



### <u> Question 16 </u>



### <u> Question 17 </u>

