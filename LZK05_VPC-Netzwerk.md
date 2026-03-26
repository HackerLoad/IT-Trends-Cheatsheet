# Cloud Computing – Eigenes Netzwerk (VPC)

## VPC erstellen

Ein VPC (Virtual Private Cloud) ist ein logisch isoliertes, virtuelles Netzwerk innerhalb von AWS.

Beim Erstellen werden festgelegt:
- **Name**: z.B. `klausurwi24`
- **CIDR-Block**: z.B. `192.168.70.0/25`

---

## CIDR-Wert verstehen

CIDR (Classless Inter-Domain Routing) definiert den IP-Adressbereich eines Netzwerks.

```
192.168.70.0/25
              ^--- Präfixlänge: 25 Bits für Netzwerk, 7 Bits für Hosts
                   → 2^7 = 128 Adressen, davon 2 reserviert (Netz + Broadcast)
                   → 126 nutzbare Hosts
```

### Schnellübersicht: Hosts je CIDR

| CIDR | Gesamte Adressen | Nutzbare Hosts |
|------|-----------------|----------------|
| /24  | 256 | 254 |
| /25  | 128 | 126 |
| /26  | 64  | 62  |
| /27  | 32  | 30  |
| /28  | 16  | 14  |

---

## Subnetze berechnen

Wenn aus einem VPC `192.168.70.0/25` maximal **4 Subnetze** entstehen sollen:

- 4 Subnetze = 2² → 2 Bits für Subnetze abzweigen
- Neue Präfixlänge: /25 + 2 = **/27**
- Hosts pro Subnetz: 2^(32-27) - 2 = 30 nutzbare Hosts

```
VPC:       192.168.70.0/25
Subnetz 1: 192.168.70.0/27   → Hosts: .1 – .30
Subnetz 2: 192.168.70.32/27  → Hosts: .33 – .62
Subnetz 3: 192.168.70.64/27  → Hosts: .65 – .94
Subnetz 4: 192.168.70.96/27  → Hosts: .97 – .126
```

---

## Subnetze mit eigener Routing-Tabelle und Internetzugang

Damit eine EC2-Instanz im Subnetz das Internet nutzen kann:

1. **Internet Gateway** erstellen und an das VPC anhängen
2. **Routing-Tabelle** für das public Subnetz anlegen
3. Route hinzufügen: `0.0.0.0/0` → Internet Gateway
4. Routing-Tabelle dem Subnetz zuordnen
5. EC2-Instanz muss eine **öffentliche IP** oder Elastic IP haben

```
EC2 (public Subnetz)
        |
  Routing-Tabelle
  0.0.0.0/0 → igw-xxxxx
        |
  Internet Gateway
        |
     Internet
```

---

## EC2-Instanz mit HTTP-Daemon einrichten

```bash
# Apache installieren (Amazon Linux 2)
sudo yum install -y httpd

# Inhalt der Startseite setzen
echo "<html><body><h1>Hello PUBLIC</h1></body></html>" | sudo tee /var/www/html/index.html

# Dienst starten und dauerhaft aktivieren
sudo systemctl start httpd
sudo systemctl enable httpd

# Lokal testen
curl localhost
```

Für eine private Instanz entsprechend:
```bash
echo "<html><body><h1>Hello PRIVATE</h1></body></html>" | sudo tee /var/www/html/index.html
```

---

## Sicherheitsgruppen konfigurieren

### Public-Instanz (von außen erreichbar)

```
Eingehende Regeln:
- HTTP  | Port 80  | Quelle: 0.0.0.0/0         (alle)
- HTTPS | Port 443 | Quelle: 0.0.0.0/0         (alle)
- SSH   | Port 22  | Quelle: eigene IP (optional)
```

### Private Instanz (nur aus dem VPC erreichbar)

```
Eingehende Regeln:
- HTTP | Port 80 | Quelle: 192.168.70.0/25   (nur VPC-intern)
- SSH  | Port 22 | Quelle: 192.168.70.0/25   (nur VPC-intern)

Kein Eintrag für 0.0.0.0/0 → kein Zugriff von außen
```

---

## Public vs. Private Subnetz

| Eigenschaft | Public Subnetz | Private Subnetz |
|-------------|---------------|-----------------|
| Internet Gateway | Vorhanden (Route 0.0.0.0/0) | Nicht direkt; ggf. NAT Gateway |
| Öffentliche IP | Ja | Nein |
| Erreichbar von außen | Ja | Nein |
| Typischer Inhalt | Webserver, Load Balancer | Datenbanken, Backend-Logik |

---

## Praxispartner-Anwendungsfall

Bei einem Kundenprojekt zur Web-Anwendungsentwicklung lässt sich diese Konstellation einsetzen:

- **Public Subnetz**: Webserver / Frontend-EC2, erreichbar für Kunden aus dem Internet
- **Private Subnetz**: Datenbankserver / Backend-EC2, nur intern erreichbar

**Vorteil 1 – Sicherheit:** Sensible Daten (z.B. Kundendaten in der Datenbank) sind nicht direkt aus dem Internet erreichbar.

**Vorteil 2 – Klare Trennung:** Öffentliche und interne Ressourcen sind sauber voneinander getrennt, was Wartung, Skalierung und Kostenanalyse erleichtert.
