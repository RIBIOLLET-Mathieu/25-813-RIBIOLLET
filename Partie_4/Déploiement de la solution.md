# 0) Configuration DNS afin de pouvoir télécharger les éléments ci-dessous
```bash
echo "nameserver 193.48.120.32" | sudo tee /etc/resolv.conf > /dev/null
```

# 1) Récupération de l'image
En faisant la recherche internet "github docker-compose" je me suis rendu sur le github où sont placées les releases de docker-compose.  
On a copié le l'adresse du lien de la version nous intéressant (docker-compose-linux-x86_64). Ce lien copié j'ai utilisé la commande "wget" pour récupérer l'éxécutable.  
Le fichier récupéré, nous l'avons placer dans le dossier "bin" afin de pouvoir l'éxécuter.  
```bash
wget https://github.com/docker/compose/releases/download/v2.34.0/docker-compose-linux-x86_64
```

# 2) Installation de Prometheus
## 2.1) Préparation
Dans un dossier "prometheus", nous avons créer le fichier "docker-compose.yml", dont le contenu est :
```yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./data:/prometheus
    ports:
      - "80:9090"
```
Note : Le "80:9090" dans la catégorie "ports" nous permet d'accèder depuis le VPN de l'université (car le port 9090 n'est pas directement ouvert).

Dans un dossier /prometheus/config on crée le fichier "prometheus.yml", dont le contenu est :
```yml
global:
  scrape_interval: 15s  # Fréquence de collecte des métriques

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]
```
## 2.2) Création et lancement du contenur
On peut maintenant entrée la commande suivante afin de lancer le conteneur Prometheus.  
```bash
docker-compose-linux-x86_64 up -d
```
On vérifie via la commande ```bash docker ps```
Prometheus est maintenant accessible sur l'@IP 192.168.141.230:80

# 3) Installation de Grafana
## 3.1) Préparation
## 3.2) Lancement


