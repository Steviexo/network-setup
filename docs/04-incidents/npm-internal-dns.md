# Incident: Interne NPM-Domains im MikroTik-Netz nicht erreichbar

## Metadaten

* **Datum:** 2026-04
* **Bereich:** Netzwerk / DNS / Reverse Proxy
* **Komponenten:** MikroTik, Docker-Host `192.168.88.106`, Nginx Proxy Manager (NPM), interne Clients
* **Betroffene Dienste:** Plex, Home Assistant, Grocy, BarcodeBuddy, Vikunja, Photos, Paperless
* **Status:** Gelöst

---

## Kurzbeschreibung

Mehrere interne Dienste, die über externe Domains hinter dem Nginx Proxy Manager (NPM) veröffentlicht werden, waren aus dem MikroTik-internen Netz nicht oder nur teilweise erreichbar. Extern funktionierten die betroffenen Dienste, intern war das Verhalten uneinheitlich.

Die Ursache war keine Container- oder NPM-Störung, sondern eine inkonsistente interne Namensauflösung: Einige Subdomains wurden intern auf die öffentliche WAN-IP aufgelöst statt direkt auf den Docker-/NPM-Host `192.168.88.106`. Dadurch liefen interne Zugriffe in Hairpin-/Loopback-Probleme.

---

## Symptome

Folgende Symptome wurden beobachtet:

* `plex.meinedomain.de` war intern nicht erreichbar.
* `homeassistant.meinedomain.de` war intern nicht über die externe Domain erreichbar, jedoch direkt über `Host-IP:Port`.
* `barcodebuddy.meinedomain.de` war intern zunächst nicht erreichbar.
* `grocy.meinedomain.de` war intern zunächst nicht erreichbar.
* `vikunja.meinedomain.de` funktionierte intern bereits.
* Das Verhalten war insgesamt uneinheitlich und dadurch irreführend.

---

## Ausgangslage

* Der Docker-Host läuft auf einem HP EliteDesk mit der IP `192.168.88.106`.
* NPM läuft als Docker-Container auf diesem Host.
* Interne Clients nutzen den MikroTik als DNS-Server (`192.168.88.1`).
* Die betroffenen Dienste werden per Reverse Proxy über NPM veröffentlicht.
* Die Umgebung befand sich parallel in einem laufenden Netzwerkumbau.

---

## Analyse

### 1. DNS-Auflösung prüfen

Auf einem internen Client wurden die betroffenen Domains geprüft.

Beobachtung:

* `vikunja.meinedomain.de` wurde zeitweise intern auf `192.168.88.106` aufgelöst.
* `homeassistant.meinedomain.de` und `plex.meinedomain.de` wurden auf die öffentliche IP aufgelöst.
* `grocy.meinedomain.de` war intern ebenfalls nicht korrekt erreichbar.

Beispielhafte Prüfungen:

```bash
nslookup plex.meinedomain.de
nslookup homeassistant.meinedomain.de
nslookup grocy.meinedomain.de
getent hosts plex.meinedomain.de
resolvectl status
resolvectl query plex.meinedomain.de
```

### 2. DNS-Quelle identifizieren

Der interne Client nutzte den MikroTik als DNS-Server:

```text
Current DNS Server: 192.168.88.1
DNS Servers: 192.168.88.1
```

Damit war klar, dass die relevante Namensauflösung im internen Netz durch den MikroTik erfolgte.

### 3. MikroTik prüfen

Auf dem MikroTik wurden bestehende statische DNS-Einträge und Cache-Einträge für die betroffenen Domains geprüft:

```rsc
/ip dns static print where name~"meinedomain.de"
/ip dns cache print where name~"meinedomain.de"
```

Es waren keine vollständigen statischen Einträge für alle betroffenen NPM-Domains vorhanden.

---

## Root Cause

Die interne DNS-Auflösung für mehrere über NPM veröffentlichte Subdomains war unvollständig bzw. inkonsistent.

Statt intern direkt auf den Docker-/NPM-Host `192.168.88.106` zu zeigen, wurden mehrere Subdomains auf die öffentliche WAN-IP aufgelöst. Dadurch mussten interne Clients den Umweg über die externe Adresse nehmen. In dieser Topologie führte das zu Hairpin-/NAT-Loopback-Problemen bzw. inkonsistentem Verhalten.

**Kernaussage:** Das Hauptproblem lag nicht an Docker, nicht primär an NPM und nicht an den einzelnen Containern, sondern an fehlendem Split-DNS auf dem MikroTik.

---

