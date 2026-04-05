# Architektur: Interne Domains über MikroTik Split-DNS für Nginx Proxy Manager

## Ziel

Diese Dokumentation beschreibt die Architekturentscheidung, interne Dienste hinter dem Nginx Proxy Manager (NPM) im Heimnetz **nicht über Hairpin NAT**, sondern per **Split-DNS über den MikroTik** aufzulösen.

Ziel ist, dass interne Clients dieselben FQDNs verwenden können wie externe Clients, ohne dabei den Umweg über die öffentliche WAN-IP zu nehmen.

---

## Kontext

In der Umgebung werden mehrere Dienste über externe Domains veröffentlicht, die intern und extern unter derselben Adresse erreichbar sein sollen, zum Beispiel:

* `homeassistant.meinedomain.de`
* `grocy.meinedomain.de`
* `plex.meinedomain.de`
* `paperless.meinedomain.de`
* `barcodebuddy.meinedomain.de`
* `vikunja.meinedomain.de`
* `photos.meinedomain.de`

Der Reverse Proxy läuft als Nginx Proxy Manager (NPM) auf einem Docker-Host mit der IP `192.168.88.106`.

Interne Clients verwenden den MikroTik als DNS-Server (`192.168.88.1`).

---

## Problemstellung

Ohne interne DNS-Steuerung werden diese Domains regulär öffentlich aufgelöst. Interne Clients erhalten dann die öffentliche WAN-IP und versuchen, interne Dienste über den Weg

```text
interner Client -> öffentliche IP -> Router/NAT -> zurück ins interne Netz
```

zu erreichen.

Das führt je nach Topologie und NAT-Verhalten zu Problemen wie:

* Hairpin-/NAT-Loopback-Abhängigkeiten
* uneinheitlicher interner Erreichbarkeit
* unnötigen Umwegen
* schwer nachvollziehbaren Fehlerbildern
* zusätzlicher Komplexität bei Fehlersuche und Betrieb

---

## Architekturentscheidung

Interne Reverse-Proxy-Domains werden per **statischer DNS-Auflösung auf dem MikroTik** direkt auf den Docker-/NPM-Host `192.168.88.106` aufgelöst.

Damit gilt für interne Clients:

```text
interner Client -> MikroTik DNS -> 192.168.88.106 -> NPM -> Zielservice
```

anstatt:

```text
interner Client -> öffentliche WAN-IP -> Hairpin NAT / Loopback -> NPM -> Zielservice
```

Diese Entscheidung entspricht einem klassischen **Split-DNS-Ansatz**.

---

## Begründung

### 1. Konsistente interne Erreichbarkeit

Alle internen Clients erhalten für relevante Dienste dieselbe interne Ziel-IP. Dadurch wird das Verhalten reproduzierbar und unabhängig vom Hairpin-/Loopback-Verhalten des Routers.

### 2. Geringere Komplexität

DNS-Overrides auf dem MikroTik sind gezielter und leichter nachvollziehbar als zusätzliche NAT-/Firewall-Konstrukte.

### 3. Bessere Fehlersuche

Wenn interne Domains direkt auf den Reverse Proxy zeigen, lassen sich Probleme sauber in diese Ebenen aufteilen:

* DNS
* Reverse Proxy
* Upstream-Service
* Anwendungskonfiguration

Ohne diesen Ansatz vermischen sich Routing-, NAT- und DNS-Probleme unnötig.

### 4. Kompatibel mit laufendem Netzwerkumbau

Die Lösung greift nicht in Routing, WAN, NAT-Grundlogik oder Docker-Netzwerke ein, sondern nur in die interne Namensauflösung. Sie ist deshalb risikoarm und gut als stabiler Zwischen- oder Dauerzustand geeignet.

### 5. Einheitliche URLs intern und extern

Benutzer und Anwendungen können intern und extern dieselben Domains verwenden. Das reduziert Sonderfälle, Bookmarks, App-spezifische Abweichungen und Verwechslungsgefahr.

---

## Komponenten

### MikroTik

* Rolle: interner DNS-Server für Clients
* Aufgabe: statische DNS-Auflösung der internen Reverse-Proxy-Domains auf `192.168.88.106`

### Docker-Host

* Host: HP EliteDesk
* IP: `192.168.88.106`
* Aufgabe: Betrieb des NPM-Containers und der angebundenen Dienste

### Nginx Proxy Manager (NPM)

* Rolle: Reverse Proxy
* Aufgabe: Aufteilung eingehender Anfragen anhand des Hostnamens auf die jeweiligen Zielservices

### Interne Clients

* verwenden den MikroTik als DNS-Server
* greifen über dieselben FQDNs wie externe Clients auf Dienste zu

---

## Namensauflösung

### Grundsatz

Alle internen Domains, die über NPM veröffentlicht werden und intern erreichbar sein sollen, müssen auf dem MikroTik statisch auf den Docker-/NPM-Host zeigen.

### Beispiel

