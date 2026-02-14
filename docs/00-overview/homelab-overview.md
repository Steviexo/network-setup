# HomeLab Overview

Dieses HomeLab dient als praxisorientierte Lern- und Betriebsumgebung mit Fokus auf Netzwerkadministration, Linux-Systeme und strukturierte Incident-Dokumentation.

Ziel ist es, produktionsnahe Denkweisen (Architektur, Betrieb, Monitoring, Troubleshooting) im kleinen Maßstab umzusetzen und nachvollziehbar zu dokumentieren.

---

## Zielsetzung

* Praktisches Lernen durch reale Infrastruktur
* Aufbau administrativer Routine (Routing, Firewall, VPN, DNS)
* Strukturierte Incident-Dokumentation
* Entwicklung eines technischen Portfolios

---

## Netzwerkarchitektur (High-Level)

Client-Geräte
↓
MikroTik Router (Routing, DNS, VPN)
↓
FRITZ!Box (Internet-Gateway)
↓
ISP

---

## Rollen der Komponenten

### MikroTik

* Zentrale Routing-Instanz
* DNS-Handling
* VPN-Termination
* Firewall-Regeln
* Netzsegmentierung

### FRITZ!Box

* WAN-Anbindung (DSL/Kabel)
* Internet-Gateway
* Minimale Routing-Funktion
* Firmware manuell kontrolliert (keine automatischen Updates)

### Clients

* Linux-Workstation (Ubuntu LTS)
* Weitere Testgeräte
* Spielkonsole (Testfall für Netzwerkdesign)

---

## Betriebsprinzipien

* Änderungen werden dokumentiert
* Incidents werden mit Root-Cause-Analyse festgehalten
* Layer-basiertes Troubleshooting (L2 → L3 → L7)
* Architekturentscheidungen werden begründet

---

## Dokumentationsstruktur

Das Repository ist folgendermaßen aufgebaut:

```
docs/
├── 00-overview/        → Gesamtüberblick
├── 01-architecture/    → Designentscheidungen
├── 02-services/        → System- und Service-Dokumentation
├── 03-operations/      → Betriebskonzepte
├── 04-incidents/       → Vorfälle mit RCA
└── 05-troubleshooting/ → Allgemeine Debugging-Methoden
```

---

## Langfristige Weiterentwicklung

* Monitoring für Gateway- und Service-Erreichbarkeit
* Weitere Netzsegmentierung
* Ausbau von Automatisierung
* Erweiterung der Incident-Dokumentation

---

Dieses HomeLab ist keine reine Testumgebung, sondern eine strukturierte Lern- und Betriebsplattform zur Vorbereitung auf eine Tätigkeit im Bereich Linux- und Netzwerkadministration.
