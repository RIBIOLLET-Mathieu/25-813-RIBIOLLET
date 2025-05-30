# 6. Partie 4 - Projet Prometheus / Grafana / Netflow / Logs

Objectifs
- Déployer un outils de supervision : Prometheus
- Mettre en oeuvre un outil de création de tableau de bord (en autre) : Grafana
- Déployer des conteneurs Docker

# Synthèse Prometheus et ses outils

## Introduction
Prometheus est un outil de supervision open-source conçu pour collecter, stocker et analyser des métriques sous forme de séries temporelles. Il est souvent utilisé avec Grafana pour la visualisation des données.

## Fonctionnement de Prometheus
Prometheus fonctionne selon un modèle **pull**, où il récupère régulièrement des métriques depuis des **exporters** ou directement depuis des applications qui exposent des données via HTTP.

# Supervision avec Prometheus et Grafana

## Introduction
Prometheus est une solution open-source de supervision des systèmes et des applications. Conçu pour collecter, stocker et analyser des métriques sous forme de séries temporelles, Prometheus est particulièrement adapté pour surveiller des infrastructures modernes et dynamiques. Grâce à son architecture flexible et sa capacité à gérer de grandes quantités de données, il s'intègre parfaitement dans des environnements distribués, tels que les clusters Kubernetes ou les environnements Cloud.  

Prometheus fonctionne sur un modèle **pull**, où il interroge directement les cibles (serveurs, applications, services) à intervalles réguliers pour collecter des métriques. Ce modèle est adapté aux environnements modernes où les cibles peuvent être éphémères ou changer fréquemment, comme dans le cas de Kubernetes.  

Lorsqu'il est couplé à **Grafana**, Prometheus offre une solution complète de supervision et de visualisation des données en temps réel. Grafana permet de créer des tableaux de bord dynamiques et interactifs pour explorer et analyser les métriques collectées par Prometheus. Cette combinaison permet aux équipes techniques de surveiller de manière proactive leurs infrastructures, de visualiser les performances des systèmes et de détecter rapidement toute anomalie.  

L’un des atouts majeurs de Prometheus est sa scalabilité et sa capacité à s’adapter à des environnements à grande échelle, rendant la supervision des applications Cloud natives et des systèmes distribués plus facile et plus fiable.  

## Avantages de Prometheus
Prometheus présente de nombreux avantages par rapport aux solutions de supervision traditionnelles telles que Nagios ou Zabbix. Voici une comparaison détaillée qui montre en quoi Prometheus se distingue :  

| Critère               | Prometheus                                      | Autres solutions (ex: Nagios, Zabbix) |
|-----------------------|------------------------------------------------|--------------------------------------|
| **Modèle de collecte** | Utilise un modèle **pull**, où Prometheus interroge régulièrement les cibles pour collecter les métriques. Ce modèle est flexible et s’adapte bien aux environnements dynamiques. | Beaucoup d'autres outils utilisent un modèle **push**, où les cibles envoient les données, ce qui peut rendre la gestion plus complexe. |
| **Stockage**          | Prometheus utilise une base de données optimisée pour les séries temporelles, permettant un stockage efficace et rapide des métriques sur de longues périodes. | Les solutions traditionnelles utilisent souvent des bases de données relationnelles ou des fichiers, qui ne sont pas conçus pour gérer de grandes quantités de données temporelles. |
| **Scalabilité**       | Il est conçu pour être hautement scalable et peut facilement s’adapter à des infrastructures distribuées et dynamiques telles que Kubernetes ou des environnements Cloud. | Les autres solutions sont souvent moins flexibles dans des environnements changeants ou très dynamiques, nécessitant plus de configuration pour s’adapter. |
| **Configuration**     | La configuration de Prometheus se fait principalement via des fichiers YAML simples, ce qui facilite l'automatisation et l’intégration dans des systèmes d’infrastructure as code (IaC). | La configuration dans les outils traditionnels peut être plus complexe et moins flexible, avec des interfaces souvent plus lourdes. |
| **Visualisation**     | Prometheus s’intègre nativement à **Grafana**, un outil de visualisation puissant qui permet de créer des tableaux de bord dynamiques pour explorer les métriques collectées. | D'autres outils offrent des interfaces graphiques, mais elles sont souvent moins puissantes ou moins adaptées à des visualisations personnalisées et interactives. |
| **Gestion des alertes** | Intégration avec **Alertmanager**, un composant dédié pour la gestion des alertes, qui permet une flexibilité maximale dans la définition et l’envoi des notifications. | Les systèmes d'alertes dans d’autres outils sont souvent intégrés de manière moins flexible et offrent moins de personnalisation. |
| **Communauté**       | Prometheus bénéficie d’une large communauté active qui contribue régulièrement à son développement et propose une multitude de ressources, de plugins et de guides. | Certaines solutions peuvent avoir une communauté moins active ou des ressources moins accessibles pour les utilisateurs. |

