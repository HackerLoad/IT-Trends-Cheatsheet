# AWS VPC Anleitung – Klausur WI24 Aufgabe 3

## Inhaltsverzeichnis
1. [VPC erstellen](#1-vpc-erstellen)
2. [Subnetze erstellen](#2-subnetze-erstellen)
3. [Internet Gateway & Routing](#3-internet-gateway--routing)
4. [Public EC2-Instanz erstellen](#4-public-ec2-instanz-erstellen-bastion-host)
5. [Private EC2-Instanz erstellen](#5-private-ec2-instanz-erstellen)
6. [HTTP-Daemon einrichten](#6-http-daemon-einrichten)
7. [Zugriff testen](#7-zugriff-testen)

---

## 1. VPC erstellen

1. Melde dich im **AWS Academy Learner Lab** an und starte die Lab-Umgebung
2. Navigiere in der AWS-Konsole zu **VPC** (Suchleiste: "VPC")
3. Klicke links auf **"Your VPCs"** → **"Create VPC"**
4. Wähle **"VPC only"**
5. Fülle folgendes aus:
   - Name: `klausurwi24`
   - IPv4 CIDR: `192.168.70.0/25`
   - Rest bleibt Standard
6. Klicke **"Create VPC"**

> Das `/25` gibt dir 128 IP-Adressen (126 nutzbar für Hosts).

---

## 2. Subnetze erstellen

Navigiere zu **"Subnets"** → **"Create subnet"** und wähle das VPC `klausurwi24` aus.

Das Netz `192.168.70.0/25` wird in 3 Subnetze aufgeteilt:

| Name | CIDR | Nutzbare Hosts |
|---|---|---|
| `sub1_pub` | `192.168.70.0/27` | 30 |
| `sub2_priv` | `192.168.70.32/27` | 30 |
| `sub3` | `192.168.70.64/26` | 62 |

Erstelle jedes Subnetz einzeln mit dem jeweiligen Namen und CIDR-Block.

> `sub3` erhält den größten verbleibenden Block (`/26`), damit insgesamt die meisten Hosts adressierbar sind.

---

## 3. Internet Gateway & Routing

### Internet Gateway erstellen

1. Gehe zu **"Internet Gateways"** → **"Create internet gateway"**
   - Name: `igw-klausurwi24`
2. Nach dem Erstellen: **"Actions"** → **"Attach to VPC"** → `klausurwi24` auswählen

### Route Table für sub1_pub erstellen

1. Gehe zu **"Route Tables"** → **"Create route table"**
   - Name: `rt-public`
   - VPC: `klausurwi24`
2. Route Table auswählen → **"Routes"** → **"Edit routes"** → **"Add route"**
   - Destination: `0.0.0.0/0`
   - Target: dein Internet Gateway (`igw-klausurwi24`)
3. **"Subnet associations"** → **"Edit subnet associations"** → `sub1_pub` auswählen

> `sub2_priv` und `sub3` verbleiben in der Default-Route-Table ohne Internet-Route und sind damit privat.

---

## 4. Public EC2-Instanz erstellen (Bastion Host)

1. Navigiere zu **EC2** → **"Launch Instance"**
2. Einstellungen:
   - Name: `bastion-host`
   - AMI: **Amazon Linux 2023**
   - Instance type: `t2.micro`
   - Key pair: vorhandenes auswählen oder neues erstellen
3. **"Network settings"** → **"Edit"**:
   - VPC: `klausurwi24`
   - Subnet: `sub1_pub`
   - Auto-assign public IP: **Enable**
4. **Security Group** erstellen:
   - Name: `sg-public`
   - Inbound Rule: SSH (Port 22) – Source: `0.0.0.0/0`
5. **"Launch Instance"** klicken

---

## 5. Private EC2-Instanz erstellen

1. Navigiere zu **EC2** → **"Launch Instance"**
2. Einstellungen:
   - Name: `private-instance`
   - AMI: **Amazon Linux 2023**
   - Instance type: `t2.micro`
   - Key pair: dasselbe wie beim Bastion Host
3. **"Network settings"** → **"Edit"**:
   - VPC: `klausurwi24`
   - Subnet: `sub2_priv`
   - Auto-assign public IP: **Disable**
4. **Security Group** erstellen:
   - Name: `sg-private`
   - Inbound Rules:
     - SSH (Port 22) – Source: `192.168.70.0/25`
     - HTTP (Port 80) – Source: `192.168.70.0/25`
5. **"Launch Instance"** klicken

> Keine Regel für `0.0.0.0/0` – die Instanz ist ausschließlich aus dem VPC erreichbar.

---

## 6. HTTP-Daemon einrichten

### Schritt 1: Key auf den Bastion Host kopieren

```bash
scp -i dein-keypair.pem dein-keypair.pem ec2-user@<public-ip>:/home/ec2-user/
```

### Schritt 2: Mit dem Bastion Host verbinden

```bash
ssh -i dein-keypair.pem ec2-user@<public-ip>
```

### Schritt 3: Berechtigungen setzen

```bash
chmod 400 dein-keypair.pem
```

### Schritt 4: Von dort zur privaten Instanz verbinden

```bash
ssh -i dein-keypair.pem ec2-user@<private-ip>
```

### Schritt 5: Apache installieren und konfigurieren

```bash
# Apache installieren
sudo yum install -y httpd

# Einfache HTML-Seite erstellen
echo "Hallo, ich bin eine private Instanz" | sudo tee /var/www/html/index.html

# Apache starten und dauerhaft aktivieren
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

## 7. Zugriff testen

### Von außen (muss fehlschlagen)

Rufe in deinem lokalen Browser folgende URL auf:

```
http://<private-ip>
```

Erwartetes Ergebnis: **Timeout** – kein Zugriff von außen möglich.

### Von innen (muss funktionieren)

Führe auf dem Bastion Host folgenden Befehl aus:

```bash
curl http://<private-ip>
```

Erwartetes Ergebnis:

```
Hallo, ich bin eine private Instanz
```

---

## Benötigte Screenshots

| Screenshot | Wo in der Konsole |
|---|---|
| Ressourcenzuordnung des VPC | VPC-Konsole → dein VPC → Tab "Resource map" |
| Sicherheitsgruppe `sg-private` | EC2 → Security Groups → `sg-private` → Inbound Rules |
| Kein Zugriff von außen | Browser-Timeout beim Aufruf der privaten IP |
| Zugriff von innen funktioniert | `curl`-Output vom Bastion Host |

---

## Theorie

### VPC – doppeldeutige Bedeutung

- **Virtual Private Cloud**: Logisch isoliertes Netzwerk in AWS
- **Virtual Private Connection**: Verschlüsselte Verbindung zwischen Netzwerken (VPN)

### Vorteile der Konstellation für den Praxispartner

1. Interne Server (z.B. Datenbank, Backend) sind von außen nicht erreichbar – erhöhte Sicherheit
2. Klare Netzwerksegmentierung zwischen öffentlichen und privaten Diensten

### Nachteil

- Höherer Verwaltungsaufwand: Für den Zugriff auf private Instanzen wird ein Bastion Host, VPN oder AWS Systems Manager benötigt
