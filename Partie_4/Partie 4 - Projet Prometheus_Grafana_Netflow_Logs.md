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

