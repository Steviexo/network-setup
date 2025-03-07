# ⚙️ MikroTik VPN-Konfiguration (WireGuard)

Dieses Dokument beschreibt die detaillierte Konfiguration des **WireGuard VPN-Servers** auf einem MikroTik-Router, einschließlich der Einrichtung des VPN-WLANs für Streaming-Geräte.

---

## **📌 1️⃣ WireGuard-VPN einrichten**

### **1.1 WireGuard-Interface erstellen**
```sh
/interface/wireguard add listen-port=13231 name=wireguard1
```
- Den generierten **öffentlichen Schlüssel** notieren.

### **1.2 IP-Adresse für das WireGuard Interface setzen**
```sh
/ip/address add address=10.10.10.1/24 interface=wireguard1 network=10.10.10.0
```

### **1.3 Firewall-Regeln für WireGuard**
```sh
/ip/firewall/filter add action=accept chain=input dst-port=13231 protocol=udp
/ip/firewall/filter add action=accept chain=forward in-interface=wireguard1
/ip/firewall/filter add action=accept chain=forward out-interface=wireguard1
```

### **1.4 NAT für VPN-Clients aktivieren**
```sh
/ip/firewall/nat add action=masquerade chain=srcnat out-interface=ether1
/ip/firewall/nat add action=masquerade chain=srcnat src-address=10.10.10.0/24
```

---

## **📡 2️⃣ WLAN für VPN-Streaming einrichten**

### **2.1 Spezielles WLAN-Interface für VPN-Traffic erstellen**
```sh
/interface wireless add name=wlan-for-streaming-vpn ssid=LAN_solo mode=ap frequency=2412 band=2ghz-g security-profile=VPN-Security
```

### **2.2 IP-Adresse für das WLAN setzen**
```sh
/ip/address add address=192.168.10.1/24 interface=wlan-for-streaming-vpn
```

### **2.3 DHCP-Server für das VPN-WLAN einrichten**
```sh
/ip pool add name=streaming-vpn-pool ranges=192.168.10.10-192.168.10.250
/ip dhcp-server add name=dhcp-streaming-vpn interface=wlan-for-streaming-vpn address-pool=streaming-vpn-pool lease-time=10m
```

---

## **🔀 3️⃣ Routing und Firewall für VPN-WLAN**

### **3.1 Routing für VPN-WLAN über WireGuard leiten**
```sh
/ip route add dst-address=0.0.0.0/0 gateway=wireguard1 routing-mark=vpn-traffic
```

### **3.2 Firewall-Regeln zur Traffic-Kontrolle**
```sh
/ip firewall mangle add action=mark-routing chain=prerouting src-address=192.168.10.0/24 new-routing-mark=vpn-traffic
/ip firewall filter add action=accept chain=forward connection-state=established,related
/ip firewall filter add action=drop chain=forward src-address=192.168.10.0/24 dst-address=192.168.88.0/24
```

### **3.3 NAT für VPN-WLAN aktivieren**
```sh
/ip firewall nat add chain=srcnat src-address=192.168.10.0/24 action=masquerade
```

---

## **🌍 4️⃣ Externe Geräte mit dem VPN verbinden**

### **4.1 Externen WireGuard-Peer hinzufügen**
```sh
/interface wireguard peers add allowed-address=10.10.10.3/32 interface=wireguard1 public-key="CLIENT-PUBLIC-KEY"
```

### **4.2 Beispiel WireGuard-Konfigurationsdatei für externe Nutzer**
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

---

## **🔍 5️⃣ Überprüfung und Tests**

### **5.1 Überprüfen, ob WireGuard läuft**
```sh
/interface wireguard print
/interface wireguard peers print
```

### **5.2 DHCP-Server-Status prüfen**
```sh
/ip dhcp-server print
/ip dhcp-server lease print
```

### **5.3 Firewall-Regeln checken**
```sh
/ip firewall filter print
/ip firewall nat print
```

---

## **📌 Fazit**
- **WireGuard-VPN ist eingerichtet** ✅
- **Spezielles WLAN für VPN-Streaming läuft** ✅
- **Firewall-Regeln sichern den Traffic** ✅
- **Externe Geräte können sich verbinden** ✅

🚀 **Dein MikroTik ist nun optimal für sicheres Streaming über VPN konfiguriert!** 🔥
