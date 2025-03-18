# 3. Partie 1 - Mise en place d'une maquette de réseau local avec haute disponibilité
Objectifs
- Réaliser une étude préparatoire sur la maquette
- Mettre en place les différentes machines (PC / Routeur)
- Réaliser la configuration réseau des machines

**************************************************
# 3.1 Schéma du réseau

Le réseau déployé lors de ce mini-projet est le suivant.
<picture>
 <source media="(prefers-color-scheme: dark)" srcset="https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/ETRS813%20-%20B10%20-%20Mini-projet%20Supervision.png">
 <source media="(prefers-color-scheme: light)" srcset="https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/ETRS813%20-%20B10%20-%20Mini-projet%20Supervision.png">
 <img alt="YOUR-ALT-TEXT" src="https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/ETRS813%20-%20B10%20-%20Mini-projet%20Supervision.png">
</picture>

**************************************************
# 3.2 Etude théorique préparatoire
### <u> Question 1 </u>
Dans la théorie, nous utilisons le protocole OSPF comme protocole de routage. Nous apprenons donc :
- x9 routes vers le sous-réseau des autres binômes. (On peut compter 2 fois (donc x18) vu qu'il y a 2 next-hop pour aller aux réseau internes).
- x2 route vers les sous-réseaux directement connectés (Le notre et le VLAN633)
- x1 loopback du routeur
- x20 routes vers les loopbacks des routeurs des autres binômes
- x1 route vers le sous-réseau "Prof".
- x2 routes vers les sous-réseau VLAN140 et VLAN176
- x2 routes par défault utilisant les routeurs RPROF1 et RPROF2 comme passerelle par défault.

Donc au final, 47 routes seront présentes dans notre table de routage.

Pour le routeur R1 nous pouvons avoir une table de routage comme présenté ci-dessous (Le coût de référence est 10 (10x le débit du plus haut lien) :  
Une colonne "Justification" (fictive donc) permet d'expliquer le choix pertinent de la route.  
| Réseau destination |  Next-Hop  | Coût | Justification | 
|--------------------|------------|-----|-----|
|  10.200.2.0/24 | DC int_R1_LAN |  X  | Route vers le Réseau interne |
|  10.250.0.0/24 | DC int_R1_WAN |  X  | Route vers le VLAN 633 |
|  10.100.9.0/24 | 10.250.0.18 |  20  | Réseau interne "Prof" |
|  10.200.1.0/24 | 10.250.0.102|  20  | Exemple d'un réseau interne d'un autre binôme |
|  192.168.176.0/24 | 10.250.0.254 |  20  | Route vers le VLAN 176 |
|  192.168.140.0/23 | 10.250.0.253 |  20  | Route vers le VLAN 140 |
|  0.0.0.0/0 | 10.250.0.254  |    | Route par défault |
|            | 10.250.0.253  |    | Route par défault |

La table de routage du routeur R2 sera semblable à celle de R1.

### <u> Question 2 </u>
Le protocole VRRP (Virtual Router Redundancy Protocol) assure la redondance de la passerelle par défaut pour des machines d'un réseau interne en utilisant une adresse IP virtuelle associé à un groupe de routeurs. Avec VRRP nous avons la présence d'un routeur maître, assurant le service de routage, ect. En cas de défaillance de ce dernier, VRRP sélectionne le routeur esclave (ou secondaire) enfin d'assurer la continuité du service.

### <u> Question 3 </u>
La configuration VRRP, théoriquement mise en place sur R1 et R2, permet d'assurer la disponibilité aux réseaux externes pour les machines A et B. Les réseaux externes sont disponibles tant que R1 et R2 fonctionnent correctement.
Fonctionnment résumé de VRRP :
1) Élection du routeur maître : Un routeur est désigné comme maître et possède l’adresse IP virtuelle associée au routeur virtuel. Il gère le trafic des machines A et B. Dans notre exemple, disons que c'est R1.
2) Routeurs de secours : Un ou plusieurs routeurs sont désignés comme backup, dans notre exemple, R2. Ces derniers surveillent l’état du maître via des messages VRRP envoyés périodiquement.
3) Défaillance et bascule : Si le routeur maître devient inactif (absence de messages VRRP), le routeur backup, ne voyant plus de messages VRRP, prend le relais en assumant l’adresse IP virtuelle, il communique ce changement aux machienes A et B, garantissant ainsi la continuité du trafic réseau.

Les informations VRRP circulent sur le réseau dans des messages "VRRP Advertissement Messages", envoyé par le maître.

A et B ont une adresse IP virtuelle configurée comme route par défaut, connue par les routeurs R1 et R2. Les messages "VRRP Advertissement Messages", envoyé périodiquement par le maître en multicast contiennent une valeur de priorité qui permet d'identifier quel routeur doit être associé à l'adresse IP virtuelle à l'instant t. Pour indiquer aux machines A et B à quel routeur il faut envoyer les trames pour communiquer avec l'extérieur seul le routeur associé à l'adresse virtuelle répondra aux requêtes ARP. Les machines savent donc qu'elles doivent envoyer leurs paquets vers le routeur maître.

