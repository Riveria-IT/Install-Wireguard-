# WireGuard VPN Installation mit wg-easy (Docker)
Aktuell, stabil und anfängerfreundlich (Stand 2025)

## Übersicht
Diese Anleitung installiert:
- Docker
- WireGuard
- wg-easy (Web UI)
- Passwort-Hash (SHA256)
- Firewall-Regeln
- Start & Nutzung

Alle Befehle einzeln kopierbar und mobile-freundlich.

---

# 1. Voraussetzungen
- Ubuntu 22.04 oder 24.04
- Root-Zugriff
- Öffentliche IPv4
- UDP Port 51820 muss freigegeben werden

---

# 2. System aktualisieren

sudo apt update  
sudo apt upgrade -y

---

# 3. Docker installieren

## Benötigte Pakete

sudo apt install -y ca-certificates curl gnupg lsb-release

## Defekten Docker-Key entfernen (Fix)

sudo rm -f /usr/share/keyrings/docker.gpg

## Docker GPG-Key neu hinzufügen

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg

## Docker Repository eintragen

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

## Paketlisten aktualisieren

sudo apt update

## Docker installieren

sudo apt install -y docker-ce  
sudo apt install -y docker-ce-cli  
sudo apt install -y containerd.io  
sudo apt install -y docker-compose-plugin

## Docker aktivieren

sudo systemctl enable docker

---

# 4. Ordner für wg-easy erstellen

mkdir -p ~/wg-easy  
cd ~/wg-easy

---

# 5. Passwort für wg-easy erstellen (SHA256-Hash)

## Passwort sicher eingeben (unsichtbar)

read -s PASSWORD

## SHA256-Hash erzeugen

echo -n "$PASSWORD" | sha256sum | awk '{print $1}'

Den ausgegebenen Hash kopieren und in die docker-compose.yml einfügen.

---

# 6. docker-compose.yml erstellen

nano docker-compose.yml

### Inhalt einfügen:

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

Speichern:  
CTRL + O → ENTER  
CTRL + X

---

# 7. wg-easy starten

docker compose up -d

---

# 8. Webinterface öffnen

http://SERVER-IP:51821

Login:  
admin + dein Passwort

---

# 9. Firewall konfigurieren

## Hetzner Cloud Firewall

UDP 51820 erlauben

## UFW lokal

sudo ufw allow 51820/udp

---

# 10. Backup

Folgenden Ordner sichern:

~/wg-easy/config

---

# Fertig
WireGuard läuft stabil und ist einsatzbereit.