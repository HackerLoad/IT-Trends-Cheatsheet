# Cloud Computing – AWS CLI / Preise / Service-Modelle

## Datei aus S3-Bucket herunterladen (CLI)

```bash
# Einzelne Datei herunterladen
aws s3 cp s3://mein-bucket/datei.pdf ./

# In einen bestimmten Ordner herunterladen
aws s3 cp s3://mein-bucket/datei.pdf "%USERPROFILE%\Downloads\"   # Windows
aws s3 cp s3://mein-bucket/datei.pdf ~/Downloads/                 # Linux/macOS
```

---

## Skript: Datei aus Bucket herunterladen

Das Skript erhält zwei Parameter: Bucketname und Dateiname.

### Windows (Batch / .bat)

```batch
@echo off
REM Parameter: %1 = Bucketname, %2 = Dateiname
set BUCKET=%1
set DATEI=%2

aws s3 cp s3://%BUCKET%/%DATEI% "%USERPROFILE%\Downloads\%DATEI%"

echo Download abgeschlossen: %USERPROFILE%\Downloads\%DATEI%
```

Aufruf:
```bash
download.bat mein-bucket datei.pdf
```

### Linux/macOS (Bash / .sh)

```bash
#!/bin/bash
# Parameter: $1 = Bucketname, $2 = Dateiname
BUCKET=$1
DATEI=$2

aws s3 cp s3://$BUCKET/$DATEI ~/Downloads/$DATEI

echo "Download abgeschlossen: ~/Downloads/$DATEI"
```

Aufruf:
```bash
bash download.sh mein-bucket datei.pdf
```

---

## AWS-Preise ermitteln

Preise lassen sich über den **AWS Pricing Calculator** ermitteln: [calculator.aws](https://calculator.aws)

### Beispielrechnung: S3 Glacier Deep Archive (Region Frankfurt / eu-central-1)

| Posten | Menge | Preis pro GB | Monatliche Kosten |
|--------|-------|-------------|-------------------|
| Speicher (dauerhaft) | 2.048 GB (2 TB) | ~0,00181 USD/GB | ~3,72 USD |
| Upload (PUT-Requests) | 100 GB | ~0,0038 USD/GB | ~0,38 USD |
| Retrieval (Standard) | – | ~0,0024 USD/GB | ggf. ~0,24 USD |

> Preise ändern sich – immer im offiziellen AWS Pricing Calculator nachschlagen.

### Wichtige Hinweise zu AWS-Preisen

- Preise sind **regionabhängig** (Frankfurt meist etwas teurer als US-East)
- **Free Tier**: Bestimmte Services sind im ersten Jahr kostenfrei bis zu einem Limit
- **Pay-per-Use**: Es wird nur abgerechnet, was tatsächlich genutzt wird
- Datenübertragung **ins** Internet (Egress) kostet extra; Daten **in** AWS laden (Ingress) ist meist kostenlos

---

## Die drei Cloud-Service-Modelle

### Überblick

```
                    IaaS          PaaS          SaaS
Anwendung           Nutzer        Nutzer        Anbieter
Runtime             Nutzer        Anbieter      Anbieter
Middleware          Nutzer        Anbieter      Anbieter
Betriebssystem      Nutzer        Anbieter      Anbieter
Virtualisierung     Anbieter      Anbieter      Anbieter
Server/Hardware     Anbieter      Anbieter      Anbieter
```

### IaaS – Infrastructure as a Service

**Definition:** Der Anbieter stellt die grundlegende IT-Infrastruktur bereit (Server, Speicher, Netzwerk). Der Nutzer verwaltet Betriebssystem und alles darüber selbst.

**Beispiele:** AWS EC2, AWS S3, Google Compute Engine, Microsoft Azure VMs

**Vorteil:** Maximale Kontrolle und Flexibilität

---

### PaaS – Platform as a Service

**Definition:** Der Anbieter stellt eine Plattform zur Entwicklung und Ausführung von Anwendungen bereit. Betriebssystem und Middleware werden vom Anbieter verwaltet.

**Beispiele:** AWS Elastic Beanstalk, Google App Engine, Heroku

**Vorteil:** Entwickler konzentrieren sich auf Code, nicht auf Infrastruktur

---

### SaaS – Software as a Service

**Definition:** Eine fertige Softwareanwendung wird über das Internet bereitgestellt. Der Nutzer braucht nichts zu installieren oder zu verwalten.

**Beispiele:** Microsoft 365, Google Workspace, Salesforce, Zoom

**Vorteil:** Sofort nutzbar, keine Wartung, überall erreichbar

---

### Vergleichstabelle

| Merkmal | IaaS | PaaS | SaaS |
|---------|------|------|------|
| Kontrolle | Hoch | Mittel | Gering |
| Verwaltungsaufwand | Hoch | Mittel | Gering |
| Flexibilität | Hoch | Mittel | Gering |
| Zielgruppe | Administratoren | Entwickler | Endnutzer |
| Beispiel (AWS) | EC2 | Elastic Beanstalk | WorkMail |