### <u> Question 4 </u>
Le protocole OSPF est un protocol de routage permettant à tous les routeurs de la topologie d'avoir la même "vue" du réseau. Grâce à l'algorithme SPF (Short Path First), utilisé par OSPF, les routeurs remplissent leur table de routage. OSPF est curcial pour garantir la connectivité.

L'utilisation d'un routage statique n'est pas pertinent dans notre cas. Si sur le routeur RPROF1 (auquel nous n'avons pas forcèment accès) nous configurons une route statique vers notre réseau interne, il va tout le temps rediriger les paquets vers le routeur R1 (si c'est le prochain saut indiqué).

Si R1 devient indisponible (mauvaise configuration, panne électrique ou physique,...) alors les réseaux internes perdront leur accessibilité aux réseaux externes.
L'usage d'OSPF va résoudre ce souci en échangeant dynamiquement des informations sur les routes disponibles, cela permettra ainsi à R2 de prendre le relais.

En somme, l'utilisation conjointe d'OSPF et de VRRP offre une solution complète pour assurer la continuité des services réseau.

**************************************************
# 3.3 Mise en place et configuration des machines virtuelles

4 machines virtuelles sont a configurer : 
- Machines A et B, deux Alma Linux 9.2
- Machines R1 et R2, deux routeur Cisco CSR1000v

La configuration de ces machines sera faîte en 4 étapes. Etant seul pour ce mini-projet j'ai réalisé les 4 étapes moi-même.

## 3.3.1 Etape 1 - Création VMs A et R1, configuration IP
### <u> Question 5 </u>
Afin de tester la configuration effectuée à cette étape, le test ci-dessous seront réalisés :
Sur A : 
- Ping <@IP_R1_LAN>
Réponse attendu :
- Positif
Résultat du test après configuration :
- OK

Ce test nous permet de vérifier 2 choses, que A a une configuration IP correcte (adresse de sous-réseau, passerelle par défaut (IP de R1) correcte) et que R1 a également une configuration IP correcte.

## 3.3.2 Etape 2 - Configuration OSPF dans R1
### <u> Question 6 </u>
Pour tester la mise en place du protocole OSPF nous allons, avant la mise en place, observer notre table de routage sur R1.
Cette dernière ne contient pas de route OSPF. Elle ne contient également pas de route vers les VLAN 140 et VLAN 176.

Après la mise en place d'OSPF, tous les réseaux internes des autres binômes ayant configuré OSPF sur leur(s) routeur(s) apparaissent. Les routes pour accèder aux VLAN 140 et VLAN 176. 
Nous avons également testé l'accès à internet des machines du réseau internes :
Sur A :
- Ping 8.8.8.8
Réponse attendu :
- Positif
Résultat du test après configuration :
- OK

## 3.3.3 Etape 3 -Création VMs B et R2, configuration IP et OSPF
Dans cette partie nous effectuons la configuration IP de la machine B et du routeur R2. Cette configuration est semblable à l'étape 1 et 2.
Sur B, la passerelle par défaut a été définie à 10.200.2.253 (@IP de l'interface réseau interne) afin de tester la configuration IP de B et R2.
Sur B :
- Ping 8.8.8.8
Réponse attendu :
- Positif
Résultat du test après configuration :
- OK

## 3.3.4 Etape 4 - Mise en place VRRP
### <u> Question 7 </u>
La mise en place de VRRP a été effectuée dans R1 et R2. Sur les machines internes (A et B), la route par défault à été changé par l'@IP virtuelle : 10.200.2.100

Testons divers scénarios
1) Les 2 routeurs sont UP
- L'état VRRP de R1 est Master
- L'état VRRP de R2 est Backup
Sur A et sur B :
- Ping 8.8.8.8
Réponse attendu :
- Positif et passerelle par défaut -> R1
Résultat du test après configuration :
- OK

2) R1 est DOWN, R2 est UP
- L'état VRRP de R1 est passé à Init
- L'état VRRP de R2 est passé à Master
Sur A :
- Ping 8.8.8.8
Réponse attendu :
- Positif et passerelle par défaut -> R2
Résultat du test après configuration :
- OK
=> R2 a donc bien pris le rôle de Master

3) R1 et R2 sont DOWN
Sur A :
- Ping 8.8.8.8
Réponse attendu :
- Négatif
Résultat du test après configuration :
- OK (ping négatif)

Les tests ont été réalisés en utilisant la commande "tracepath 8.8.8.8". Leur résultat confirme la théorie. Notre configuration VRRP est donc correcte.

=> [Configuration du routeur 813-B10-R1](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_1/Partie%201%20-%20Configuration%20de%20813-B10-R1)
=> [Configuration du routeur 813-B10-R2](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_1/Partie%201%20-%20Configuration%20de%20813-B10-R2)
