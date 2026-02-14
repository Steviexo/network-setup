# Service-Dokumentation: MikroTik VPN

## Ziel des VPN

Das VPN dem sicheren Remote-Zugriff auf das HomeLab-Netzwerk.

Use Cases:

- Administrativer Zugriff auf interne Systeme von außerhalb
- Sicherer Zugriff auf Webinterfaces (z. B. RouterOS)
- Zugriff auf interne Dienste ohne Portfreigaben ins öffentliche Internet

## Architektur

Remote Client → Internet → FRITZ!Box → MikroTik → internes LAN
- FRITZ!Box leitet relevanten Traffic an den MikroTik weiter
- MikroTik terminiert das VPN
- Interne Ressourcen bleiben nicht direkt exponiert

Designentscheidung:
Das VPN wird auf dem MikroTik terminiert, um Routing- und Firewall-Kontrolle zentral zu halten.

## Sicherheitsüberlegungen

- Keine direkten Portfreigaben auf interne Hosts
- VPN als einziger administrativer Zugang
- Starke Authentifizierung
- Logging aktiviert

Langfristig geplant:
- Monitoring auf Login-Versuche
- Optional: Fail2Ban-ähnliche Mechanismen (falls umsetzbar)

## Konfigurationsübersicht

### 1. Interface-Setup

VPN-Interface angelegt

IP-Pool definiert

### 2. Firewall-Regeln

Input-Regeln angepasst

Forward-Regeln geprüft

### 3. NAT-Konfiguration

Masquerading korrekt konfiguriert

### 4. Benutzer- und Authentifizierung

Benutzer angelegt

Starke Credentials

## Abhängigkeiten

- Funktionierende Internetverbindung
- Korrekte Portweiterleitung in der FRITZ!Box
- Routing korrekt gesetzt

## Typische Fehlerquellen

- Falsche NAT-Regeln
- Fehlende Forward-Regeln
- Kein Hairpin-NAT bei internen Tests
- DNS-Probleme

## Verweis auf Implementierungs-Incident

Details zu aufgetretenen Problemen während der Einrichtung:
Siehe Incident-Dokument in docs/04-incidents/.

## Wartung

- Regelmäßige Überprüfung der Firewall-Regeln
- Prüfung auf Firmware-Updates (manuell)
- Test des VPN-Zugangs nach größeren Konfigurationsänderungen
