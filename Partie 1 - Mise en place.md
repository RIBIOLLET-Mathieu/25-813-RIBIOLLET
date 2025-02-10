## Partie 1 - Mise en place d'une maquette de réseau local avec haute disponibilité
Objectifs
- Réaliser une étude préparatoire sur la maquette
- Mettre en place les différentes machines (PC / Routeur)
- Réaliser la configuration réseau des machines

## Question 1
Dans notre table de routage, il y aura théoriquement, lorsque les protocoles auront convergés, 15 lignes. Ces 15 lignes comprennent chaque réseau interne des binômes, plus celui du Prof. La route pour accèder au réseau directement connecté "10.250.0.0/24" et les routes pour aller vers les vlans 176 et 140. La route par défault (direction internet) est la 14e route

Table de routage limitée de R1 :
| Réseau destination | Next-Hop | Coût |
|-------------------:|----------|------|
|     1              |          |      |
|     2              |          |      |
|     3              |          |      |
|     4              |          |      |
|     5              |          |      |
|     6              |          |      |
|     7              |          |      |

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
