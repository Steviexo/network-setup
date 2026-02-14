# 📖 Network-setup – Netzwerk & Routing Dokumentation

![Docs](https://img.shields.io/badge/docs-structured-blue)
![Status](https://img.shields.io/badge/status-active-green)
![Focus](https://img.shields.io/badge/focus-networking%20%7C%20linux%20admin-orange)

## 📝 Ziel dieses Repositories

✅ **Dokumentation meiner Router- & Netzwerk-Setups**\
✅ **Strukturierte Anleitungen für zukünftige Anpassungen & Erweiterungen**
✅ **Hilfestellung für andere, die ähnliche Setups umsetzen möchten**

Dieses Repository wächst mit meinen Erfahrungen und ist eine zentrale Anlaufstelle für Netzwerk- und Router-Themen.

## 📂 Struktur des Repositories

```
docs/
├── 00-overview/        → Gesamtüberblick & Topologie
├── 01-architecture/    → Designentscheidungen & Begründungen
├── 02-services/        → Service-Dokumentation (z. B. VPN)
├── 03-operations/      → Betriebsstrategien (Monitoring, Updates, Backups)
├── 04-incidents/       → Konkrete Vorfälle mit Root-Cause-Analyse
└── 05-troubleshooting/ → Allgemeine Debugging-Methoden
```

Die Struktur ist bewusst so gewählt, dass ich auch Monate später noch nachvollziehen kann:

* Warum habe ich das so gebaut?
* Was läuft wo?
* Was ist wann kaputtgegangen?
* Wie habe ich es analysiert?
  
## 🔍 Incident-Dokumentation

Jeder relevante Vorfall wird strukturiert dokumentiert – inklusive Denkweg.

Typische Struktur:

* TL;DR
* Kontext
* Symptome
* Hypothesen
* Untersuchung (Layer 2 → 3 → 7)
* Technische Artefakte (CLI-Output, Logs)
* Root Cause
* Lessons Learned
* Präventive Maßnahmen

Mir geht es dabei nicht nur um „Fixen“, sondern um Verständnis.

Beispiele:

* FRITZ!Box Boot-Hang nach Firmware-Update
* MikroTik VPN Implementierungsproblem (Konzeptfehler)

---

## 🧱 Architektur – Überblick

Aktuelle High-Level-Architektur:

Client → MikroTik (Routing, DNS, VPN, Firewall) → FRITZ!Box (WAN-Gateway) → ISP

Der MikroTik ist die zentrale Routing- und Sicherheitsinstanz. Die FRITZ!Box übernimmt primär die WAN-Anbindung. Alle Clients befinden sich vollständig hinter dem MikroTik.

Detaillierte Diagramme:

* [Netzwerk-Topologie, Trennung zwischen Data-Plane und Control-Plane dokumentiert](docs/00-overview/network-topology.md)

---

## 📌 Warum dieses Repository existiert

Dieses Repository dokumentiert meine HomeLab-Netzwerk-Umgebung und dient gleichzeitig als persönliches Netzwerk- & Infrastruktur-Portfolio.

Ich nutze dieses Projekt, um:

* reale Netzwerkarchitekturen aufzubauen
* Routing-, Firewall- und VPN-Konzepte praktisch umzusetzen
* Probleme systematisch zu analysieren
* Incident-Dokumentation strukturiert zu betreiben
* meine Entwicklung im Bereich Linux- & Netzwerkadministration nachvollziehbar festzuhalten

Es ist bewusst mehr als eine Sammlung von Konfigurationsnotizen – es ist mein technisches Logbuch.

---

## 🛠 Services & Systeme

Dokumentierte Komponenten u. a.:

* MikroTik (Routing, DNS, VPN, Firewall)
* FRITZ!Box als WAN-Gateway
* Docker-Host (HP EliteDesk 800 G5 Mini)
* Self-Hosted Services (z. B. Ollama, NetBox, Portainer, NPM, paperless-ngx)

Zu jedem Service dokumentiere ich:

* Zielsetzung
* Architektur
* Sicherheitsüberlegungen
* Typische Fehlerquellen
* Wartungshinweise

## 🧠 Betriebsprinzipien

* Änderungen werden nicht nur durchgeführt, sondern dokumentiert.
* Probleme werden schichtbasiert analysiert (OSI-Denken).
* Architekturentscheidungen werden begründet.
* Sensible Daten werden nicht veröffentlicht.
* Fehler sind Lernmaterial – nicht nur Störungen.

---
## 🚀 Einstieg für Leser

Empfohlene Reihenfolge:

1. [HomeLab-Überblick](docs/00-overview/homelab-overview.md`)
2. [Netzwerk-Topologie](docs/00-overview/network-topology.md)
3. [Ein Incident aus](docs/04-incidents/)

So bekommt man sowohl die Architektur als auch meinen Denkansatz beim Troubleshooting mit.

---
## 📈 Weiterentwicklung

Geplante nächste Schritte:

* Monitoring (Gateway-Checks, Service-Verfügbarkeit)
* Weitere Netzsegmentierung (VLANs)
* Mehr Automatisierung
* Ausbau der Sicherheitsmechanismen

Dieses Repository begleitet meinen Weg in Richtung professioneller Linux- und Netzwerkadministration – mit realer Infrastruktur, echten Problemen und dokumentierten Lösungen.
