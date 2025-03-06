# 🔒 Sicherheit im MikroTik VPN-Setup

Dieses Dokument beschreibt Best Practices und Konfigurationsmaßnahmen, um dein **MikroTik WireGuard VPN sicher zu halten** und ungewollte Zugriffe zu verhindern.

---

## **📌 Grundlegende Sicherheitsmaßnahmen**
### ✅ **1️⃣ Starke VPN-Schlüssel verwenden**
- Generiere für jeden WireGuard-Client **einzigartige Schlüsselpaare**.
- Vermeide die Wiederverwendung von Schlüsseln.
- Prüfe die gespeicherten Peers:
  ```sh
  /interface wireguard peers print
  ```

### ✅ **2️⃣ Zugang nur für bekannte IPs erlauben**
- Falls externe Nutzer eine **statische IP** haben, beschränke den Zugriff:
  ```sh
  /ip firewall filter add chain=input action=accept src-address=EXTERNE-IP protocol=udp dst-port=13231
  ```
- Alle anderen Verbindungen blockieren:
  ```sh
  /ip firewall filter add chain=input action=drop protocol=udp dst-port=13231
  ```

### ✅ **3️⃣ Logging von VPN-Zugriffen aktivieren**
- Logge erfolgreiche und fehlgeschlagene Anmeldeversuche:
  ```sh
  /system logging add topics=wireguard action=memory
  ```
- Prüfe Logs mit:
  ```sh
  /log print where message~"wireguard"
  ```

---

## **🛡️ Firewall-Schutz für das VPN**
### ✅ **4️⃣ Nur VPN-Clients Zugriff auf das lokale Netzwerk geben**
- Erlaube nur VPN-Nutzer, die sich verbinden dürfen:
  ```sh
  /ip firewall filter add chain=forward action=accept src-address=10.10.10.0/24 dst-address=192.168.88.0/24
  ```
- Blockiere alle anderen unerwünschten Weiterleitungen:
  ```sh
  /ip firewall filter add chain=forward action=drop src-address=10.10.10.0/24
  ```

### ✅ **5️⃣ Zugang zum MikroTik-Router absichern**
- Erlaube nur SSH-Zugriffe von deinem lokalen Netzwerk:
  ```sh
  /ip firewall filter add chain=input action=accept src-address=192.168.88.0/24 protocol=tcp dst-port=22
  /ip firewall filter add chain=input action=drop protocol=tcp dst-port=22
  ```
- Blockiere unnötige Dienste für externe Nutzer:
  ```sh
  /ip service disable telnet
  /ip service disable ftp
  ```

---

## **🔐 DNS-Sicherheit & Datenschutz**
### ✅ **6️⃣ Sichere DNS-Server nutzen**
- Setze Google DNS oder Cloudflare DNS, um DNS-Leaks zu vermeiden:
  ```sh
  /ip dns set servers=8.8.8.8,1.1.1.1 allow-remote-requests=yes
  ```
- Falls externe VPN-Nutzer ebenfalls diesen DNS-Server nutzen sollen:
  ```sh
  /ip firewall nat add chain=dstnat protocol=udp dst-port=53 action=redirect to-ports=53
  ```

### ✅ **7️⃣ DNS-Leaks für externe VPN-Nutzer verhindern**
- Setze in der WireGuard-Konfiguration der Clients folgende DNS-Einstellung:
  ```ini
  [Interface]
  DNS = 8.8.8.8, 1.1.1.1
  ```

---

## **🚧 Weitere Schutzmaßnahmen**
### ✅ **8️⃣ Zugriff auf den MikroTik-Router auf ein VLAN beschränken**
- Falls VLANs genutzt werden, sollte der Router-Admin-Zugang nur aus einem Management-Netzwerk möglich sein:
  ```sh
  /interface vlan add name=vlan-management vlan-id=99 interface=bridge
  ```

### ✅ **9️⃣ Port-Scanning verhindern**
- Blockiere aggressive Scan-Versuche:
  ```sh
  /ip firewall filter add chain=input action=drop protocol=tcp tcp-flags=syn src-address-list=port-scanners
  ```
- Automatisches Sperren verdächtiger IPs:
  ```sh
  /ip firewall address-list add list=port-scanners timeout=1d
  ```

---

## **📌 Fazit**
- **VPN nur für autorisierte Clients mit starken Schlüsseln.**
- **Firewall-Regeln sichern den Zugriff auf das Netzwerk.**
- **DNS-Sicherheit stellt sicher, dass kein Leak entsteht.**
- **Logging hilft, unbefugte Zugriffe zu erkennen.**

🚀 **Mit diesen Maßnahmen bleibt dein VPN sicher und geschützt!** 🔐
