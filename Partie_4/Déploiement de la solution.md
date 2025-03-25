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
```bash
docker volume create portainer_data
docker run -d \
  -p 9000:9000 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce

```
Accès à Portainer --> ```http://localhost:9000```
