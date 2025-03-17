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
```
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
```
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
```
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

2) Lancer le script une deuxième fois  
   Après cette deuxième exécution, le fichier de sortie affichera toujours la première ligne, inchangée. Une deuxième ligne apparaîtra, et cette fois-ci, une valeur sera  
   attribuée à la variable "rate". Ce débit correspond à la différence entre les valeurs d'octets et le temps écoulé entre la première et la deuxième mesure.

Script à la fin de cette étape :
```
```

# 5.4 Gestion du fichier vide et gestion du rebouclage du compteur d'octets
### <u> Question 20 </u>

Script à la fin de cette étape :
```
```

# 5.5 Utilisation du cron pour que le script s'exécute toutes les minutes
### <u> Question 21 </u>

Script à la fin de cette étape :
```
```

# 5.6 Script générique
### <u> Question 22 </u>

Script final :
```
```