## Lösung

Auf dem MikroTik wurden statische DNS-Einträge für alle relevanten internen NPM-Domains angelegt, sodass interne Clients diese Subdomains direkt auf `192.168.88.106` auflösen.

### Beispielbefehle

```rsc
/ip dns static
add name=homeassistant.meinedomain.de address=192.168.88.106 comment="interner NPM-Proxy"
add name=grocy.meinedomain.de address=192.168.88.106 comment="interner NPM-Proxy"
add name=plex.meinedomain.de address=192.168.88.106 comment="interner NPM-Proxy"
add name=paperless.meinedomain.de address=192.168.88.106 comment="interner NPM-Proxy"
add name=barcodebuddy.meinedomain.de address=192.168.88.106 comment="interner NPM-Proxy"
add name=vikunja.meinedomain.de address=192.168.88.106 comment="interner NPM-Proxy"
add name=photos.meinedomain.de address=192.168.88.106 comment="interner NPM-Proxy"
```

Hinweis: Für `vikunja.meinedomain.de` existierte bereits ein Eintrag.

### Caches leeren

Nach dem Anlegen der Einträge wurden die DNS-Caches geleert.

Auf dem MikroTik:

```rsc
/ip dns cache flush
```

Auf dem Linux-Client:

```bash
resolvectl flush-caches
```

---

## Verifikation

Nach dem Setzen der statischen DNS-Einträge wurden die Auflösungen und die Erreichbarkeit erneut geprüft.

### Prüfkommandos

```bash
nslookup plex.meinedomain.de
nslookup homeassistant.meinedomain.de
nslookup grocy.meinedomain.de
getent hosts plex.meinedomain.de
```

### Erwartetes Ergebnis

Alle relevanten internen NPM-Domains lösen auf:

```text
192.168.88.106
```

### Ergebnis nach Umsetzung

* Plex war intern wieder erreichbar.
* Home Assistant war intern wieder über die externe Domain erreichbar.
* Vikunja war weiterhin erreichbar.
* BarcodeBuddy war wieder erreichbar.
* Nach Ergänzung von `grocy.meinedomain.de` verschwand auch der Fehler in BarcodeBuddy.
* Grocy war anschließend intern ebenfalls über `grocy.meinedomain.de` erreichbar.
* Zuletzt wurde noch `photos.meinedomain.de` ergänzt, um die interne Erreichbarkeit vollständig zu machen.

---

## Auswirkungen

Die interne Erreichbarkeit aller relevanten NPM-Dienste wurde wiederhergestellt, ohne Änderungen an Routing, NAT-Regeln, Docker oder den einzelnen Service-Containern vornehmen zu müssen.

Die Lösung war gezielt, risikoarm und mit dem laufenden Netzwerkumbau kompatibel.

---

## Lessons Learned

1. **Bei uneinheitlicher interner Erreichbarkeit von Reverse-Proxy-Domains zuerst DNS prüfen.**
2. **Split-DNS ist in dieser Umgebung sauberer und robuster als Hairpin NAT.**
3. **Wenn interne Clients den MikroTik als DNS verwenden, müssen alle relevanten NPM-Domains dort konsistent gepflegt werden.**
4. **Einzelne funktionierende Dienste können irreführend sein, wenn nur Teilmengen der Domains intern korrekt aufgelöst werden.**
5. **Vor Änderungen an Docker, NPM oder Firewalls immer zuerst prüfen, ob das Problem bereits auf DNS-Ebene entsteht.**

---

## Betriebsrelevante Hinweise

* Der Eintragssatz hängt aktuell am Docker-/NPM-Host `192.168.88.106`.
* Falls sich die IP dieses Hosts ändert, müssen die statischen DNS-Einträge auf dem MikroTik angepasst werden.
* Falls künftig ein anderer interner DNS-Server als der MikroTik verwendet wird, müssen diese Einträge dorthin migriert werden.
* Neue Dienste hinter NPM sollten direkt mitgedacht werden: Externe Veröffentlichung allein reicht nicht, wenn interne Clients dieselben Domains ebenfalls nutzen sollen.

---

## Anhang: Relevante Prüfbefehle

### Linux-Client

```bash
nslookup <subdomain>.meinedomain.de
getent hosts <subdomain>.meinedomain.de
resolvectl status
resolvectl query <subdomain>.meinedomain.de
resolvectl flush-caches
```

### MikroTik

```rsc
/ip dns static print where name~"meinedomain.de"
/ip dns cache print where name~"meinedomain.de"
/ip dns cache flush
```
