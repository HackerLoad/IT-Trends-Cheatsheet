# Cloud Computing – Einleitung / Überblick

## Cloud-Definition

Cloud Computing bezeichnet die Bereitstellung von IT-Ressourcen (z.B. Server, Speicher, Datenbanken, Software) über das Internet auf Abruf – nach dem Pay-per-Use-Prinzip. Der Nutzer betreibt keine eigene Hardware, sondern mietet Kapazitäten bei einem Anbieter.

### Bildliche Darstellung

```
        Nutzer A        Nutzer B        Nutzer C
           |               |               |
           +-------+-------+-------+-------+
                           |
                      [ Internet ]
                           |
               +-----------+-----------+
               |         CLOUD         |
               |  +---------+          |
               |  | Server  |          |
               |  +---------+          |
               |  | Speicher|          |
               |  +---------+          |
               |  | Software|          |
               |  +---------+          |
               +-----------------------+
```

Mehrere Nutzer greifen über das Internet auf gemeinsam genutzte, aber logisch getrennte Ressourcen zu.

---

## Argumente FÜR Cloud-Nutzung

- **Kostenersparnis**: Keine eigene Hardware kaufen oder warten; nur bezahlen, was genutzt wird
- **Skalierbarkeit**: Ressourcen können in Minuten hoch- oder herunterskaliert werden
- **Verfügbarkeit**: Hohe Uptime-Garantien durch redundante Rechenzentren (z.B. 99,99 %)
- **Ortsunabhängigkeit**: Zugriff von überall mit Internetverbindung
- **Automatische Updates**: Anbieter übernimmt Wartung und Sicherheitsupdates
- **Schnelle Bereitstellung**: Neue Server oder Dienste in Minuten verfügbar (kein Lieferweg)

---

## Argumente GEGEN Cloud-Nutzung

- **Datenschutz / Compliance**: Sensible Daten liegen bei einem Drittanbieter (DSGVO-Problematik)
- **Internetabhängigkeit**: Kein Internet = kein Zugriff auf die eigenen Systeme
- **Vendor Lock-in**: Abhängigkeit vom Anbieter; Wechsel ist aufwändig
- **Laufende Kosten**: Bei dauerhaft hoher Nutzung kann Cloud teurer sein als eigene Hardware
- **Kontrollverlust**: Kein direkter Zugriff auf die physische Infrastruktur
- **Sicherheitsrisiken**: Geteilte Infrastruktur birgt potenzielle Angriffsflächen

---

## Beispiel-Services einer Cloud (AWS)

| Kategorie | AWS-Service | Beschreibung |
|-----------|------------|--------------|
| Compute | EC2 | Virtuelle Server |
| Speicher | S3 | Objektspeicher |
| Datenbank | RDS | Managed relationale Datenbanken |
| Netzwerk | VPC | Eigenes virtuelles Netzwerk |
| Serverless | Lambda | Code ausführen ohne Server zu verwalten |
| KI/ML | SageMaker | Machine-Learning-Modelle trainieren und deployen |
| API | API Gateway | REST-APIs bereitstellen und verwalten |

---

## Praxispartner-Anwendungsfall

Ein Unternehmen (z.B. Accenture) könnte seinen internen Dokumenten-Archivierungsprozess in die Cloud verlagern. Statt eigener Server werden Dokumente in S3 gespeichert, über IAM-Rollen abgesichert und über ein CloudFront-CDN weltweit schnell ausgeliefert. Vorteil: Keine eigene Serverinfrastruktur, automatische Skalierung bei Lastspitzen, globale Verfügbarkeit.
