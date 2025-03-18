```bash
#!/bin/bash

# --- Définition des valeurs en dur ---
oid="1.3.6.1.2.1.2.2.1.16.3"   # OID pour le compteur d'octets sortants de l'interface 3
agent_ip="10.200.2.254"        # Adresse IP de l'agent SNMP
community="123test123"         # Communauté SNMP (ici, "123test123" pour SNMP v2c)
filename="throughput_int3.txt" # Fichier où enregistrer les résultats

# --- Récupération de la valeur du compteur ---
value=$(snmpget -v2c -c $community -Oqv $agent_ip $oid)

# --- Récupérer la date en nombre de secondes depuis le 01/01/1970 ---
timestamp=$(date +%s)

# --- Vérification de l'existence du fichier, création si besoin ---
if [ ! -e "$filename" ]; then
    touch $filename
fi

# --- Lecture de la dernière ligne du fichier ---
last_line=$(tail -n 1 "$filename")

#Si le fichier est vide, on ne peut pas procèder au calcul
if [ -n "$last_line" ]; then
    # Récupération du timestamp et de la dernière valeur de compteur récupérée
    last_timestamp=$(echo "$last_line" | cut -d ";" -f 1)
    last_value=$(echo "$last_line" | cut -d ";" -f 2)

    # Calcul de la différence de temps (en secondes)
    time_diff=$((timestamp - last_timestamp))

    # Calcul de la différence d'octets
    byte_diff=$((value - last_value))

    # Calcul du débit en bits par seconde
    if [ $time_diff -gt 0 ]; then
        rate=$(( (byte_diff * 8) / time_diff ))
    else
        rate=0
    fi
else
    rate=0
fi

# --- Enregistrement des résultats dans le fichier ---
echo "${timestamp};${value};${rate}" | tee -a "$filename"

# --- Affichage ---
cat "$filename"
```
=> [Retour Partie 3](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_3/Partie%203%20-%20Script%20bash%20de%20mesure%20de%20debit.md)  
=> [Retour Sommaire](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET)

