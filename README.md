# WireGuard + wg-easy – Vollständige Installationsanleitung (Docker, Ubuntu 22/24)
Stand: 2025

Diese README ist **vollständig GitHub‑Mobile kompatibel**.  
Alle Befehle sind in **```bash**‑Codeblöcken** – dadurch erscheint auf GitHub der **Copy‑Button**.

---

# 1. Voraussetzungen

- Ubuntu 22.04 oder 24.04  
- Root‑Zugriff  
- Öffentliche IPv4  
- UDP‑Port 51820 muss freigegeben werden

---

# 2. System aktualisieren

```bash
sudo apt update
```

```bash
sudo apt upgrade -y
```

---

# 3. Docker installieren

## 3.1 Benötigte Pakete

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

## 3.2 Defekten Docker-Key löschen (Hetzner Fix)

```bash
sudo rm -f /usr/share/keyrings/docker.gpg
```

## 3.3 Docker GPG-Key neu hinzufügen

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o docker.gpg
```

```bash
sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg docker.gpg
```

```bash
rm docker.gpg
```

## 3.4 Docker Repository eintragen

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

## 3.5 Paketlisten aktualisieren

```bash
sudo apt update
```

## 3.6 Docker installieren

```bash
sudo apt install -y docker-ce
```

```bash
sudo apt install -y docker-ce-cli
```

```bash
sudo apt install -y containerd.io
```

```bash
sudo apt install -y docker-compose-plugin
```

## 3.7 Docker aktivieren

```bash
sudo systemctl enable docker
```

---

# 4. Ordner für wg-easy erstellen

```bash
mkdir -p ~/wg-easy
```

```bash
cd ~/wg-easy
```

---

# 5. Passwort-Hash (bcrypt) für wg-easy erzeugen

## 5.1 dein Passwort eingeben

```bash
docker run --rm -it ghcr.io/wg-easy/wg-easy wgpw 'DEIN-PASSWORT-HIER'
```

Hash kopieren → kommt in die docker-compose.yml

---

# 6. docker-compose.yml erstellen

```bash
nano docker-compose.yml
```

### Inhalt einfügen:

```bash
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
```

Speichern:  
CTRL + O → ENTER  
CTRL + X

---

# 7. wg-easy starten

```bash
docker compose up -d
```

---

# 8. Webinterface öffnen

```bash
http://SERVER-IP:51821
```

Login:  
admin + dein Passwort

---

# 9. Firewall konfigurieren

## Hetzner Firewall

```bash
UDP 51820 erlauben
```

## UFW lokal

```bash
sudo ufw allow 51820/udp
```

---

# 10. Backup

```bash
~/wg-easy/config
```

Dieser Ordner enthält alle Keys & Client‑Konfigurationen.

---

# Fertig

WireGuard + wg‑easy ist vollständig eingerichtet und einsatzbereit.