### Composants principaux de Prometheus
Prometheus est composé de plusieurs éléments qui travaillent ensemble pour fournir une solution complète de supervision et de gestion des métriques. Les principaux composants sont les suivants :  

- **Prometheus Server** : C'est le cœur de la solution. Il collecte et stocke les métriques dans une base de données optimisée pour les séries temporelles. Ce serveur interroge périodiquement les cibles configurées pour récupérer les données.  
- **Exporters** : Ce sont des applications ou des services qui exposent des métriques spécifiques à Prometheus. Par exemple, un exporter peut récupérer des informations sur l’utilisation des ressources système d’un serveur et les rendre disponibles pour Prometheus.  
- **Alertmanager** : Ce composant gère les alertes générées par Prometheus. Lorsqu'une condition d’alerte est remplie (par exemple, une utilisation CPU trop élevée), Alertmanager peut envoyer des notifications via différents canaux comme email, Slack ou PagerDuty.  
- **Grafana** : Outil de visualisation qui permet de créer des tableaux de bord interactifs à partir des métriques collectées par Prometheus. Il s'intègre parfaitement avec Prometheus pour afficher les données sous forme de graphiques, de jauges et d'autres visualisations.  

## Collecte des métriques
Prometheus collecte des métriques en interrogeant régulièrement les cibles qui exposent des données via des points de terminaison HTTP. Ces cibles peuvent être des serveurs, des équipements réseau, ou des applications. Les métriques sont fournies dans un format texte spécifique que Prometheus peut analyser.  
Les cibles doivent être configurées pour exposer les métriques dans un format compatible avec Prometheus, souvent via des **exporters** qui rendent les données accessibles.

### Exemples d’exporters courants
Voici quelques exemples d'exporters couramment utilisés pour récupérer des métriques :

- **Node Exporter** : Il collecte des métriques système sur les machines Linux ou Windows, telles que l’utilisation du processeur, de la mémoire, du stockage, etc. Il fournit des informations détaillées sur les ressources matérielles du serveur.   
- **Blackbox Exporter** : Il permet de tester la disponibilité des services en envoyant des requêtes HTTP, ICMP (ping) ou TCP. Cela permet de vérifier l'état des services externes comme les serveurs web ou les bases de données.   
- **cAdvisor** : Cet exporter collecte des métriques sur les conteneurs Docker ou Kubernetes, notamment l'utilisation des ressources comme le CPU, la mémoire et le réseau.  
- **SNMP Exporter** : Cet exporter collecte des métriques des équipements réseau (comme des routeurs ou des switches) via le protocole SNMP, essentiel pour surveiller l’infrastructure réseau.

## Configuration de Prometheus
La configuration de Prometheus repose sur un fichier centralisé, généralement nommé `prometheus.yml`. Ce fichier permet de définir les cibles à interroger, la fréquence des collectes, ainsi que d'autres paramètres comme les règles d’alerte et l'intégration avec des outils comme Grafana ou Alertmanager.

Voici un exemple de configuration minimale de Prometheus dans le fichier `prometheus.yml` :

```yml
global:
  scrape_interval: 15s  # Fréquence de collecte des métriques

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```
Dans cet exemple :  
global : Cette section définit les paramètres globaux. Ici, nous indiquons que les métriques doivent être collectées toutes les 15 secondes.  
scrape_configs : Cette section définit les cibles spécifiques à interroger. Ici, nous configurons Prometheus pour interroger un serveur Node Exporter en utilisant l'adresse localhost:9100.

