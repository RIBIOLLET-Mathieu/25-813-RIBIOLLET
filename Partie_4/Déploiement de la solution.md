# 0) Configuration DNS afin de pouvoir télécharger les éléments ci-dessous
```bash
echo "nameserver 193.48.120.32" | sudo tee /etc/resolv.conf > /dev/null
```

# 1) Installation de Docker

```bash
sudo dnf install -y docker
sudo systemctl enable --now docker
docker --version
```

# 2) Installation de Docker Compose
```bash
sudo dnf install -y docker-compose
docker-compose --version
```

# 3) Installation, configuration et accès à Portainer

## 3.1) Placement de la machine B dans le vlan140
Ajout de l'interface NIC4. Obtention de l'IP 192.168.141.230
Ouverture des ports 9443 et 9000 sur la machine B pour être sûre.

## 3.2) Création du volume et lancement de Portainer.
```bash
podman volume create portainer_data
podman run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
```

Accès à Portainer --> ```https://192.168.141.230:9443``` (mdp : etrs813admin)

# 4) Préparation de la mise en place de Prometheus et Grafana
Tout sera placé dans un dossier monitoring
```bash
mkdir ~/monitoring
cd ~/monitoring
```
Rédaction d'un docker-compose.yml :
```yml
version: '3.7'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9443:9443"
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
```

Et d'un prometheus.yml :
```yml
global:
  scrape_interval: 15s  # Intervalle de récupération des métriques

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9443']
```

