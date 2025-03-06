# 📌 Troubleshooting: MikroTik VPN-Setup für Streaming

Dieses Dokument hilft bei der Fehlerbehebung, falls das **MikroTik WireGuard VPN** nicht wie erwartet funktioniert.

## **🚨 Allgemeine Diagnose**
1. **Neustart hilft oft** – Teste zuerst:
   ```sh
   /system reboot
   ```
2. **Logs überprüfen**:
   ```sh
   /log print where message~"dhcp|wireguard|firewall"
   ```
3. **Netzwerk-Status checken**:
   ```sh
   /ip address print
   /interface print
   /interface wireguard print
   ```

---

## **❌ Problem: VPN-WLAN wird nicht gefunden**
### 🔎 **Mögliche Ursachen & Lösungen**
✅ **1. Falsche Frequenz oder Band-Einstellung**
- In **WebFig → Wireless → Interfaces**:
  - Setze **Band** auf `2GHz-G` oder `2GHz-N`
  - Ändere **Frequenz** auf `2412 MHz` (Kanal 1) oder `2462 MHz` (Kanal 11)

✅ **2. SSID wird nicht gesendet**
- In **WebFig → Wireless → WLAN-Interface**:
  - **„Hide SSID“ muss deaktiviert sein**

✅ **3. WLAN an falschem Master-Interface**
- Falls das WLAN **wifi1** als Master hat, ändere es auf **wifi2**:
  ```sh
  /interface wireless set wlan-for-streaming-vpn master-interface=wifi2
  ```

✅ **4. Router-Software aktualisieren**
- **System → Packages → Check for Updates**

---

## **❌ Problem: Gerät kann sich nicht mit VPN-WLAN verbinden**
### 🔎 **Mögliche Ursachen & Lösungen**
✅ **1. DHCP-Server funktioniert nicht**
- Prüfe unter **IP → DHCP Server**, ob `dhcp-streaming-vpn` läuft
- Falls invalid:
  ```sh
  /ip dhcp-server enable dhcp-streaming-vpn
  ```
- Falls weiterhin invalid:
  ```sh
  /ip dhcp-server remove dhcp-streaming-vpn
  /ip dhcp-server add name=dhcp-streaming-vpn interface=wlan-for-streaming-vpn address-pool=streaming-vpn-pool authoritative=yes disabled=no
  ```

✅ **2. Firewall blockiert DHCP**
- Erstelle folgende Regel:
  ```sh
  /ip firewall filter add chain=input action=accept protocol=udp dst-port=67-68 src-address=192.168.10.0/24
  ```

✅ **3. IP-Adresse nicht korrekt gesetzt**
- Überprüfe:
  ```sh
  /ip address print
  ```
- Falls `192.168.10.1/24` nicht auf `wlan-for-streaming-vpn` gesetzt ist:
  ```sh
  /ip address add address=192.168.10.1/24 interface=wlan-for-streaming-vpn
  ```

---

## **❌ Problem: Kein Internet trotz VPN-Verbindung**
### 🔎 **Mögliche Ursachen & Lösungen**
✅ **1. NAT fehlt für VPN-Traffic**
- Prüfe unter **IP → Firewall → NAT**, ob folgende Regel existiert:
  ```sh
  /ip firewall nat add chain=srcnat src-address=192.168.10.0/24 action=masquerade out-interface=wireguard1
  ```

✅ **2. Routing falsch konfiguriert**
- Prüfe:
  ```sh
  /ip route print where dst-address=0.0.0.0/0
  ```
- Falls das VPN-WLAN nicht über WireGuard geleitet wird:
  ```sh
  /ip route add dst-address=0.0.0.0/0 gateway=wireguard1 routing-mark=vpn-traffic
  ```

✅ **3. DNS falsch konfiguriert**
- Teste, ob DNS funktioniert:
  ```sh
  /ping 8.8.8.8
  /ping google.com
  ```
- Falls DNS nicht funktioniert:
  ```sh
  /ip dns set allow-remote-requests=yes servers=8.8.8.8,1.1.1.1
  ```

---

## **❌ Problem: Externe Nutzer können nicht auf das VPN zugreifen**
### 🔎 **Mögliche Ursachen & Lösungen**
✅ **1. Port-Forwarding in FritzBox fehlt**
- Stelle sicher, dass UDP **Port 13231** an den MikroTik weitergeleitet wird.

✅ **2. WireGuard-Peer nicht richtig konfiguriert**
- Falls der externe Nutzer keine Verbindung bekommt:
  ```sh
  /interface wireguard peers print
  ```
- Falls der Peer fehlt, neu hinzufügen:
  ```sh
  /interface wireguard peers add allowed-address=10.10.10.3/32 interface=wireguard1 public-key="CLIENT-PUBLIC-KEY"
  ```

✅ **3. Firewall blockiert externen Zugriff**
- Prüfe:
  ```sh
  /ip firewall filter print
  ```
- Falls nötig, neue Regel hinzufügen:
  ```sh
  /ip firewall filter add action=accept chain=input protocol=udp dst-port=13231
  ```

---

## **📌 Fazit**
- Diese Anleitung hilft dir, typische Probleme bei deinem VPN-Setup zu lösen.
- Falls du weiterhin Probleme hast, prüfe die **Logs** und die **Firewall-Regeln**.

🚀 **Dein VPN sollte jetzt stabil laufen!** 🎉
