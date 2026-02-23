# Design Decisions

Dieses Dokument beschreibt bewusste Architektur- und Betriebsentscheidungen innerhalb meines HomeLabs.

Ziel ist es, nicht nur Konfigurationen festzuhalten, sondern Denkprozesse nachvollziehbar zu machen.

---

## 1. MikroTik als zentrale Routing- und Sicherheitsinstanz

### Entscheidung

Der MikroTik übernimmt Routing, NAT, DNS, VPN und Firewall-Funktionalität.

### Alternative

* Routing direkt über die FRITZ!Box
* Getrennte Firewall-Appliance

### Begründung

Ich wollte eine zentrale, flexibel konfigurierbare Instanz mit granularer Kontrolle über:

* Firewall-Regeln
* NAT
* VPN-Termination
* DNS-Handling

RouterOS bietet hierfür deutlich mehr Möglichkeiten als die FRITZ!Box.

### Nachteile (bewusst akzeptiert)

* Höhere Komplexität
* Größeres Fehlerpotenzial bei Fehlkonfiguration
* Mehr Eigenverantwortung im Betrieb

---

## 2. FRITZ!Box weiterhin als Gateway (kein Bridge-Mode)

### Entscheidung

Die FRITZ!Box bleibt im Standard-Gateway-Modus und terminiert die WAN-Verbindung.

### Alternative

* Bridge-Mode
* PPPoE direkt auf dem MikroTik

### Begründung

* Stabiler Betrieb
* Geringeres Risiko bei Fehlkonfiguration
* Schnelle Wiederherstellung bei Problemen

### Nachteile

* Doppelte NAT-Struktur
* Leicht erhöhte Komplexität im Routing

---

## 3. DNS über MikroTik (Forward & Cache)

### Entscheidung

Clients nutzen den MikroTik als DNS-Resolver.

### Alternative

* Direkter DNS-Zugriff auf externe Resolver
* Lokaler Pi-hole / dedizierter DNS-Server

### Begründung

* Zentrale Kontrolle
* Möglichkeit für Logging
* Potenzielle Erweiterung um Filterfunktionen

### Nachteile

* Abhängigkeit vom Router

---

## 4. VPN-Termination auf dem MikroTik

### Entscheidung

VPN wird direkt auf dem MikroTik terminiert.

### Alternative

* VPN auf der FRITZ!Box
* Externer VPN-Server (z. B. Docker-Container)

### Begründung

* Einheitliche Routing- und Firewall-Kontrolle
* Klare Trennung zwischen WAN und internem Netz

### Nachteile

* Höhere Verantwortung für Sicherheitskonfiguration

---

## 5. Strukturierte Incident-Dokumentation

### Entscheidung

Jeder relevante Vorfall wird nach festem Template dokumentiert.

### Alternative

* Keine Dokumentation
* Lose Notizen

### Begründung

* Nachhaltiges Lernen
* Nachvollziehbarkeit
* Portfolio-Aufbau

### Nachteile

* Zeitaufwand

---

## 6. Manuelle Firmware-Updates

### Entscheidung

Automatische Updates sind deaktiviert.

### Begründung

* Vermeidung ungeplanter Ausfälle
* Kontrolle über Wartungszeitpunkte

---

# Zukünftige Design-Entscheidungen

## VLAN-Segmentierung

Geplante Trennung von:

* Management
* Lab/Test
* IoT / Consumer

Ziel: Reduzierung lateraler Bewegungsmöglichkeiten und bessere Kontrolle.

---

## Monitoring-Konzept

Geplant:

* Gateway-Health-Checks
* Service-Erreichbarkeit
* Event-basierte Benachrichtigungen

---

## PPPoE direkt auf MikroTik (Evaluierung)

Langfristige Überlegung, um Double-NAT zu vermeiden und volle WAN-Kontrolle zu erhalten.

Risikoabwägung noch offen.

---

# Was würde ich heute anders machen?

* Frühzeitiger Architektur-Entwurf vor Implementierung neuer Services
* Früheres Monitoring statt reaktiver Fehlersuche
* Klare Trennung zwischen Experimentier- und Produktionsbereich
* Mehr Fokus auf Netzsegmentierung von Anfang an

---

Dieses Dokument wird fortlaufend erweitert, wenn neue Architekturentscheidungen getroffen oder bestehende angepasst werden.
