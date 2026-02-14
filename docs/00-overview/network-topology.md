# Netzwerk-Topologie (Mermaid-Diagramm)

Dieses Diagramm stellt die aktuelle High-Level-Architektur des HomeLabs dar.

---

```mermaid
flowchart TD

    %% =====================
    %% Extern / WAN
    %% =====================
    ISP[(ISP / Internet)]

    %% FritzBox als Gateway (WAN)
    FRITZBOX["FRITZ!Box 7530
WAN-Gateway
LAN: 192.168.178.1/24"]

    %% MikroTik: WAN im FritzBox-LAN, LAN/WLAN für Clients
    MIKROTIK["MikroTik
WAN: 192.168.178.49/24
GW: 192.168.178.1
LAN/WLAN: 192.168.88.1/24
DHCP: 192.168.88.100-199
DNS: MikroTik (Forward/Cache)"]

    %% Clients und Services
    UBUNTU["Ubuntu Workstation
IP via DHCP (192.168.88.x)"]
    LAPTOP["Laptop / Testgeräte
IP via DHCP (192.168.88.x)"]
    PS4["PS4 / IoT / Consumer Devices
IP via DHCP (192.168.88.x)"]

    DOCKER["HP EliteDesk 800 G5 Mini (Docker Host)
Statische IP (empfohlen): 192.168.88.10
Services: Ollama, NetBox, Portainer, NPM, paperless-ngx"]

    %% Datenpfade
    ISP -->|"Public IP (dynamisch)"| FRITZBOX
    FRITZBOX -->|"LAN-Uplink"| MIKROTIK

    %% NAT / Routing Hinweise
    MIKROTIK -->|"Routing + NAT (Masquerade)"| UBUNTU
    MIKROTIK --> LAPTOP
    MIKROTIK --> PS4
    MIKROTIK --> DOCKER

    %% Notiz: Clients sprechen immer über MikroTik
    UBUNTU -.->|"Default Route: 192.168.88.1"| MIKROTIK
    LAPTOP -.->|"Default Route: 192.168.88.1"| MIKROTIK
    PS4 -.->|"Default Route: 192.168.88.1"| MIKROTIK
    DOCKER -.->|"Default Route: 192.168.88.1"| MIKROTIK
```

> **Anonymisierungshinweis (Repo/Public):** Öffentliche IP/Anbieter-Zugangsdaten nie dokumentieren. Interne RFC1918-Netze (z. B. 192.168.x.x) sind normalerweise unkritisch, aber Hostnamen/Ports/öffentliche DynDNS-Namen ggf. als Platzhalter schreiben.

---

## Architekturprinzip

* Die FRITZ!Box übernimmt ausschließlich die WAN-Anbindung.
* Der MikroTik ist zentrale Routing- und Sicherheitsinstanz.
* Clients befinden sich vollständig hinter dem MikroTik.

---

## Optional: Erweiterung (zukünftig geplant)

Beispiel für mögliche Erweiterungen:

```mermaid
flowchart TD

    ISP[(ISP / Internet)]

    FRITZBOX["FRITZ!Box"]

    MIKROTIK["MikroTik Router"]

    VLAN1["LAN Segment"]
    VLAN2["Lab / Test Segment"]

    SERVER["Docker Host\nHP EliteDesk"]

    ISP --> FRITZBOX
    FRITZBOX --> MIKROTIK
    MIKROTIK --> VLAN1
    MIKROTIK --> VLAN2
    VLAN2 --> SERVER
```

Dieses erweiterte Diagramm kann genutzt werden, sobald weitere Netzsegmente oder Services hinzukommen.
