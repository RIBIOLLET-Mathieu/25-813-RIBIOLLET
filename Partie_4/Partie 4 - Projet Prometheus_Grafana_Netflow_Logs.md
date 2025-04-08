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
- Cliquez sur `+ > Dashboard` dans le menu latéral.
- Cliquez sur `Add new panel` pour ajouter un nouveau panel. Chaque panel représente une source d'information qui apparaîtra sous forme d'élément dans la grille du tableau de bord.

### 2.2 Configurer un panel pour afficher le débit entrant

- Dans la section `Query`, sélectionnez `Prometheus` comme source de données.
- Entrez la requête suivante pour récupérer les octets entrants : ifHCInOctets{ifDescr="GigabitEthernet1", instance="10.200.2.254"}

Cette requête permet de récupérer le volume de données entrant sur l'interface réseau `GigabitEthernet1` de l'instance `10.200.2.254`. Elle filtre les données de Prometheus pour afficher le trafic entrant sur cette interface, offrant ainsi une vue détaillée de la performance du réseau pour cet équipement.
- Configurez l'axe Y avec l'unité `bytes/sec` pour une lecture plus pertinente des données.

### 2.3 Ajouter un panel pour le débit sortant
- Répétez les mêmes étapes pour ajouter un nouveau panel pour le débit sortant.
- Utilisez la requête suivante pour récupérer les octets sortants : ifHCOutOctets{ifDescr="GigabitEthernet2", instance="10.200.2.253"}

### 2.4 Personnalisation des panels
- Nommez chaque panel pour améliorer la lisibilité du tableau de bord.
- Vous pouvez également personnaliser le type de visualisation de chaque panel (graphique, jauge, etc.) pour rendre les données encore plus claires et parlantes.

### 2.5 Sauvegarder le tableau de bord
- Avant de sauvegarder, donnez un nom explicite à votre tableau de bord.
- Cliquez sur `Save` pour enregistrer les modifications et finaliser la création du tableau de bord.



### Procédure de tests 
Afin de tester si notre service snmp-exporter collecte correctement les informations et que notre solution Prometheus+Grafana fonctionne nous allons générer du traffic depuis le réseau interne vers l'extérieur. Les graphiques de débits ("Dashboard") verront alors des données apparaître.  
Pour simuler l'utilisation du réseau par une production cliente nous avons rédigé le script ci-dessous. Il généré 10 iperf avec différents débits aléatoires (de 1M à 5M).  
```bash
#!/bin/bash

# --- TESTS PARTIE PROJET ---

# Adresse IP du serveur iperf3
SERVER="192.168.141.230"

# Nombre de tests
COUNT=10

# Durée de chaque test (en secondes)
DURATION=30

echo "Simulation d'un trafic client avec débit variable..."

for i in $(seq 1 $COUNT); do
    # Génère un débit aléatoire entre 5 et 100 Mbps
    BANDWIDTH=$(( RANDOM % 5 + 1 ))M # 5M à 100M

    echo "Test $i/$COUNT avec un débit simulé de $BANDWIDTH"
    iperf3 -c "$SERVER" -b "$BANDWIDTH" -t "$DURATION"
    echo "-------------------------------"
    sleep 2
done

echo "Simulation terminée."
```
Les informations de débits des interfaces s'affichent alors sur nos Dashboard Grafana, validant ainsi le fonctionnement de notre solution jusqu'à présent. Par exemple, voici le débit entrant et sortant observé sur l'interface GigabitEthernet2 du routeur R1. Ces débits font suite à l'éxécution du script ci-dessus.  
![VIII-Débits des interfaces de R1](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Tests%20de%CC%81bits.png)  

## Dashboard - Interfaces ayant le plus de débit sur la dernière heure.











## Dashboard - sysUptime des routeurs

Le fichier "snmp.yml", déployé plus tôt prenait déjà en compte la collecte des informations "uptime" dans routeurs, ainsi il nous a simplement fallu importer un dashboard permettant d'afficher l'uptime des équipements. Ce dashboard a été trouvé sur un GitHub (après une recherche "Dashboard uptime Github sur internet).  
Ce tableau de bord nous permet de surveiller le système "uptime" des nos routeurs, qui indique depuis combien de temps l’équipement est en fonctionnement.  
