# WireGuard + wg-easy – Vollständige Installationsanleitung (Docker, Ubuntu 22/24)
Stand: 2025

## Übersicht
Diese Anleitung installiert:
- Docker Engine
- Docker Compose Plugin
- WireGuard
- wg-easy WebUI
- Passwort-Hash (SHA256)
- Firewall-Regeln
- Start & Backup

Alle Befehle sind einzeln kopierbar und iPhone‑freundlich.

---

# 1. Voraussetzungen
- Ubuntu 22.04 oder 24.04
- Root-Rechte
- Öffentliche IPv4
- UDP-Port 51820 muss geöffnet werden

---

# 2. System aktualisieren

sudo apt update

sudo apt upgrade -y

---

# 3. Docker installieren

## 3.1 Pakete installieren

sudo apt install -y ca-certificates curl gnupg lsb-release

## 3.2 Defekten Docker-Key löschen (Hetzner Fix)

sudo rm -f /usr/share/keyrings/docker.gpg

## 3.3 Docker GPG-Key neu hinzufügen

curl -fsSL https://download.docker.com/linux/ubuntu/gpg

sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg

## 3.4 Docker Repository eintragen

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

## 3.5 Update

sudo apt update

## 3.6 Docker installieren

sudo apt install -y docker-ce

sudo apt install -y docker-ce-cli

sudo apt install -y containerd.io

sudo apt install -y docker-compose-plugin

## 3.7 Docker aktivieren

sudo systemctl enable docker

---

# 4. Ordner für wg-easy erstellen

mkdir -p ~/wg-easy

cd ~/wg-easy

---

# 5. Passwort (SHA256 Hash) erstellen

## 5.1 Passwort unsichtbar eingeben

read -s PASSWORD

## 5.2 SHA256 Hash erzeugen

echo -n "$PASSWORD" | sha256sum | awk '{print $1}'

Hash kopieren → wird in docker-compose.yml eingetragen.

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

## Hetzner Firewall

UDP 51820 erlauben

## UFW lokal

sudo ufw allow 51820/udp

---

# 10. Backup

Sichere:

~/wg-easy/config

Enthält alle Keys & Konfigurationen.

---

# Fertig
WireGuard + wg-easy ist vollständig eingerichtet und einsatzbereit.