```text
homeassistant.meinedomain.de -> 192.168.88.106
grocy.meinedomain.de         -> 192.168.88.106
plex.meinedomain.de          -> 192.168.88.106
paperless.meinedomain.de     -> 192.168.88.106
barcodebuddy.meinedomain.de  -> 192.168.88.106
vikunja.meinedomain.de       -> 192.168.88.106
photos.meinedomain.de        -> 192.168.88.106
```

---

## Datenfluss

### Externe Clients

```text
externer Client -> öffentliche DNS-Auflösung -> öffentliche WAN-IP -> Portweiterleitung / Reverse Proxy -> NPM -> Zielservice
```

### Interne Clients

```text
interner Client -> MikroTik DNS Static -> 192.168.88.106 -> NPM -> Zielservice
```

### Vorteil

Interne und externe Clients verwenden denselben Hostnamen, aber intern wird der Umweg über die öffentliche Adresse vermieden.

---

## MikroTik-Beispielkonfiguration

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

Nach Änderungen sollte der MikroTik-DNS-Cache geleert werden:

```rsc
/ip dns cache flush
```

Auf Linux-Clients kann zusätzlich der lokale Resolver-Cache geleert werden:

```bash
resolvectl flush-caches
```

---

## Betriebsregeln

### 1. Neue NPM-Dienste immer vollständig aufnehmen

Wenn ein neuer Dienst über NPM veröffentlicht wird und intern unter derselben Domain erreichbar sein soll, muss direkt geprüft werden, ob ein statischer DNS-Eintrag auf dem MikroTik erforderlich ist.

### 2. Änderungen an der NPM-Host-IP nachziehen

Die Architektur hängt daran, dass `192.168.88.106` der korrekte Docker-/NPM-Host ist. Bei IP-Änderungen müssen alle statischen DNS-Einträge angepasst werden.

### 3. DNS-Zuständigkeit beachten

Die Lösung setzt voraus, dass interne Clients den MikroTik als DNS-Server verwenden. Wenn sich die DNS-Architektur ändert, müssen die Einträge migriert werden.

### 4. Keine implizite Abhängigkeit von Hairpin NAT

Interne Erreichbarkeit über Reverse-Proxy-Domains soll nicht davon abhängen, ob Hairpin NAT zufällig funktioniert oder nicht.

---

## Vor- und Nachteile

### Vorteile

* konsistente interne Erreichbarkeit
* gleiche Domains intern und extern
* geringere NAT-/Loopback-Abhängigkeit
* einfachere Fehlersuche
* geringe Eingriffstiefe
* gut dokumentierbar und nachvollziehbar

### Nachteile / Trade-offs

* statische Einträge müssen gepflegt werden
* bei IP-Änderungen des Docker-/NPM-Hosts ist Nacharbeit nötig
* bei Wechsel des internen DNS-Servers müssen die Einträge migriert werden
* zusätzlicher administrativer Schritt bei neuen Diensten

---

## Abgrenzung zu Hairpin NAT

Hairpin NAT kann technisch ebenfalls dazu genutzt werden, interne Clients über die öffentliche IP wieder zurück ins interne Netz zu führen. In dieser Umgebung wurde dieser Ansatz **nicht** als primäre Architektur gewählt.

Gründe:

* unnötige Abhängigkeit von NAT-Loopback-Verhalten
* höherer Debugging-Aufwand
* potenziell uneinheitliches Verhalten je nach Dienst oder Topologie
* schlechtere Transparenz im Betrieb

Split-DNS ist in dieser Umgebung der klarere und stabilere Ansatz.

---

## Validierung / Prüfpunkte

### Linux-Client

```bash
nslookup <subdomain>.meinedomain.de
getent hosts <subdomain>.meinedomain.de
resolvectl status
resolvectl query <subdomain>.meinedomain.de
```

### Erwartung

Relevante interne Domains lösen auf:

```text
192.168.88.106
```

### MikroTik

```rsc
/ip dns static print where name~"meinedomain.de"
/ip dns cache print where name~"meinedomain.de"
```

---

## Konsequenzen für die weitere Architektur

Diese Entscheidung ist sowohl als stabiler Betriebszustand als auch als Übergangslösung während des laufenden Netzwerkumbaus geeignet.

Falls die Infrastruktur später weiter zentralisiert oder segmentiert wird, bleibt das Grundprinzip bestehen:

* interne Dienste sollen intern direkt auflösbar sein
* Reverse-Proxy-Domains sollen intern nicht vom Umweg über die öffentliche WAN-IP abhängen

Die konkrete technische Umsetzung kann später auf einen anderen internen DNS-Server migriert werden, ohne dass das Architekturprinzip geändert werden muss.

---

## Verwandte Dokumente

* [Incident-Dokumentation zur Störung der internen Erreichbarkeit von NPM-Domains](docs/04-incidents/npm-internal-dns.md)
* Troubleshooting-Dokumentation für DNS-/Reverse-Proxy-Probleme
* Topologie- und Architekturübersicht des Heimnetzes
