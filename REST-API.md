# REST-Schnittstellen (REST API)

## Definition

REST (Representational State Transfer) ist ein Architekturstil für verteilte Systeme. Eine REST-API stellt Ressourcen über HTTP bereit, die von Clients angesprochen werden können.

## Kernprinzipien

- **Stateless**: Der Server speichert keinen Client-Zustand zwischen Anfragen
- **Ressourcenbasiert**: Alles wird als Ressource modelliert, ansprechbar über eine URL
- **Einheitliche Schnittstelle**: Standardisierte HTTP-Methoden
- **Client-Server-Trennung**: Frontend und Backend sind voneinander entkoppelt

## HTTP-Methoden

| Methode | Bedeutung |
|--------|-----------|
| GET | Ressource abrufen |
| POST | Ressource erstellen |
| PUT | Ressource ersetzen |
| PATCH | Ressource teilweise aktualisieren |
| DELETE | Ressource löschen |

## URL-Aufbau

```
https://api.beispiel.de/v1/ressource?parameter=wert
```

- **Base URL**: `https://api.beispiel.de`
- **Version**: `/v1/`
- **Ressource**: `ressource`
- **Query-Parameter**: `?parameter=wert`

## Antwortformate

REST-APIs liefern häufig JSON zurück, können aber auch plain-Text oder XML zurückgeben.

```json
{
  "id": 1,
  "name": "Beispiel",
  "status": "aktiv"
}
```

## Vorteile

- Plattformunabhängig (funktioniert mit jedem HTTP-fähigen Client)
- Skalierbar durch Zustandslosigkeit
- Einfache Integration in Web-Frontends, Mobile-Apps und interne Tools
- Weit verbreitet, gut dokumentiert

## Praxisbeispiel (Accenture / Unternehmen)

REST-APIs werden eingesetzt, um Microservices bereitzustellen, auf die Web-Frontends oder interne Tools zugreifen. Jeder Service ist unabhängig ansprechbar und kann separat skaliert werden.

## AWS API Gateway – Beispielaufruf

```bash
curl https://abc123.execute-api.us-east-1.amazonaws.com/stage/ressource?ID=xyz
```

Mit AWS API Gateway lassen sich REST-Endpunkte serverlos bereitstellen, die z.B. Lambda-Funktionen aufrufen.
