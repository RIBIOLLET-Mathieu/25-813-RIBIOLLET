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

Pour le routeur R1 nous pouvons avoir une table de routage comme présenté ci-dessous (Pour la colonne "coût" nous n'avons pas de coût de référence) :
Une colonne "Justification" (fictive donc) permet d'expliquer le choix pertinent de la route.
| Réseau destination |  Next-Hop  | Coût | Justification | 
|-------------------:|------------|-----|-----|
|  10.200.2.0/24     | DC int_R1_LAN |  X  | Réseau interne |
|  10.250.0.0/24     | DC int_R1_WAN |  X  | VLAN 633 |
|  10.100.9.0/24     | 10.250.0.18 |  X  | Réseau interne "Prof" |
|  10.200.1.0/24     | 10.250.0.102|  X  | Exemple d'un réseau interne d'un autre binôme |
|  192.168.176.0/24     | 10.250.0.254 |  X  | VLAN 176 |
|  192.168.140.0/23     | 10.250.0.253 |  X  | VLAN 140 |
|  0.0.0.0/0     | 10.250.0.254 |  X  | Route par défault |

Table de routage limitée de R2 :
| Réseau destination | Next-Hop | Coût |
|-------------------:|----------|------|
|     1              |          |      |
|     2              |          |      |
|     3              |          |      |
|     4              |          |      |
|     5              |          |      |
|     6              |          |      |
|     7              |          |      |
