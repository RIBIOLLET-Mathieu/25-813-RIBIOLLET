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
Script à la fin de cette étape : [Script Partie3 - 5.1](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/script_Partie3%20-%205-1.md)  

**************************************************
# 5.2 Gestion de la date et enregistrement des résultats dans un fichier
Script à la fin de cette étape, ajout de la date et enregistrement dans un fichier.
[Script Partie3 - 5.2](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/script_Partie3%20-%205-2.md)  

Plusieurs tests ont été menés, voici le contenu du fichier de sorti à la fin de cette étape :
```bash
[root@813-B10-A Partie5_Script]# ./snmp5-2.sh 
1742225060;165753177
1742224732;165746421
1742224820;165748407
1742225060;165753177
```

**************************************************
# 5.3 Lecture de la dernière ligne du fichier, calcul et enregistrement du débit
### <u> Question 19 </u>
Pour tester notre script à ce jour (voir ci-dessous), nous procédons comme suit :  
1) Lancer le script une première fois  
   Lors de cette première exécution, le fichier de sortie contiendra uniquement une seule ligne, et la valeur de la variable "rate" sera égale à 0. Cette valeur est 
   normale, car le script ne peut pas effectuer de comparaison avec un relevé précédent.
   Sortie du test de cette étape : [Script Partie3 - 5.3](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/script_Partie3%20-%205-3.md)  

**************************************************
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

Script à la fin de cette étape : [Script Partie3 - 5.4](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/script_Partie3%20-%205-4.md)  

**************************************************
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
A savoir que crontab est "lié" à l'utilisateur sur lequel on le configure. Si on configurer crontab avec l'utilisateur root, avec le script sera éxécuté avec root.  
Dans le crontab de l'utilisateur "etudiant", notre configuration ne sera pas présente.
Crontab s'éxécute dans son répertoire courant. Afin qu'il puisse trouver et éxécuter correctement le script on spécifié un variable "workdir" au début du script.

Script à la fin de cette étape : [Script Partie3 - 5.5](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/script_Partie3%20-%205-5.md)

**************************************************
# 5.6 Script générique
### <u> Question 22 </u>
Afin de rendre le script plus générique nous avons récupéré les valeurs des arguments dans nos variables. Bien évidemment une vérification de la présence des arguments est faîte en début de script. Hormis cela, rien n'a été modifié.  
-> Afin de se rendre compte du débit moyen, un petit script calculant ce dernier et renseignant la valeur calculée dans un fichier "debit_moyen.txt" a été écrit.
Script final : [Script Partie3 - 5.6_FINALE](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/script_Partie3%20-%205-6-FINAL.md)
