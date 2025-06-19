# Calibre Server Installatie

Dit project zet een Calibre e-book server op met Docker, met Nginx als reverse proxy, beheerd via Ansible. Volg deze stappen om het project te starten na het clonen van de repository.

## Vereisten
- **Besturingssysteem**: CentOS of een vergelijkbaar RHEL-gebaseerd systeem.
- **Benodigde Tools**:
  - Git
  - Ansible (`pip install ansible`)
  - Docker en Docker Compose
- **Toegang**: Root- of sudo-rechten.
- **Netwerk**: Zorg ervoor dat de host bereikbaar is (standaard IP: `192.168.232.128`).

## Installatie-instructies

### 1. Clone de Repository
```bash
git clone <repository-url>
cd <repository-directory>
```

### 2. Installeer Afhankelijkheden
Installeer de benodigde pakketten op de host:
```bash
sudo dnf install -y git ansible python3-pip
sudo pip3 install ansible
```

### 3. Configureer Inventory
Bewerk het `inventory.yml` bestand om de host en variabelen te controleren:
```bash
nano inventory.yml
```
Zorg ervoor dat het volgende bevat:
```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
  vars:
    calibre_server:
      user: calibre
      group: calibre
      timezone: Europe/Amsterdam
      password: secure_calibre_password_123
      gui_port: 8080
      content_port: 8081
```
Sla op en sluit af (`Ctrl+O`, `Enter`, `Ctrl+X`).

### 4. Voer de Ansible Playbook uit
Voer de playbook uit om Docker in te stellen, configuraties te deployen en containers te starten:
```bash
sudo ansible-playbook -i inventory.yml playbook.yml
```

### 5. Configureer Firewall
Open de benodigde poorten voor externe toegang:
```bash
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --zone=public --add-port=8081/tcp --permanent
sudo firewall-cmd --reload
```

### 6. Controleer Containers
Controleer of de Calibre en Nginx containers draaien:
```bash
sudo docker ps
```
Verwachte uitvoer:
- `calibre` container op poorten `8080` en `8081`.
- `nginx` container op poort `80`.

### 7. Test Toegang
#### Op de Host
```bash
curl http://localhost:80
curl http://localhost:8080
curl http://localhost:8081/content
```
Open in een browser (indien beschikbaar):
- `http://localhost:8080` (Calibre GUI)
- `http://localhost:8081/content` (Content Server)

#### Vanaf je PC
Open in een browser:
- `http://192.168.232.128` (via Nginx)
- `http://192.168.232.128:8080` (Calibre GUI)
- `http://192.168.232.128:8081/content` (Content Server)

**Inloggegevens**:
- **Gebruikersnaam**: Laat leeg of gebruik `admin`.
- **Wachtwoord**: `secure_calibre_password_123`.

### 8. Problemen Oplossen
Als toegang niet werkt:
- Controleer container logs:
  ```bash
  sudo docker logs calibre
  sudo docker logs nginx
  ```
- Controleer Nginx configuratie:
  ```bash
  cat /home/calibre/calibre-server/nginx/conf/default.conf
  ```
- Controleer firewall:
  ```bash
  sudo firewall-cmd --list-ports
  ```
- Test netwerk:
  ```bash
  ping 192.168.232.128
  nc -zv 192.168.232.128 8080
  ```

### Configuratiedetails
- **Docker Compose**: Gelegen op `/home/calibre/calibre-server/docker-compose.yml`.
- **Nginx Config**: Gelegen op `/home/calibre/calibre-server/nginx/conf/default.conf`.
- **Calibre Data**: Opgeslagen in `/home/calibre/calibre-server/config` en `/home/calibre/calibre-server/books`.

## Opmerkingen
- Zorg ervoor dat geen andere processen poorten 80, 8080 of 8081 gebruiken voordat je de playbook uitvoert.
- Als er authenticatieproblemen zijn, inspecteer `/home/calibre/calibre-server/config` voor aangepaste gebruikersinstellingen.
- Voor verdere hulp, raadpleeg de Ansible playbook logs of Docker container logs.
