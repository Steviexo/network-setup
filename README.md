# GitHub-Einführung

Neu bei GitHub? Keine Sorge! Schauen Sie sich unsere [GitHub-Einführung](https://github.com/steviexo/github-intro/blob/main/intro.md) an, um die Grundlagen zu lernen.


# 📖 Network-setup – Netzwerk & Routing Dokumentation

Willkommen im Repository **network-setup**! Dieses Repository dient als zentrale Wissenssammlung für meine Router- und Netzwerk-Konfigurationen. Hier dokumentiere ich meine Erfahrungen und Best Practices zur Einrichtung und Optimierung meines Netzwerks.

## 📝 Ziel dieses Repositories

✅ **Dokumentation meiner Router- & Netzwerk-Setups**\
✅ **Strukturierte Anleitungen für zukünftige Anpassungen & Erweiterungen**
✅ **Hilfestellung für andere, die ähnliche Setups umsetzen möchten**

Dieses Repository wächst mit meinen Erfahrungen und ist eine zentrale Anlaufstelle für Netzwerk- und Router-Themen.

## 📂 Verzeichnisstruktur & Unterthemen

```
router-configs/
├── docs/                      # Dokumentation zu diesem Repository
│   ├── README.md              # Einführung & Übersicht
│   ├── mikrotik-vpn.md        # MikroTik VPN: Einrichtung & Grundkonfiguration
│   ├── firewall-rules.md      # Sicherheitsmaßnahmen & Firewall-Regeln
│   ├── network-segmentation.md # VLANs & Netzwerktrennung
│   ├── configuration.md       # Detaillierte MikroTik-Befehle zur VPN-Konfiguration
│   ├── external-access.md     # Anleitung für VPN-Nutzer (SmartDNS, WireGuard-Client)
│   ├── troubleshooting.md     # Häufige Probleme & Lösungen
│   ├── security.md            # Best Practices zur Absicherung (DNS-Leaks, Firewall)
│   ├── changelog.md           # Änderungen & Updates an der Konfiguration (Optional)
├── templates/                  
│   ├── mikrotik-config.rsc     # Beispiel-Konfigurationsskript für MikroTik
│   ├── vpn-setup-guide.md      # Schritt-für-Schritt-Anleitung
├── scripts/                    # Automatisierung & Skripte
│   ├── auto-vpn-reconnect.rsc  # MikroTik-Skript für automatische VPN-Neuverbindung
│   ├── firewall-setup.rsc      # Automatisches Setup von Firewall-Regeln
├── images/                     # Netzwerkdiagramme & Screenshots
│   ├── vpn-topology.png        # Schema der VPN-Verbindung
│   ├── firewall-rules-diagram.png # Visualisierung der Firewall-Regeln
└── README.md                   # Haupt-README des Repos
```

### 🔹 **Router & Netzwerk-Setup**

- **[MikroTik VPN](docs/mikrotik-vpn.md)**: VPN zum Streamen einrichten
- **[Firewall-Regeln](docs/firewall-rules.md)**: Sicherheitsregeln & Schutzmaßnahmen
- **[Fritzbox Bridge Mode](docs/fritzbox-bridge.md)**: Nutzung der Fritzbox als Modem
- **[Netzwerk-Segmentierung](docs/network-segmentation.md)**: VLANs & Subnetting
- **[Externer Zugriff](docs/external-access.md)**: VPN-Nutzung auf verschiedenen Geräten
- **[Absicherung & Security](docs/security.md)**: Best Practices für eine sichere Konfiguration

### 🚧 **In Arbeit / Geplante Inhalte**

- **Optimierung der Netzwerk-Performance**
- **DNS- & DHCP-Server auf MikroTik**
- **Dynamische Routen & Load Balancing**
- **Erweiterte Firewall-Filter & Portweiterleitungen**

---

## 🖥 Technische Ausstattung

Dieses Repository basiert auf folgendem Setup:

- **Haupt-Router**: FritzBox 7530 (mit Internetzugang, DHCP-Server)
- **Sekundär-Router**: MikroTik hAP ax³ (per LAN verbunden, für VPN & Netzwerksteuerung)
- **VPN-Dienst**: WireGuard
- **Betriebssysteme**: Ubuntu 24.04 LTS, Windows, macOS
- **Netzwerktopologie**: /

Falls du ein anderes Setup verwendest, können einige Konfigurationsschritte abweichen.

---

## 📝 Warum dieses Repository?

Ich nutze GitHub primär als **persönliches Dokumentations- und Wissensmanagement-Tool**. Dieses Repository hilft mir dabei, …

- meine **Netzwerk- & Router-Konfigurationen** zu dokumentieren
- anderen (auch ohne tiefes technisches Wissen) Lösungen für ähnliche Probleme bereitzustellen
- meine **Erfahrungen als zukünftiger Systemadministrator** sichtbar zu machen

Falls du Fragen hast oder Feedback geben möchtest, freue ich mich über deine Nachricht! 😊

---

## 🚀 Verwendung

Hier findest du alle wichtigen Informationen zur Nutzung dieses Repositories.

- **Installation & Einrichtung**: Lies die [MikroTik VPN-Anleitung](docs/mikrotik-vpn.md) für erste Schritte.
- **Netzwerk-Segmentierung**: Mehr zur Trennung von Netzwerken findest du in [network-segmentation.md](docs/network-segmentation.md).
- **Firewall & Sicherheit**: Schutzmaßnahmen in [firewall-rules.md](docs/firewall-rules.md).

## 🤝 Mitwirken

Falls du Änderungen oder Verbesserungen beisteuern möchtest, lies die [CONTRIBUTING.md](CONTRIBUTING.md).

## 🔗 Weiterführende Informationen

- [MikroTik Dokumentation](https://wiki.mikrotik.com)
- [OpenVPN Offizielle Seite](https://openvpn.net)
- [Projekt-Template](https://github.com/steviexo/project-template)

---

✍ **Letzte Aktualisierung:**

