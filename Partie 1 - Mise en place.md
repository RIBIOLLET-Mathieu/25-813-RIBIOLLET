## Partie 1 - Mise en place d'une maquette de réseau local avec haute disponibilité
Objectifs
- Réaliser une étude préparatoire sur la maquette
- Mettre en place les différentes machines (PC / Routeur)
- Réaliser la configuration réseau des machines

## Question 1
Dans la théorie, nous utilisons le protocole OSPF comme protocole de routage. Nous apprenons donc :
- x9 routes vers le sous-réseau des autres binômes.
- x2 route vers les sous-réseaux directement connectés (Le notre et le VLAN633)
- x1 route vers le sous-réseau "Prof".
- x2 routes vers les sous-réseau VLAN140 et VLAN176
- x1 route par défault utilisant les routeurs RPROF1 et RPROF2 comme passerelle par défault.

Donc au final, 15 routes seront présentes dans notre table de routage.

Pour le routeur R1 nous pouvons avoir une table de routage comme présenté ci-dessous (Pour la colonne "coût" nous n'avons pas de coût de référence, nous ne renseignons donc pas de valeur) :
Une colonne "Justification" (fictive donc) permet d'expliquer le choix pertinent de la route.
| Réseau destination |  Next-Hop  | Coût | Justification | 
|-------------------:|------------|-----|-----|
|  10.200.2.0/24 | DC int_R1_LAN |  X  | Route vers le Réseau interne |
|  10.250.0.0/24 | DC int_R1_WAN |  X  | Route vers le VLAN 633 |
|  10.100.9.0/24 | 10.250.0.18 |  X  | Réseau interne "Prof" |
|  10.200.1.0/24 | 10.250.0.102|  X  | Exemple d'un réseau interne d'un autre binôme |
|  192.168.176.0/24 | 10.250.0.254 |  X  | Route vers le VLAN 176 |
|  192.168.140.0/23 | 10.250.0.253 |  X  | Route vers le VLAN 140 |
|  0.0.0.0/0 | 10.250.0.254 |  X  | Route par défault |

La table de routage du routeur R2 sera semblable à celle de R1.

## Question 2
Le protocole VRRP (Virtual Router Redundancy Protocol) assure la redondance de la passerelle par défaut en utilisant une adresse IP virtuelle associé à un groupe de routeurs. Avec VRRP nous avons la présence d'un routeur maître, assurant le service de routage, ect. En cas de défaillance de ce dernier, VRRP sélectionne le routeur esclave (ou secondaire) enfin d'assurer la continuité du service.

## Question 3 

La configuration VRRP, théoriquement mise en place sur R1 et R2, permet d'assurer la disponibilité aux réseaux externes pour les machines A et B. Les réseaux externes sont disponibles tant que R1 et R2 fonctionnent correctement.
Fonctionnment résumé de VRRP :
1) Élection du routeur maître : Un routeur est désigné comme maître et possède l’adresse IP virtuelle associée au routeur virtuel. Il gère le trafic des machines A et B. Dans notre exemple, disons que c'est R1.
2) Routeurs de secours : Un ou plusieurs routeurs sont désignés comme backup, dans notre exemple, R2. Ces derniers surveillent l’état du maître via des messages VRRP envoyés périodiquement.
3) Défaillance et bascule : Si le routeur maître devient inactif (absence de messages VRRP), le routeur backup, ne voyant plus de messages VRRP, prend le relais en assumant l’adresse IP virtuelle, il communique ce changement aux machienes A et B, garantissant ainsi la continuité du trafic réseau.

Les informations VRRP circulent sur le réseau dans des messages "VRRP Advertissement Messages", envoyé par le maître.

A et B ont une adresse IP virtuelle configurée comme route par défaut, connue par les routeurs R1 et R2. Les messages "VRRP Advertissement Messages", envoyé périodiquement par le maître en multicast, contiennent une valeur de priorité qui est indiqué aux machines A et B. Les machines savent donc qu'elles doivent envoyés leurs paquets vers le routeur maître.
