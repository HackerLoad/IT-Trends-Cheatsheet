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
   - Subnet: `sub1_pub` ← **vorerst öffentlich, damit httpd installiert werden kann!**
   - Auto-assign public IP: **Enable** ← **vorerst aktivieren**
4. **Security Group** erstellen:
   - Name: `sg-private`
   - Inbound Rules:
     - SSH (Port 22) – Source: `0.0.0.0/0` ← **vorerst offen für Installation**
     - HTTP (Port 80) – Source: `0.0.0.0/0` ← **vorerst offen für Installation**
5. **"Launch Instance"** klicken

---

## 6. HTTP-Daemon einrichten

> **Quick & Dirty:** Die Instanz läuft zunächst im öffentlichen Subnetz, damit `yum` das Internet erreichen kann. Nach der Installation wird sie ins private Subnetz verschoben.

### Schritt 1: Mit der (noch öffentlichen) privaten Instanz verbinden

```bash
ssh -i dein-keypair.pem ec2-user@<public-ip-der-privaten-instanz>
```

### Schritt 2: Apache installieren und konfigurieren

```bash
# Apache installieren
sudo yum install -y httpd

# Einfache HTML-Seite erstellen
echo "Hallo, ich bin eine private Instanz" | sudo tee /var/www/html/index.html

# Apache starten und dauerhaft aktivieren
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Schritt 3: Instanz ins private Subnetz verschieben

Da AWS das nachträgliche Ändern des Subnetzes einer laufenden Instanz nicht direkt erlaubt, gibt es zwei Möglichkeiten:

**Option A – Instanz stoppen und Subnetz über AMI-Rebuild wechseln:**
1. Instanz stoppen
2. Rechtsklick → **"Create Image"** (AMI erstellen)
3. Neue Instanz aus diesem AMI starten, diesmal mit:
   - Subnet: `sub2_priv`
   - Auto-assign public IP: **Disable**

**Option B – Sicherheitsgruppe einschränken (schneller für die Klausur):**

Die Instanz bleibt physisch in `sub1_pub`, wird aber durch die Sicherheitsgruppe so abgeschottet, dass sie nur noch aus dem VPC erreichbar ist:

1. EC2 → Instanz auswählen → **"Security"** → Sicherheitsgruppe `sg-private` öffnen
2. Inbound Rules anpassen:
   - SSH (Port 22) – Source: `192.168.70.0/25` ← nur noch aus dem VPC
   - HTTP (Port 80) – Source: `192.168.70.0/25` ← nur noch aus dem VPC
3. Auto-assign Public IP spielt dann keine Rolle mehr – von außen kommt niemand durch

> Für die Klausur reicht Option B vollständig aus. Der Screenshot der Sicherheitsgruppe zeigt dann die eingeschränkten Regeln, und der Browser-Timeout sowie der funktionierende `curl` vom Bastion Host beweisen das gewünschte Verhalten.

---

## 7. Zugriff testen

### Von außen (muss fehlschlagen)

Rufe in deinem lokalen Browser folgende URL auf:

```
http://<private-ip>
```

Erwartetes Ergebnis: **Timeout** – kein Zugriff von außen möglich.

### Von innen (muss funktionieren)

Verbinde dich zunächst mit dem Bastion Host und führe dort aus:

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