La configuration de Prometheus est flexible et permet de définir plusieurs jobs de collecte, de personnaliser les intervalles de scrapping et d’ajouter des règles d’alerte selon les besoins.

## Conclusion
En résumé, Prometheus, combiné avec Grafana et Alertmanager, constitue une solution complète et puissante pour la surveillance proactive et réactive des infrastructures et des applications. Sa flexibilité, sa scalabilité et son écosystème en constante évolution en font l'un des outils les plus populaires dans la supervision des systèmes modernes.

# Mise en place

Nous avons déjà déployé Prometheus et Grafana à l’aide d’une solution Docker, en utilisant docker-compose. Tous les détails concernant ce déploiement sont disponibles ici : [Déploiement Prometheus/Grafana](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_4/D%C3%A9ploiement%20de%20la%20solution.md).  


## SNMP Exporter - Débits des interfaces des routeurs
Prometheus ne prend pas en charge le protocole SNMP nativement. Pour contourner cela, on utilise un composant externe appelé ```snmp-exporter```. Ce service fait office de passerelle entre SNMP et Prometheus : il interroge les équipements via SNMP, puis expose les données au format HTTP, que Prometheus peut ensuite scraper.  

```yml
---
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "80:9090"
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "443:3000"
    volumes:
      - grafana_data:/var/lib/grafana
  snmp-exporter:
    image: prom/snmp-exporter
    container_name: snmp-exporter
    ports:
      - "9116:9116"
    volumes:
      - ./snmp/snmp.yml:/etc/snmp_exporter/snmp.yml
volumes:
  prometheus_data:
  grafana_data:
```
Dans notre cas, on s’intéresse principalement aux débits réseau transitant par les interfaces de nos routeurs. Inutile de récupérer tous les OID disponibles : on se concentre sur le module if_mib, qui regroupe les informations réseau essentielles (interfaces, trafic entrant/sortant, etc.).  
Le contenu complet du fichier snmp.yml est disponible [ici](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_4/snmp_yml.md).  

Nous n’avons pas rédigé ce fichier manuellement : nous avons simplement récupéré la configuration du module if_mib depuis le dépôt GitHub officiel de snmp-exporter. Cela évite les erreurs et nous permet de gagner du temps en utilisant une configuration éprouvée.  

Ensuite, nous modifions le fichier prometheus.yml pour que Prometheus sache interroger le snmp-exporter. Pour cela, on ajoute un nouveau job SNMP dans la section scrape_configs. Ce job contient les informations nécessaires pour interroger les équipements réseau via SNMP (adresses IP, version SNMP, communauté, etc.).  
```yml
global:
  scrape_interval: 5s  # Fréquence de collecte des métriques

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]
  - job_name: 'debit'
    scrape_interval: 5s
    static_configs:
      - targets: ['10.200.2.254', '10.200.2.253'] # IP des routeurs
        labels:
          group: 'router'
    metrics_path: /snmp
    params:
      module: [if_mib]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: snmp-exporter:9116
```

Une fois tous les fichiers mis à jour, nous redémarrons les services avec "docker-compose". On relance également Grafana pour qu’il prenne en compte les nouvelles données. 

Plusieurs étapes ont été suivies pour récupérer les informations de débits de nos routeurs :
### Étape 1 : Configuration de la source de données
La première étape consiste à **ajouter Prometheus comme source de données** dans Grafana. Cela nous permettra de visualiser et d'analyser les métriques récupérées par Prometheus directement depuis Grafana. Pour ce faire, voici les étapes à suivre :

1. Accédez à `Configuration > Data Sources`.
2. Cliquez sur `Add data source`, puis sélectionnez `Prometheus`.
3. Entrez l'URL de votre serveur Prometheus (dans notre cas : `http://prometheus:9090`) et sauvegardez.

### Étape 2 : Création du tableau de bord
Une fois la source de données configurée, nous allons créer un tableau de bord (dashboard) pour visualiser les données collectées. Nous avons explorés les options des Dashboard Grafana et avons mise en place un affichage nous satisfaisant.

