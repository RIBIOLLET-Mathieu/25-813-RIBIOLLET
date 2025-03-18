```bash
#!/bin/bash

# --- Définition des valeurs en dures ---
oid="1.3.6.1.2.1.2.2.1.16.3"   # OID pour le compteur d'octets sortants de l'interface 3
agent_ip="10.200.2.254"        # Adresse IP de l'agent SNMP
community="123test123"         # Communauté SNMP (ici, "123test123" pour SNMP v2c)

# --- Récupération de la valeur du compteur ---
value=$(snmpget -v2c -c $community -Oqv $agent_ip $oid)

# --- Affichage ---
echo "${value}"
```
=> [Retour Partie 3](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/Partie%203%20-%20Script%20bash%20de%20mesure%20de%20debit.md)  
=> [Retour Sommaire](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET)
