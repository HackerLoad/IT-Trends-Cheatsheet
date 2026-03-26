# AWS EC2 (Elastic Compute Cloud)

## Definition

EC2 ist der virtuelle Server-Dienst von AWS. Instanzen sind virtuelle Maschinen, die in einer VPC betrieben werden.

## Kernkonzepte

- **Instanz**: Eine laufende virtuelle Maschine
- **AMI (Amazon Machine Image)**: Vorlage für das Betriebssystem der Instanz
- **Key Pair**: SSH-Schlüsselpaar zur sicheren Anmeldung
- **Instance Type**: Definiert CPU, RAM und Netzwerk der Instanz (z.B. `t2.micro`)
- **Elastic IP**: Statische öffentliche IP-Adresse für eine Instanz

## Instanz starten – Wichtige Einstellungen

1. AMI wählen (z.B. Amazon Linux 2, Ubuntu)
2. Instance Type wählen (z.B. `t2.micro` für Free Tier)
3. VPC und Subnetz zuweisen
4. Sicherheitsgruppe konfigurieren
5. Key Pair erstellen oder auswählen

## SSH-Verbindung

```bash
# Berechtigungen des Schlüssels setzen (Linux/macOS)
chmod 400 schluessel.pem

# Verbindung herstellen
ssh -i "schluessel.pem" ec2-user@<öffentliche-IP-oder-DNS>

# Beispiel
ssh -i "WI24.pem" ec2-user@ec2-13-223-94-115.compute-1.amazonaws.com
```

## HTTP-Daemon (httpd) einrichten

```bash
# Apache installieren (Amazon Linux)
sudo yum install -y httpd

# Startseite setzen
echo "<html><body><h1>Hallo, ich bin eine private Instanz</h1></body></html>" | sudo tee /var/www/html/index.html

# Dienst starten und dauerhaft aktivieren
sudo systemctl start httpd
sudo systemctl enable httpd

# Status prüfen
sudo systemctl status httpd

# Lokal testen
curl localhost
```

## Sicherheitsgruppen

Sicherheitsgruppen sind zustandsbehaftete (Stateful) Firewalls auf Instanz-Ebene.

```
Eingehende Regel für private Instanz:
- Protokoll: HTTP  | Port: 80  | Quelle: 192.168.70.0/25  (nur aus dem VPC)
- Protokoll: SSH   | Port: 22  | Quelle: 192.168.70.0/25  (nur aus dem VPC)
```

Kein Eintrag für `0.0.0.0/0` → keine Erreichbarkeit aus dem Internet.

## Öffentlich vs. Privat erreichbar

| Konfiguration | Erreichbar von außen? |
|--------------|----------------------|
| Security Group erlaubt `0.0.0.0/0` + public Subnetz | Ja |
| Security Group erlaubt nur VPC-CIDR + private Subnetz | Nein |

## Wichtige AWS CLI Befehle

```bash
# Laufende Instanzen auflisten
aws ec2 describe-instances

# Instanz stoppen
aws ec2 stop-instances --instance-ids i-0123456789abcdef0

# Instanz starten
aws ec2 start-instances --instance-ids i-0123456789abcdef0
```

## Praxisbeispiel

Eine EC2-Instanz im privaten Subnetz hostet einen internen Webserver, der nur aus dem VPC heraus erreichbar ist. Der httpd-Dienst läuft dauerhaft und antwortet auf HTTP-Anfragen mit einer einfachen HTML-Seite. Von außen schlägt jeder Verbindungsversuch fehl (Timeout), da die Sicherheitsgruppe keinen externen Zugriff erlaubt.
