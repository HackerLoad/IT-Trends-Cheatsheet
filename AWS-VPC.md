# AWS VPC (Virtual Private Cloud)

## Doppeldeutigkeit des Begriffs VPC

| Begriff | Bedeutung |
|--------|-----------|
| VPC (AWS) | Virtual Private Cloud – logisch isoliertes, virtuelles Netzwerk innerhalb der AWS Public Cloud |
| VPC (allgemein) | Wird manchmal auch synonym für VPN (Virtual Private Network) verwendet |

## Definition: Virtual Private Cloud (AWS)

Eine VPC ist ein virtuelles Netzwerk, das einem klassischen On-Premise-Netzwerk sehr ähnlich ist – jedoch vollständig in der Cloud betrieben wird. Sie bietet vollständige Kontrolle über IP-Adressbereiche, Subnetze, Routing und Sicherheitsregeln.

## Kernkomponenten

- **Subnetz**: Unterteilung des VPC-Netzwerks; kann public oder private sein
- **Internet Gateway**: Ermöglicht den Internetzugang für public Subnetze
- **NAT Gateway**: Erlaubt privaten Subnetzen ausgehenden Internetzugriff, ohne direkt erreichbar zu sein
- **Routing-Tabelle**: Steuert, wohin Netzwerkverkehr weitergeleitet wird
- **Sicherheitsgruppe**: Virtuelle Firewall auf EC2-Instanz-Ebene (Stateful)
- **Netzwerk-ACL**: Firewall auf Subnetz-Ebene (Stateless)

## Typische Netzwerkaufteilung

```
VPC: 192.168.70.0/25  (126 nutzbare Hosts)
├── sub1_pub   → public Subnetz  (z.B. 192.168.70.0/27)
├── sub2_priv  → private Subnetz (z.B. 192.168.70.32/27)
└── sub3       → weiteres Subnetz
```

## CIDR-Notation kurz erklärt

```
192.168.70.0/25
              ^--- 25 Bits für Netzwerkanteil → 7 Bits für Hosts → 2^7 - 2 = 126 Hosts
```

| CIDR | Hosts (nutzbar) |
|------|----------------|
| /24  | 254 |
| /25  | 126 |
| /26  | 62 |
| /27  | 30 |
| /28  | 14 |

## Public vs. Private Subnetz

| Eigenschaft | Public Subnetz | Private Subnetz |
|-------------|---------------|-----------------|
| Internetzugang | Direkt über Internet Gateway | Nur über NAT Gateway (ausgehend) |
| Erreichbarkeit | Von außen erreichbar | Nicht direkt von außen erreichbar |
| Typischer Inhalt | Webserver, Load Balancer | Datenbanken, interne Logik |

## Sicherheitsgruppen

Sicherheitsgruppen steuern ein- und ausgehenden Datenverkehr auf Instanz-Ebene.

Beispiel – Sicherheitsgruppe für eine private Instanz (nur Zugriff aus dem VPC):

```
Eingehende Regeln:
- HTTP (Port 80)  | Quelle: 192.168.70.0/25
- SSH  (Port 22)  | Quelle: 192.168.70.0/25
```

Kein Eintrag für Zugriff aus dem Internet → Instanz ist von außen nicht erreichbar.

## AWS CLI – VPC & EC2

```bash
# SSH-Verbindung zu einer EC2-Instanz (mit Key-Pair)
ssh -i "schluessel.pem" ec2-user@ec2-xx-xx-xx-xx.compute-1.amazonaws.com

# HTTP-Anfrage von innerhalb des VPC testen
curl http://192.168.70.xx

# Lokalen HTTP-Daemon testen
curl localhost
```

## Vorteile einer Public/Private-Konstellation

- Sicherheit: Sensible Daten (z.B. Datenbanken) sind nicht direkt aus dem Internet erreichbar
- Datenschutz: Klare Trennung von öffentlichen und internen Ressourcen
- Skalierbarkeit: Subnetze und Instanzen können unabhängig voneinander erweitert werden

## Nachteil

Eine VPC-Konfiguration allein reicht nicht für vollständige Sicherheit. Interne Bedrohungen (z.B. Mitarbeiter mit Zugriffsrechten, Portweiterleitungen) müssen zusätzlich durch IAM-Rollen, Logging und Monitoring abgesichert werden.

## Praxisbeispiel

Bei einem Web-Anwendungsprojekt hostet das public Subnetz das Kunden-Frontend (Webserver), während das private Subnetz die Datenbank mit sensiblen Nutzerdaten enthält. Die Datenbank ist nur über den Webserver im public Subnetz erreichbar – nicht direkt aus dem Internet.
