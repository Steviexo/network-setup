# Netzwerk-Topologie (Mermaid-Diagramm)

Dieses Diagramm stellt die aktuelle High-Level-Architektur des HomeLabs dar.

---

```mermaid
flowchart TD

    ISP[(ISP / Internet)]

    FRITZBOX["FRITZ!Box\nInternet-Gateway"]

    MIKROTIK["MikroTik Router\nRouting | DNS | VPN | Firewall"]

    CLIENTS["Clients\nUbuntu Workstation\nTestgeräte\nPS4"]

    ISP --> FRITZBOX
    FRITZBOX --> MIKROTIK
    MIKROTIK --> CLIENTS
```

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
