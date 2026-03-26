# Cloud Computing – REST-Schnittstellen

## REST-Schnittstelle auf der Kommandozeile bedienen

```bash
# Einfacher GET-Request mit curl
curl https://api.beispiel.de/v1/ressource

# Mit Query-Parameter
curl https://api.beispiel.de/v1/ressource?ID=abc123

# Mit Ausgabe in Datei
curl https://api.beispiel.de/v1/ressource -o ausgabe.txt

# POST-Request mit JSON-Body
curl -X POST https://api.beispiel.de/v1/ressource \
  -H "Content-Type: application/json" \
  -d '{"name": "test"}'
```

---

## Aufbau einer REST-Schnittstelle

```
https://api.beispiel.de/v1/nutzer/42?format=json
^^^^^   ^^^^^^^^^^^^^^^^ ^^ ^^^^^^ ^^ ^^^^^^^^^^^
Schema  Domain/Host      Ver Ressource ID  Parameter
```

| Bestandteil | Beispiel | Bedeutung |
|-------------|---------|-----------|
| Schema | `https://` | Übertragungsprotokoll |
| Host | `api.beispiel.de` | Server-Adresse |
| Pfad/Ressource | `/v1/nutzer/42` | Angefragte Ressource |
| Query-Parameter | `?format=json` | Zusätzliche Filteroptionen |

### HTTP-Methoden

| Methode | Bedeutung | Beispiel |
|---------|-----------|---------|
| GET | Ressource lesen | Nutzer abrufen |
| POST | Ressource erstellen | Neuen Nutzer anlegen |
| PUT | Ressource ersetzen | Nutzerdaten komplett überschreiben |
| PATCH | Ressource teilweise ändern | Nur E-Mail-Adresse aktualisieren |
| DELETE | Ressource löschen | Nutzer löschen |

### HTTP-Statuscodes

| Code | Bedeutung |
|------|-----------|
| 200 | OK – Anfrage erfolgreich |
| 201 | Created – Ressource erstellt |
| 400 | Bad Request – fehlerhafte Anfrage |
| 401 | Unauthorized – nicht authentifiziert |
| 403 | Forbidden – keine Berechtigung |
| 404 | Not Found – Ressource nicht gefunden |
| 500 | Internal Server Error – Serverfehler |

---

## Wesentliche Eigenschaften und Vorteile von REST

| Eigenschaft | Beschreibung |
|-------------|-------------|
| Stateless | Server speichert keinen Zustand zwischen Anfragen; jede Anfrage ist vollständig in sich |
| Einheitliche Schnittstelle | Standardisierte HTTP-Methoden, unabhängig vom Anbieter |
| Client-Server-Trennung | Frontend und Backend sind entkoppelt und unabhängig voneinander entwickelbar |
| Ressourcenbasiert | Alles wird als Ressource modelliert, ansprechbar über eine eindeutige URL |
| Plattformunabhängig | Jeder HTTP-fähige Client (Browser, App, Skript) kann eine REST-API nutzen |
| Cacheability | Antworten können gecacht werden, was die Performance verbessert |

---

## Microservice-Architektur vs. Service-Orientierte Architektur (SOA)

| Merkmal | SOA | Microservices |
|---------|-----|--------------|
| Servicegröße | Groß, viele Funktionen pro Service | Klein, ein Service = eine Funktion |
| Kommunikation | Oft über einen zentralen ESB (Enterprise Service Bus) | Direkt über leichtgewichtige APIs (REST, gRPC) |
| Deployment | Oft gemeinsam deployt | Jeder Service wird unabhängig deployt |
| Technologie | Oft einheitlich | Jeder Service kann eigene Technologie nutzen |
| Skalierung | Gesamte Anwendung skalieren | Einzelne Services gezielt skalieren |
| Typisches Format | SOAP / XML | REST / JSON |

**Merksatz:** SOA = große Bausteine mit zentralem Bus. Microservices = viele kleine, unabhängige Dienste.

---

## Beispiel: Anwendung mit Microservice-Architektur

### Online-Shop

```
          [ Browser / Mobile App ]
                    |
            [ API Gateway ]
           /       |        \
    [Nutzer-    [Produkt-   [Bestell-
     Service]    Service]    Service]
         |           |           |
     [User-DB]  [Produkt-DB] [Order-DB]
```

Jeder Service ist unabhängig, hat eine eigene Datenbank und kommuniziert über REST-Schnittstellen.

| Service | REST-Endpunkt (Beispiel) | Aufgabe |
|---------|--------------------------|---------|
| Nutzer-Service | `GET /api/nutzer/42` | Nutzerdaten abrufen |
| Produkt-Service | `GET /api/produkte?kategorie=elektronik` | Produkte filtern |
| Bestell-Service | `POST /api/bestellungen` | Neue Bestellung anlegen |
