# 🌍 Externer Zugriff auf das MikroTik VPN

Diese Anleitung beschreibt, wie externe Nutzer (außerhalb deines Heimnetzwerks) auf dein MikroTik WireGuard VPN zugreifen können, um Streaming-Dienste oder andere Inhalte über deine VPN-Verbindung zu nutzen.

---

## **📌 Möglichkeiten für externe Nutzer**
Da viele Streaming-Geräte (z. B. PS4, Smart-TVs) keinen eigenen VPN-Client haben, gibt es mehrere Wege, externe Nutzer in dein VPN einzubinden:

| Gerät | Methode |
|---------|----------------|
| **PC, Laptop, Smartphone** | Direkte Verbindung über den WireGuard-Client ✅ |
| **PS4 oder Smart-TV** | Verbindung über einen VPN-Router oder SmartDNS 🛠️ |

---

## **🛠️ 1️⃣ Direkte Verbindung mit WireGuard (PC, Laptop, Smartphone)**

Diese Methode eignet sich für Geräte mit **integriertem VPN-Client**, wie **Windows, macOS, Linux, iOS, Android**.

### **📌 Schritte zur Einrichtung**

### **1.1 Neuen WireGuard-Peer auf dem MikroTik anlegen**

1. Öffne ein Terminal oder WebFig und füge den neuen Nutzer hinzu:
   ```sh
   /interface wireguard peers add allowed-address=10.10.10.3/32 interface=wireguard1 public-key="CLIENT-PUBLIC-KEY"
   ```
   - **Ersetze `CLIENT-PUBLIC-KEY`** mit dem öffentlichen Schlüssel des externen Geräts.

2. Erstelle eine **WireGuard-Konfigurationsdatei** für den Nutzer:
   ```ini
   [Interface]
   PrivateKey = CLIENT-PRIVATE-KEY
   Address = 10.10.10.3/24
   DNS = 8.8.8.8, 1.1.1.1

   [Peer]
   PublicKey = SERVER-PUBLIC-KEY
   Endpoint = DEINE-ÖFFENTLICHE-IP:13231
   AllowedIPs = 0.0.0.0/0
   PersistentKeepalive = 25
   ```
   - **`DEINE-ÖFFENTLICHE-IP`** ist die WAN-IP deiner FritzBox (oder eine DynDNS-Adresse).
   - **`SERVER-PUBLIC-KEY`** ist der öffentliche Schlüssel deines MikroTik.

3. **Den Nutzer die Konfiguration importieren lassen**
   - **Windows/macOS/Linux** → WireGuard-Client installieren und Datei importieren
   - **iOS/Android** → WireGuard-App nutzen

✅ **Nach der Verbindung sollte der externe Nutzer mit deiner VPN-IP online sein!**

---

## **🛠️ 2️⃣ PS4 oder Smart-TV über VPN-Router verbinden**
Da PS4s und Smart-TVs **keinen eigenen VPN-Client** haben, muss ein **VPN-Router** genutzt werden.

### **📌 Voraussetzungen**
- Ein **zweiter Router** mit **WireGuard-Unterstützung** (z. B. MikroTik hAP Lite, OpenWRT-Router, GL.iNet).
- Dieser Router verbindet sich per VPN mit deinem MikroTik.

### **📌 Einrichtung eines externen VPN-Routers**
1. **WireGuard-Client auf dem zweiten Router einrichten**
   - Ähnlich wie bei PC/Smartphone wird eine WireGuard-Verbindung hergestellt.
   - Beispiel für die **WireGuard-Client-Konfiguration am zweiten Router**:
     ```ini
     [Interface]
     PrivateKey = CLIENT-PRIVATE-KEY
     Address = 10.10.10.4/24
     DNS = 8.8.8.8, 1.1.1.1

     [Peer]
     PublicKey = SERVER-PUBLIC-KEY
     Endpoint = DEINE-ÖFFENTLICHE-IP:13231
     AllowedIPs = 0.0.0.0/0
     PersistentKeepalive = 25
     ```

2. **PS4 oder Smart-TV mit dem neuen VPN-Router verbinden**
   - Einfach per **WLAN oder LAN-Kabel** an den VPN-Router anschließen.
   - Fertig! Der gesamte Internetverkehr läuft jetzt über dein VPN.

✅ **PS4 oder Smart-TV glaubt nun, es wäre in deinem Heimnetzwerk.**

---

## **🛠️ 3️⃣ Alternative: SmartDNS für externe Nutzer**
Falls ein **VPN-Router nicht möglich** ist, kann ein **SmartDNS-Ansatz** genutzt werden. Hierbei wird nur der DNS-Traffic über dein Heimnetz geleitet.

### **📌 Einrichtung von SmartDNS auf MikroTik**
1. **DNS-Server auf MikroTik aktivieren**
   ```sh
   /ip dns set allow-remote-requests=yes servers=8.8.8.8,1.1.1.1
   ```

2. **Firewall-Regel für externe Anfragen hinzufügen**
   ```sh
   /ip firewall nat add chain=dstnat protocol=udp dst-port=53 action=dst-nat to-addresses=192.168.10.1 to-ports=53
   ```

3. **Externer Nutzer muss folgende DNS-Server auf der PS4 oder Smart-TV eintragen:**
   - **Primärer DNS:** `DEINE-ÖFFENTLICHE-IP`
   - **Sekundärer DNS:** `8.8.8.8`

✅ **Dadurch erkennt der Streaming-Dienst die VPN-Region, während der eigentliche Traffic über die normale Internetverbindung läuft.**

❌ **Achtung:** Diese Methode funktioniert nicht bei allen Streaming-Diensten, da einige DNS-Umgehungen blockieren.

---

## **🔎 Fehlerbehebung bei externem Zugriff**
Falls der externe Zugriff nicht funktioniert, prüfe:

✅ **Port-Forwarding in der FritzBox:**
- Stelle sicher, dass **UDP 13231** an den MikroTik weitergeleitet wird.

✅ **Firewall auf MikroTik:**
- Erlaube eingehende VPN-Verbindungen:
  ```sh
  /ip firewall filter add action=accept chain=input protocol=udp dst-port=13231
  ```

✅ **Verbindungsstatus von WireGuard:**
- Prüfe, ob externe Nutzer verbunden sind:
  ```sh
  /interface wireguard peers print
  ```
- Falls nicht, prüfe die Logs:
  ```sh
  /log print where message~"wireguard"
  ```

✅ **Externer Nutzer testet die Verbindung:**
- Öffne auf dem Gerät des Nutzers `https://whatismyipaddress.com/` und prüfe, ob die **VPN-IP** angezeigt wird.

---

## **🚀 Fazit**
- **PCs, Smartphones & Laptops nutzen WireGuard-Client.**
- **PS4 & Smart-TVs brauchen einen VPN-Router oder SmartDNS.**
- **Port-Forwarding und Firewall-Regeln müssen stimmen.**

💡 **Falls du Fragen hast oder es nicht funktioniert, überprüfe die Firewall & Logs!** 🚀