### 2.1 Créer un nouveau tableau de bord
- Clique sur `+ > Dashboard` dans le menu latéral.
- Clique sur `Add new panel` pour ajouter un nouveau panel. Chaque panel représente une source d'information.  

### 2.2 Configurer un panel pour afficher le débit entrant/sortant de R1

- Dans la section `Query`, sélectionnez `Prometheus` comme source de données.
- On a entré la requête suivante pour récupérer les octets entrants : ``` irate(ifHCOutOctets{ifName=~"Gi2|Gi3", instance="10.200.2.254"}[1m]) * 8 ```
  (Note : Le * 8 à la fin permet d'avoir le débit en Mbits/s et non en Moctets/s)

Cette requête permet de récupérer le volume de données entrant sur l'interface réseau `GigabitEthernet2` `GigabitEthernet3` et  de l'instance `10.200.2.254` (R1). Elle filtre les données de Prometheus pour afficher le trafic entrant sur cette interface, offrant ainsi une vue détaillée de la performance du réseau pour cet équipement.

### 2.3 Ajouter un panel pour R2
- On répète les mêmes étapes pour ajouter un panel pour le routeur R2.

### 2.4 Personnalisation des panels
- Chaque panel est nommé pour améliorer la lisibilité du tableau de bord.
- On personnalise le type de visualisation de chaque panel (graphique, jauge, etc.) pour rendre les données encore plus claires et parlantes.

### 2.5 Sauvegarder le tableau de bord
- Enfin, on clique sur `Save` pour enregistrer les modifications et finaliser la création du tableau de bord.


### Procédure de tests 
Afin de tester si notre service snmp-exporter collecte correctement les informations et que notre solution Prometheus+Grafana fonctionne nous allons générer du traffic depuis le réseau interne vers l'extérieur. Les graphiques de débits ("Dashboard") verront alors des données apparaître.  
Pour simuler l'utilisation du réseau par une production cliente nous avons rédigé le script ci-dessous. Il généré 10 iperf avec différents débits définit en début de script.  
```bash
#!/bin/bash

# --- TESTS PARTIE PROJET ---

# Adresse IP du serveur iperf3
SERVER="192.168.141.230"

# Durée totale du test (en secondes)
TOTAL_DURATION=300 # 5 minutes

# Débits à appliquer à chaque intervalle
declare -a BANDWIDTHS=("5M" "15M" "2M" "20M" "1M" "10M" "13M" "30M" "25M" "50M")
# Durées des intervalles (en secondes)
declare -a INTERVALS=(30 30 30 30 30 30 30 30 30 30)  # Chaque intervalle dure 30 secondes

echo "Simulation d'un trafic client avec débit variable..."

# Variable pour la gestion du temps écoulé
temps_ecoule=0

# Effectuer les tests
for i in $(seq 0 $((${#BANDWIDTHS[@]} - 1))); do
    # Récupérer le débit et la durée pour cet intervalle
    BANDWIDTH=${BANDWIDTHS[$i]}
    INTERVAL=${INTERVALS[$i]}

    # Afficher le test en cours
    echo "Test $(($i + 1))/10: De $temps_ecoule s à $((temps_ecoule + INTERVAL)) s avec un débit de $BANDWIDTH"

    # Exécuter iperf3 pour ce débit et cette durée
    iperf3 -c "$SERVER" -b "$BANDWIDTH" -t "$INTERVAL" &

    # Attendre la fin de l'intervalle
    sleep $INTERVAL

    # Mettre à jour le temps écoulé
    elapsed_time=$((temps_ecoule + INTERVAL))
done

echo "Simulation terminée."
```
Les informations de débits des interfaces s'affichent alors sur nos Dashboard Grafana, validant ainsi le fonctionnement de notre solution jusqu'à présent. Par exemple, voici le débit entrant et sortant observé sur l'interface GigabitEthernet2 du routeur R1. Ces débits font suite à l'éxécution du script ci-dessus.  
Par exemple, on voit bien que le premier pic correspond à un débit de 5Mbits/s, ce qui est spécifié dans le script.  
![VIII-Débits des interfaces de R1](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Test%20de%CC%81bits%20validation%20VIII.png)

## Dashboard - Interfaces ayant le plus de débit sur la dernière heure.
L’objectif de cette fonctionnalité est d’identifier rapidement les interfaces réseau les plus sollicitées en termes de trafic sur une période glissante d’une heure. Cela permet de surveiller les flux importants, d’anticiper les risques de saturation, et de détecter d’éventuelles anomalies ou pics d’utilisation inhabituels.  

Une section dédiée a été ajoutée au dashboard afin d’afficher, sous forme de graphique, la liste des interfaces ayant généré le plus de débit (entrant et sortant) au cours de la dernière heure. Les données sont récupérées en temps réel (toutes les 5s) via des requêtes SNMP ou une API de monitoring, puis triées selon le volume de trafic mesuré.  

Afin d'afficher ces informations nous avons utiliser la commande suivant :  
```topk(4,(rate(ifHCInOctets{instance="10.200.2.254"}[1h]) + rate(ifHCOutOctets{instance="10.200.2.254"}[1h]))) * 8 ```  
Elle permet d'afficher les 4 interfaces ayant le plus de traffic (entrant et sortant) durant la dernière heure. La multiplication par 8 du résultat permet un affichage en bit/s.  
Note : Il aurait été pertinent d'afficher séparément le top du traffic entrant et sortant mais cela ne correspondait pas au cahier des charges.  

Ainsi nous observons le graphique suivant :  
![Classement traffic R1](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Classement%20traffic%20R1.png)

On note que l'interface GigabitEthernet2 est l'interface la plus utilisée. Cette utilisation est logique et correspond à la réalité puisque c'est l'interface liée au réseau interne et donc les flux clients passent par cette dernière.  


## Dashboard - Etat VRRP
Cette fonctionnalité a pour objectif d’offrir une supervision en temps réel du protocole VRRP entre les routeurs. Elle permet de s’assurer du bon fonctionnement du mécanisme de haute disponibilité, en vérifiant la répartition correcte des rôles (Master / Backup) et en détectant rapidement tout basculement ou anomalie dans le fonctionnement du protocole.

Pour afficher ces informations il nous faut mettre à jour notre fichier snmp.yml afin d'y ajouter les informations du tableau "vrrpOper".  
Le fichier snmp_yml_vrrp_update.yml est [ici](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_4/snmp_yml_vrrp_update.md)  

Après avoir modifié le fichier snmp.yml pour intégrer les informations VRRP, les conteneurs ont été redémarrés à l’aide de la commande docker afin de prendre en compte la nouvelle configuration.  
Un nouveau panel a ensuite été créé dans Grafana, basé sur la métrique vrrpOperState.  
Après plusieurs ajustements graphiques (suppression des colonnes non souhaitées), les données essentielles suivantes ont été sélectionnées pour l’affichage :
- Le nom de l’équipement concerné,
- Son état VRRP actuel : Master ou Backup.

Ainsi l'affichage est le suivant :  
![Etat VRRP R1 et R2 UP](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Etat%20VRRP%20-%20R1%20et%20R2%20UP.png)

Afin de vérifier la bonne remontée et la mise à jour en temps réel des informations VRRP, un scénario de basculement a été simulé.
L’interface GigabitEthernet2 du routeur R1 (initialement en rôle Master) a été volontairement désactivée. De ce fait, R1 devient inaccessible, et ses informations cessent d’être remontées via SNMP.

Conformément au fonctionnement du protocole VRRP, le routeur R2 détecte l'absence du Master et prend automatiquement le relais en adoptant le rôle de Master. Ce comportement attendu est bien observé sur le dashboard.  
![Etat VRRP R1 DOWN et R2 UP](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Etat%20VRRP%20-%20R1%20DOWN%20R2%20UP.png)

## Dashboard - sysUptime des routeurs

Le fichier snmp.yml, déjà déployé précédemment, inclut la collecte des informations relatives à l'uptime des routeurs. Il nous a donc suffi d'importer un dashboard existant pour visualiser l'uptime des équipements. Ce dashboard a été trouvé sur GitHub, suite à une recherche en ligne pour "Dashboard uptime GitHub".  
Ce tableau de bord nous permet de surveiller l'uptime de nos routeurs, qui indique la durée pendant laquelle un équipement a été opérationnel sans interruption.L'uptime des équipements est affiché via le graphique ci-dessous :  
![Uptime R1 et R2](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Uptime%20R1%20et%20R2.png)

# Docker web
Dans le cadre de cette partie, l'objectif était de mettre en place un serveur web dans un conteneur Docker sur la machine A, capable d’héberger et de servir trois pages web différentes, chacune accessible via une URL propre. L'utilisation de Docker permet de simplifier le déploiement, de mieux gérer les dépendances, et de garantir une bonne portabilité de l'ensemble.  

Objectifs principaux :
- Créer trois pages web distinctes (page1.html, page2.html, page3.html), chacune accessible via une URL différente.
- Utiliser Docker (et un fichier docker-compose.yml) pour déployer et gérer le serveur web.
- S’assurer que le serveur est bien accessible depuis http://localhost:800.
- Mettre en place une supervision des pages web via Prometheus et Grafana, hébergés sur une autre machine (machine B).

Pour cette mise en place, un serveur Nginx a été choisi pour héberger les pages web sur la machine A. En parallèle, un exporter Prometheus a été installé sur la même machine pour permettre la remontée des métriques. Cet exporter est identique à celui déjà utilisé sur la machine B.  

La configuration a été faite à l’aide d’un fichier docker-compose.yml, qui permet de lancer à la fois le serveur web et l’exporter Prometheus de façon simple et automatisée.  
Voici le fichier docker-compose.yml utilisé sur la machine A :
```yml
services:
  web:
    image: nginx:alpine
    ports:
      - "800:80"
    volumes:
      - /home/etudiant/web-content:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - prometheus_exporter

  prometheus_exporter:
    image: nginx/nginx-prometheus-exporter:latest
    ports:
      - "9113:9113"
    environment:
      - NGINX_STATUS_URL=http://web:80/stub_status
    command:
      - "--nginx.scrape-uri=http://web:80/stub_status"
    restart: unless-stopped 
```

Afin de mettre en place la supervision des pages web hébergées sur la machine A, il a été nécessaire de compléter la configuration Docker avec l’ajout du service Blackbox Exporter, un outil utilisé par Prometheus pour tester la disponibilité d’un service externe (comme une page web).  

Pour cela, le fichier docker-compose.yml a été mis à jour en ajoutant une nouvelle section dédiée à blackbox-exporter. Ce conteneur expose un port (9115) qui sera utilisé par Prometheus pour interroger les pages web et vérifier qu'elles répondent correctement.   
```yml
  blackbox-exporter:
    image: prom/blackbox-exporter
    ports:
      - "9115:9115"
    restart: unless-stopped
```
Une fois ce conteneur en place, il a fallu adapter le fichier prometheus.yml afin d’ajouter deux nouveaux "jobs" de supervision :  
- Un premier pour superviser Nginx directement (sur le port 9113).
- Un second pour superviser les trois pages web hébergées (page1.html, page2.html et page3.html) en passant par Blackbox Exporter.
```yml
  - job_name: 'nginx'
    static_configs:
      - targets: ['10.200.2.10:9113']
  - job_name: 'check-pages'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Vérifie que la page répond avec un code HTTP 2xx
    static_configs:
      - targets:
        - http://10.200.2.10:800/page1.html
        - http://10.200.2.10:800/page2.html
        - http://10.200.2.10:800/page3.html
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```
Cette configuration permet à Prometheus de sonder régulièrement les pages web via des requêtes HTTP, de récupérer leur statut (code de réponse, temps de réponse, etc.), et de transmettre ces données à Grafana. Côté Grafana, deux dashboards ont été créés :  
- Un pour la supervision générale de la machine A (CPU, RAM, charge, etc.).
- Un autre pour la supervision du serveur web et des pages, avec notamment l'état de chaque page, les codes HTTP renvoyés, et le temps de réponse.

Ces éléments permettent un suivi en temps réel de la disponibilité et du bon fonctionnement du service web.  
Les trois pages web (page1.html, page2.html, page3.html) ont été créées en amont et placées dans le répertoire adéquat sur la machine A, à l’endroit exact où pointe la configuration du serveur web Nginx.  

Une fois la solution déployée, il est important de la tester pour vérifier son bon fonctionnement. Pour cela, nous suivons une procédure simple en plusieurs étapes :
1) Tests de la disponibilité des pages webs :    
   L’objectif ici est de s’assurer que les pages sont bien accessibles via HTTP.  
   Pour ce faire, il suffit de se rendre dans le navigateur de notre machine personnelle et de tester les URLs suivantes :  
   ```html
   http://192.168.141.117:800/page1.html
   http://192.168.141.117:800/page2.html
   http://192.168.141.117:800/page3.html
   ```
   > Si le contenu des pages (par exemple, "Ceci est la page1") s’affiche correctement, cela valide cette étape.

