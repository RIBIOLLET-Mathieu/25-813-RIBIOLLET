```bash
#!/bin/bash

# Vérification des arguments
if [ "$#" -ne 4 ]; then
    echo "Usage: $0 <oid> <agent_ip> <community> <filename>"
    exit 1
fi

# --- Définition des valeurs en dur ---
workdir="/home/etudiant/Partie5_Script"
oid="$1"                 # OID pour le compteur d'octets sortants de l'interface 3
agent_ip="$2"            # Adresse IP de l'agent SNMP
community="$3"           # Communauté SNMP (ici, "123test123" pour SNMP v2c)
filename="${workdir}/$4" # Fichier où enregistrer les résultats

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

    # Calcul de la différence d'octets en tenant compte du rebouclage
    if [ "$value" -lt "$last_value" ]; then
        # Rebouclage détecté, ajustement en ajoutant la valeur maximale du compteur (32 bits)
        byte_diff=$(( (4294967295 - last_value) + value ))
    else
        byte_diff=$((value - last_value))
    fi

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
