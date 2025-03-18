```bash
#!/bin/bash

# --- Définition des valeurs en dur ---
oid="1.3.6.1.2.1.2.2.1.16.3"   # OID pour le compteur d'octets sortants de l'interface 3
agent_ip="10.200.2.254"        # Adresse IP de l'agent SNMP
community="123test123"         # Communauté SNMP (ici, "123test123" pour SNMP v2c)
filename="throughput_int1.txt" # Fichier où enregistrer les résultats

# --- Récupération de la valeur du compteur ---
value=$(snmpget -v2c -c $community -Oqv $agent_ip $oid)

# --- Récupérer la date en nombre de secondes depuis le 01/01/1970 ---
timestamp=$(date +%s)

# --- Enregistrement dans le fichier ---
echo "${timestamp};${value}" | tee -a "${filename}"

# --- Affichage ---
cat "$filename"
```