2) Vérification des dashboards Grafana  
   L’objectif ici est de s’assurer que la supervision des services via Prometheus et Grafana fonctionne correctement.
   Pour cela, il faut se connecter à Grafana et vérifier que les dashboards créés remontent bien les informations suivantes :
   - Le statut du serveur NGINX.
   - Le nombre de requêtes traitées par le serveur NGINX.
   - Le nombre de sessions actives vers les pages web.
   - L’état de l'accès aux pages web.

   Voici à quoi ressemble le dashboard Grafana qui rassemble ces informations :
   ![Supervision NGNX et web](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Dashboard%20NGINX%20%2B%20WEB.png)

   Pour vérifier que les bonnes informations sont bien récupérées, un test de résilience a été effectué en désactivant le service NGINX sur la machine A. Cela a permis d'observer que, pendant un     court moment vers 13h00, le service NGINX était "DOWN", ce qui a bien été reflété sur le dashboard. Ce genre de test permet de s’assurer que la supervision réagit correctement aux pannes.
   En plus de la supervision du serveur web, nous avons également supervisé le système de la machine A pour obtenir des métriques telles que :
   - L’utilisation du CPU et de la RAM.
   - Les paquets entrants et sortants sur l'interface réseau "WAN" (ici enp0s8).

   Voici le dashboard Grafana dédié à la supervision système de la machine A :
   ![Supervision systèmes A](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Dashboard%20syste%CC%80me%20machine%20A.png)

