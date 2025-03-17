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
Script à la fin de cette étape :
```
```

# 5.3 Lecture de la dernière ligne du fichier, calcul et enregistrement du débit
### <u> Question 19 </u>

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
