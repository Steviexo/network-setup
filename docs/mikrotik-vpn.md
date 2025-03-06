# MikroTik VPN-Setup für Streaming

Diese Anleitung beschreibt die Einrichtung eines **WireGuard VPN-Servers** auf einem MikroTik-Router (hap ax3), um Streaming über eine sichere Verbindung zu ermöglichen. Das Setup erlaubt sowohl lokale als auch externe Geräte, den VPN-Server zu nutzen, um Geo-Restriktionen zu umgehen oder eine sichere Verbindung zu einem zentralen Netzwerk zu haben.

## 📌 **Ziel des Setups**

- **Primär:** PlayStation 4 (PS4) und andere Streaming-Geräte im lokalen Netzwerk (WLAN) sollen über das VPN streamen können.
- **Sekundär:** Externe Geräte sollen optional ebenfalls über das VPN streamen können.

## 🛠 **Hardware & Voraussetzungen**

- **Router**: MikroTik hap ax3
- **Firmware**: RouterOS v7+
- **VPN-Protokoll**: WireGuard
- **Internet-Zugang über eine FritzBox (LAN-Anbindung)**

## 🚀 **Schritt-für-Schritt Einrichtung**

### **1️⃣ WireGuard-VPN auf MikroTik einrichten**

1. **WireGuard-Interface erstellen**

   ```sh
   /interface/wireguard add listen-port=13231 name=wireguard1
   ```

   - Den generierten öffentlichen Schlüssel notieren.

2. **IP-Adresse für das WireGuard Interface setzen**

   ```sh
   /ip/address add address=10.10.10.1/24 interface=wireguard1 network=10.10.10.0
   ```

3. **Port-Forwarding in der FritzBox einrichten**

   - **UDP-Port 13231** an die IP des MikroTik weiterleiten.

4. **Firewall-Regeln für WireGuard erstellen**

   ```sh
   /ip/firewall/filter add action=accept chain=input dst-port=13231 protocol=udp
   /ip/firewall/filter add action=accept chain=forward in-interface=wireguard1
   /ip/firewall/filter add action=accept chain=forward out-interface=wireguard1
   ```

5. **NAT für VPN-Clients aktivieren**

   ```sh
   /ip/firewall/nat add action=masquerade chain=srcnat out-interface=ether1
   /ip/firewall/nat add action=masquerade chain=srcnat src-address=10.10.10.0/24
   ```

### **2️⃣ Spezielles WLAN für VPN-Traffic erstellen**

1. **Neues WLAN-Interface für VPN-Traffic einrichten**

   ```sh
   /interface wireless add name=wlan-for-streaming-vpn ssid=LAN_solo mode=ap
   ```

   - Frequenz: 2412 MHz (Kanal 1) oder 2462 MHz (Kanal 11)
   - Band: 2GHz-G oder 2GHz-N
   - Security Profile: WPA2-PSK mit starker Passphrase

2. **IP-Adresse für dieses Netzwerk setzen**

   ```sh
   /ip/address add address=192.168.10.1/24 interface=wlan-for-streaming-vpn
   ```

3. **DHCP-Server für das VPN-WLAN einrichten**

   ```sh
   /ip pool add name=streaming-vpn-pool ranges=192.168.10.10-192.168.10.250
   /ip dhcp-server add name=dhcp-streaming-vpn interface=wlan-for-streaming-vpn address-pool=streaming-vpn-pool lease-time=10m
   ```

### **3️⃣ Routing und Firewall für VPN-WLAN anpassen**

1. **Routing für dieses Netzwerk über VPN**

   ```sh
   /ip route add dst-address=0.0.0.0/0 gateway=wireguard1 routing-mark=vpn-traffic
   ```

2. **Traffic-Markierung für das VPN-Netzwerk erstellen**

   ```sh
   /ip firewall mangle add action=mark-routing chain=prerouting src-address=192.168.10.0/24 new-routing-mark=vpn-traffic
   ```

3. **Firewall-Regeln zur Absicherung**

   ```sh
   /ip firewall filter add action=accept chain=forward connection-state=established,related
   /ip firewall filter add action=drop chain=forward src-address=192.168.10.0/24 dst-address=192.168.88.0/24
   ```

4. **NAT für VPN-Clients aktivieren**

   ```sh
   /ip firewall nat add chain=srcnat src-address=192.168.10.0/24 action=masquerade
   ```

### **4️⃣ Testen der Verbindung**

1. **PS4 & Smartphone mit **``** WLAN verbinden**
2. **Überprüfen, ob Geräte eine IP-Adresse aus **``** erhalten**
3. **Streaming-Dienste testen** (z. B. Netflix, Prime Video)

---

## **📌 Erweiterung: Externe Geräte über VPN verbinden**

Falls externe Personen außerhalb des Heimnetzwerks auf den VPN zugreifen möchten:

- **PS4 & Smart-TV** → Ein eigener **VPN-Router (MikroTik oder OpenWRT) wird benötigt.**
- **PC, Smartphone, Laptop** → Direkt per **WireGuard-Client** verbinden.

### **Externe WireGuard-Peers hinzufügen**

```sh
/interface wireguard/peers add allowed-address=10.10.10.3/32 interface=wireguard1 public-key="CLIENT-PUBLIC-KEY"
```

Weitere Details zur externen Einrichtung: **In separater Dokumentation ergänzen.**

---

## **✅ Fazit**

- VPN-Server auf MikroTik eingerichtet
- Eigenes WLAN für Streaming über VPN
- PS4 & andere Geräte erfolgreich eingebunden
- Option zur Erweiterung für externe Nutzer

🚀 **Jetzt ist dein Heimnetzwerk bereit für sicheres & uneingeschränktes Streaming!**
