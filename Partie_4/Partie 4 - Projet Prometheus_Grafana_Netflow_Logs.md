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
Prometheus est un outil de supervision open-source conçu pour collecter, stocker et analyser des métriques sous forme de séries temporelles. Il est souvent utilisé avec Grafana pour la visualisation des données.

## Avantages de Prometheus
Prometheus se distingue des autres solutions de supervision par plusieurs aspects :

| Critère               | Prometheus                                      | Autres solutions (ex: Nagios, Zabbix) |
|-----------------------|-----------------------------------------------|--------------------------------------|
| **Modèle de collecte** | Basé sur un modèle **pull**                    | Modèle souvent **push**             |
| **Stockage**          | Base de données optimisée pour les séries temporelles | Base relationnelle ou fichiers    |
| **Scalabilité**       | Conçu pour gérer des infrastructures dynamiques (Cloud, Kubernetes) | Moins adapté aux environnements dynamiques |
| **Configuration**     | Déclaration en YAML, flexible et simple        | Configuration plus complexe        |
| **Visualisation**     | Intégration native avec **Grafana**             | Interfaces propriétaires ou limitées |
| **Gestion des alertes** | Intégration avec **Alertmanager**               | Systèmes d’alertes souvent intégrés mais moins flexibles |
| **Communauté**       | Large communauté et support actif              | Moins actif selon la solution     |

### Composants principaux
- **Prometheus Server** : Collecte et stocke les métriques dans une base de données optimisée pour les séries temporelles.
- **Exporters** : Applications qui exposent des métriques pour être collectées par Prometheus.
- **Alertmanager** : Gère les alertes définies sur les métriques collectées.
- **Grafana** : Outil de visualisation des données de Prometheus.

## Collecte des métriques
Prometheus interroge périodiquement des cibles (serveurs, équipements réseau, applications) qui exposent des métriques au format HTTP.

### Exemples d’exporters courants
- **Node Exporter** : Collecte les données système d’un serveur (CPU, mémoire, disque, etc.).
- **Blackbox Exporter** : Permet de tester la disponibilité des services via des requêtes HTTP, ICMP, TCP.
- **cAdvisor** : Fournit des métriques sur l’utilisation des conteneurs Docker/Kubernetes.
- **SNMP Exporter** : Collecte les métriques des équipements réseau via SNMP.

## Configuration de base de Prometheus
La configuration de Prometheus est définie dans un fichier `prometheus.yml`. Voici un exemple minimal :

```yaml
global:
  scrape_interval: 15s  # Fréquence de collecte des métriques

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

## Alerting avec Alertmanager
Prometheus permet de définir des règles d’alerte qui déclenchent des notifications envoyées par Alertmanager.

Exemple d’alerte sur une forte utilisation CPU :
```yaml
- alert: HighCPUUsage
  expr: node_cpu_seconds_total{mode="idle"} < 10
  for: 2m
  labels:
    severity: critical
  annotations:
    summary: "CPU élevé sur {{ $labels.instance }}"
```

## Visualisation avec Grafana
Grafana se connecte à Prometheus pour afficher les métriques sous forme de graphiques et de tableaux.

### Étapes pour ajouter Prometheus comme source de données dans Grafana :
1. Aller dans **Configuration > Data Sources**
2. Ajouter une source **Prometheus**
3. Renseigner l’URL de Prometheus (`http://prometheus:9090`)
4. Sauvegarder et tester

## Conclusion
Prometheus est une solution robuste pour la supervision d’infrastructures modernes. Son intégration avec Grafana et Alertmanager permet une surveillance efficace et proactive des systèmes et applications.

# Mise en place

Nous avons déjà mis en place Prometheus et Grafana via une solution dockerisé avec docker-compose. Voir les détails de ce déploiement dans [Déploiement Prometheus/Grafana](https://github.com/RIBIOLLET-Mathieu/25-813-RIBIOLLET/blob/main/Partie_4/D%C3%A9ploiement%20de%20la%20solution.md)  
Nous avons ensuite lié Grafana à Prometheus en indiquant Prometheus comme source de données sur Grafana.  

## SNMP Exporter
Prometheus ne sait pas communiquer en SNMP nativement, nous devons installer un service nommé "snmp-exporter" pour cela. "snmp-exporter" est un service qui transforme SNMP en HTTP pour que Prometheus puisse le parcourir.  
On ajout donc cela au fichier "docker-compose.yml" :  
```yml
snmp-exporter:
    image: prom/snmp-exporter
    container_name: snmp-exporter
    ports:
      - "9116:9116"
    volumes:
      - ./snmp/snmp.yml:/etc/snmp_exporter/snmp.yml
```
Avant de monter ce service, nous devons créer le fichier snmp.yml dans le dossier snmp que nous créerons également. Ce fichier permet de définir les informations que nous allons récupérer sur les machines. 
Le contenu du fihcier "snmp.yml" est :  
```yml
default:
  version: 2
  auth:
    community: 123test123
  walk:
    - 1.3.6.1.2.1.2.2.1.10  # ifInOctets de toutes les interfaces
    - 1.3.6.1.2.1.2.2.1.16  # ifOutOctets de toutes les interfaces
  metrics:
    - name: ifInOctets
      oid: 1.3.6.1.2.1.2.2.1.10
      type: counter
    - name: ifOutOctets
      oid: 1.3.6.1.2.1.2.2.1.16
      type: counter
```
Enfin, nous ajoutons le job SNMP dans le fichier prometheus.yml. Pour cela, on ajoute le bloc yml suivant dans scrape_configs :  
```yml
  - job_name: 'snmp'
    metrics_path: /snmp
    params:
      module: [default]
    static_configs:
      - targets: ['10.200.2.254','10.200.2.253']  # IP des routeurs
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: snmp-exporter:9116
```
