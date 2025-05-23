
# 25-813-RIBIOLLET
## Dépôt Github privé mini-projet de supervision - RIBIOLLET Mathieu

Paramètres réseaux attribués :
| **Élément**           | **Adresse IP**     | **Description**          |
|-----------------------|--------------------|--------------------------|
| Réseau interne        | 10.200.2.0/24      | Réseau interne           |
| Switch interne        | VLAN612            | VLAN du switch interne   |
| R1 interne            | 10.200.2.254       | Interface interne de R1  |
| R1 externe            | 10.250.0.103       | Interface externe de R1  |
| R1 loopback           | 10.20.2.1          | Interface loopback de R1 |
| R2 interne            | 10.200.2.253       | Interface interne de R2  |
| R2 externe            | 10.250.0.104       | Interface externe de R2  |
| R2 loopback           | 10.20.2.2          | Interface loopback de R2 |


Objectifs des TPs et du mini-projet
- Mettre en place une maquette réseau local avec haute disponibilité
- Mettre en place une supervision et une métrologie à l'aide du protocol SNMP
- Développer un script Bash de mesure de débit
- Travailler sur un mini-projet sur des outils de supervision (Prometheus / Grafana / Netflow)

Table des matières :
- [Partie 1 - Mise en place](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_1/Partie%201%20-%20Mise%20en%20place.md)
  - [Partie 1 - Configuration de 813-B10-R1](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_1/Partie%201%20-%20Configuration%20de%20813-B10-R1)
  - [Partie 1 - Configuration de 813-B10-R2](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_1/Partie%201%20-%20Configuration%20de%20813-B10-R2)
- [Partie 2 - Supervision et métrologie avec SNMP](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_2/Partie%202%20-%20Supervision%20et%20m%C3%A9trologie.md)
- [Partie 3 - Script bash de mesure de débit en SNMP](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/Partie%203%20-%20Script%20bash%20de%20mesure%20de%20debit.md)
  - [Partie 3 - Script 5.1](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/script_Partie3%20-%205-1.md)
  - [Partie 3 - Script 5.2](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/script_Partie3%20-%205-2.md)
  - [Partie 3 - Script 5.3](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/script_Partie3%20-%205-3.md)
  - [Partie 3 - Script 5.4](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/script_Partie3%20-%205-4.md)
  - [Partie 3 - Script 5.5](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/script_Partie3%20-%205-5.md)
  - [Partie 3 - Scrip 5.6_FINAL](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/script_Partie3%20-%205-6-FINAL.md)
- [Partie 4](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_4/Partie%204%20-%20Projet%20Prometheus_Grafana_Netflow_Logs.md)
