# AWS S3 (Simple Storage Service)

## Definition

S3 ist ein objektbasierter Cloud-Speicherdienst von AWS. Dateien (Objekte) werden in sogenannten **Buckets** gespeichert.

## Kernkonzepte

- **Bucket**: Container für Objekte (vergleichbar mit einem Ordner auf oberster Ebene)
- **Objekt**: Die gespeicherte Datei inkl. Metadaten
- **Key**: Eindeutiger Name des Objekts innerhalb eines Buckets
- **Region**: Geografischer Standort des Buckets

## Bucket-Inhaltsverzeichnis verbergen

Standardmäßig kann man den Inhalt eines Buckets nicht auflisten, wenn kein `s3:ListBucket`-Recht vergeben wurde. Das erhöht die Sicherheit: Nutzer müssen den genauen Dateinamen kennen, um eine Datei herunterzuladen.

## Bucket Policy (JSON)

Zugriff wird über Bucket Policies gesteuert. Beispiel: Nur Lesezugriff auf ein konkretes Objekt erlauben.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mein-bucket/*"
    }
  ]
}
```

> Ohne `s3:ListBucket` kann niemand den Inhalt des Buckets auflisten – nur wer den Dateinamen kennt, kann herunterladen.

## AWS CLI – Wichtige Befehle

```bash
# Buckets auflisten
aws s3 ls

# Inhalt eines Buckets auflisten
aws s3 ls s3://mein-bucket/

# Datei herunterladen
aws s3 cp s3://mein-bucket/datei.pdf ./lokaler-pfad/

# Datei hochladen
aws s3 cp ./lokale-datei.pdf s3://mein-bucket/

# Datei löschen
aws s3 rm s3://mein-bucket/datei.pdf
```

## Vorteile

- Nahezu unbegrenzte Skalierbarkeit
- Hohe Verfügbarkeit und Durabilität (99,999999999 % – "11 Neunen")
- Fein granulierbare Zugriffssteuerung über Bucket Policies und IAM
- Günstige Kosten (Pay-per-Use)

## Sicherheit

- Bucket Policies: Steuern den Zugriff auf Bucket-Ebene
- IAM-Rollen/Policies: Steuern den Zugriff auf Nutzer-/Rollenebene
- Block Public Access: Verhindert versehentlich öffentliche Buckets
- Verschlüsselung: SSE-S3, SSE-KMS oder clientseitig

## Praxisbeispiel

Ein Unternehmen speichert interne Dokumente in einem privaten S3-Bucket. Mitarbeiter können Dateien nur herunterladen, wenn sie den exakten Dateinamen kennen – das Inhaltsverzeichnis bleibt verborgen. Zugriff erfolgt nur über authentifizierte IAM-Rollen.
