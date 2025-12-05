# WireGuard VPN Installation mit wg-easy (Docker)
Aktuelle, komplette und stabile Anleitung (Stand 2025)

## Übersicht
Diese Anleitung installiert WireGuard + Web-Interface (wg-easy) mit Docker.
Alle Schritte sind einzeln kopierbar und für Anfänger geeignet.

# 1. Voraussetzungen
- Ubuntu 22.04 oder 24.04
- Root-Zugriff
- Öffentliche IPv4
- UDP Port 51820 später freigeben

# 2. System aktualisieren
sudo apt update
sudo apt upgrade -y

# 3. Docker installieren
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo rm -f /usr/share/keyrings/docker.gpg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl enable docker

# 4. Ordner erstellen
mkdir -p ~/wg-easy
cd ~/wg-easy

# 5. Passwort-Hash erzeugen
read -s PASSWORD
echo -n "$PASSWORD" | sha256sum | awk '{print $1}'

# 6. docker-compose.yml erstellen
nano docker-compose.yml

# Inhalt einfügen:
version: "3.8"

services:
  wg-easy:
    container_name: wg-easy
    image: ghcr.io/wg-easy/wg-easy:latest
    environment:
      - WG_HOST=SERVER-IP-ODER-DOMAIN
      - PASSWORD_HASH=HIER_DEIN_SHA256_HASH
    volumes:
      - ./config:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
    restart: always

# 7. Starten
docker compose up -d

# 8. Webinterface
http://SERVER-IP:51821

Login:
admin + dein Passwort

# 9. Firewall
Hetzner: UDP 51820 erlauben
UFW lokal: sudo ufw allow 51820/udp

# 10. Backup
~/wg-easy/config sichern

# Fertig
WireGuard läuft vollständig.
