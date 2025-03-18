# 5. Partie 3 - Script bash de mesure de début en SNMP
Objectifs
- Mesurer le débit d'une interface d'un routeur pendant une longue période
- Automatiser le lancement d'un script avec "cron"
La version finale du script permettra de :
- Récupérer en SNMP la valeur du compteur d'octets en SNMP
- Calculer le débit moyen depuis la mesure précèdente
- Enregistrer les informations calculer dans un fichier
- Les enregistrements seront horodatés


### <u> Question 18 </u>
Il est préférable d'utiliser Cron ou Systemd plutôt que sleep car cron est plus efficace (moins de consommation de ressources comparé à sleep qui garde le script en arrière plan). Cron est également plus précis dans les éxécutions régulières.
Enfin, Cron permet un gestion simple en offrant des outils tels que des logs. Ces avantages justifient notre choix d'utiliser Cron pour lancer automatiquement le script.

**************************************************
# 5.1 Récupération du compteur d'octets
Script à la fin de cette étape :
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


# 5.2 Gestion de la date et enregistrement des résultats dans un fichier
Script à la fin de cette étape, ajout de la date et enregistrement dans un fichier.
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
Plusieurs tests ont été menés, voici le contenu du fichier de sorti à la fin de cette étape :
```bash
[root@813-B10-A Partie5_Script]# ./snmp5-2.sh 
1742225060;165753177
1742224732;165746421
1742224820;165748407
1742225060;165753177
```


# 5.3 Lecture de la dernière ligne du fichier, calcul et enregistrement du débit
### <u> Question 19 </u>
Pour tester notre script à ce jour (voir ci-dessous), nous procédons comme suit :  
1) Lancer le script une première fois  
   Lors de cette première exécution, le fichier de sortie contiendra uniquement une seule ligne, et la valeur de la variable "rate" sera égale à 0. Cette valeur est 
   normale, car le script ne peut pas effectuer de comparaison avec un relevé précédent.
   Sortie du test de cette étape :
   ```bash
   [root@813-B10-A Partie5_Script]# ./snmp5-3.sh 
   1742226309;165798625;0
   ```

2) Lancer le script une deuxième fois  
   Après cette deuxième exécution, le fichier de sortie affichera toujours la première ligne, inchangée. Une deuxième ligne apparaîtra, et cette fois-ci, une valeur sera  attribuée à la variable "rate". Ce débit correspond à la différence entre les valeurs d'octets et le temps écoulé entre la première et la deuxième mesure.  
   ```bash
   [root@813-B10-A Partie5_Script]# ./snmp5-3.sh 
   1742226309;165798625;0
   1742226363;165801577;437
   ```

Script à la fin de cette étape :
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

# 5.4 Gestion du fichier vide et gestion du rebouclage du compteur d'octets
La gestion de la première exécution ayant déjà été prise en compte précèdemment, aucunes modifications n'ont été effectuées concernant cela.  
### <u> Question 20 </u>
Un rebouclage du compteur donnerait un calcul de débit avec un résultat négatif, ce qui fausserait nos calculs de débits moyens par la suite.
Si la nouvelle valeur de compteur relevée est inférieur à l'ancienne valeur, cela signifie qu'un rebouclage a eu lieu. Dans ce cas, la nouvelle valeur du compteur doit être ajustée en ajoutant la capacité maximale du compteur à la différence calculée. Ainsi le calcul ne sera pas faussé.

Dans le script on modifie le calcul de la différence d'octets.
```bash
# Calcul de la différence d'octets en tenant compte du rebouclage
if [ "$value" -lt "$last_value" ]; then
    # Rebouclage détecté, ajustement en ajoutant la valeur maximale du compteur (32 bits)
    byte_diff=$(( (4294967295 - last_value) + value ))
else
    byte_diff=$((value - last_value))
fi
```

Script à la fin de cette étape :
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

# 5.5 Utilisation du cron pour que le script s'exécute toutes les minutes
### <u> Question 21 </u>
Afin que le script s'éxécute toutes les minutes, nous allons mettre en place cron, un ordonnanceur très simple intégré à linux.  
Afin que le script s'éxécute toutes les minutes il nous faut étudier le gestionnaire de crontab via la commande ```crontab -e```.  
On y ajoute la ligne suivante : ```* * * * * /home/etudiant/Partie5_Script/snmp5-5.sh```  
Explication :  
```bash
* * * * * /home/etudiant/Partie5_Script/snmp5-5.sh
│ │ │ │ │
│ │ │ │ └── Jour de la semaine (0 - 7)  →  * = Tous les jours
│ │ │ └──── Mois (1 - 12)               →  * = Tous les mois
│ │ └────── Jour du mois (1 - 31)       →  * = Tous les jours
│ └──────── Heure (0 - 23)              →  * = Toutes les heures
└────────── Minute (0 - 59)             →  * = Toutes les minutes
```

Script à la fin de cette étape :
```

```

# 5.6 Script générique
### <u> Question 22 </u>

Script final :
```
```