Pour automatiser ces tests, un script bash a été créé. Il est exécuté sur la machine B, qui est chargée de tester l’accès aux pages et de vérifier la remontée des données dans Prometheus.  
Voici le script utilisé pour effectuer les tests automatiquement :
```bash
#!/bin/bash

SERVER="http://10.200.2.10:800"
PAGES=("page1.html" "page2.html" "page3.html")

echo "=== TEST DES PAGES WEB ==="
for page in "${PAGES[@]}"; do
    url="$SERVER/$page"
    echo -n "Test de $url ... "
    http_code=$(curl -o /dev/null -s -w "%{http_code}" "$url")
    if [ "$http_code" == "200" ]; then
        echo "✅ OK (200)"
    else
        echo "❌ Erreur ($http_code)"
    fi
done

echo ""
echo "=== TEST DE PROMETHEUS ==="
PROM_URL="http://192.168.141.230:80/api/v1/targets"
status=$(curl -s "$PROM_URL" | grep -o '"health":"up"' | wc -l)
if [ "$status" -ge 1 ]; then
    echo "✅ Prometheus reçoit des données."
else
    echo "❌ Aucune cible 'up' détectée dans Prometheus."
fi

```

Une fois ce script exécuté, les résultats obtenus étaient les suivants :
```bash
=== TEST DES PAGES WEB ===
Test de http://10.200.2.10:800/page1.html ... ✅ OK (200)
Test de http://10.200.2.10:800/page2.html ... ✅ OK (200)
Test de http://10.200.2.10:800/page3.html ... ✅ OK (200)

=== TEST DE PROMETHEUS ===
✅ Prometheus reçoit des données.
```
Ces résultats montrent que les pages sont accessibles (code HTTP 200) et que Prometheus récupère bien les informations de supervision des services.

# Netflow
Cette dernière partie n'a pas eu être réalisé par manque de temps. Le fait d'être tout seul pour ce projet a également demandée plus de temps sur les autres parties.
