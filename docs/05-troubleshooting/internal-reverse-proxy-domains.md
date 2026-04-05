# Troubleshooting: Interne Reverse-Proxy-Domains über MikroTik, DNS und NPM

## Zweck

Diese Dokumentation beschreibt eine allgemeine Vorgehensweise zur Fehlersuche, wenn interne Dienste hinter einem Reverse Proxy extern erreichbar sind, intern über ihre normalen Domains jedoch nicht oder nur teilweise funktionieren.

Der Fokus liegt auf einer Umgebung mit:

* MikroTik als internem DNS-Server
* Nginx Proxy Manager (NPM) als Reverse Proxy
* Docker-Host als Zielsystem für interne Reverse-Proxy-Domains

---

## Typische Symptome

Probleme in diesem Bereich zeigen sich oft durch eines oder mehrere dieser Muster:

* Ein Dienst funktioniert extern, intern aber nicht über seine normale Domain.
* Ein Dienst funktioniert intern nur über `Host-IP:Port`, aber nicht über die Domain.
* Einige Dienste funktionieren intern, andere nicht.
* Browser-Zugriff geht, aber Apps oder Integrationen scheitern.
* Einzelne Dienste hinter NPM zeigen Verbindungs- oder Redirect-Probleme.
* Integrationen melden Fehler wie `could not connect`, obwohl der Dienst scheinbar läuft.

---

## Grundprinzip der Fehlersuche

Nicht sofort an Containern, NAT-Regeln oder Firewalls drehen.

Zuerst sauber trennen, **auf welcher Ebene** das Problem entsteht:

1. DNS / Namensauflösung
2. Erreichbarkeit des Reverse Proxy
3. NPM-Weiterleitung zum Upstream
4. Anwendung / Integration / API

Wenn diese Reihenfolge nicht eingehalten wird, endet man schnell in planlosem Konfigurationsroulette.

---

## Prüfreihenfolge

## 1. Prüfen, worauf die Domain intern auflöst

### Linux-Client

```bash
nslookup <subdomain>.meinedomain.de
getent hosts <subdomain>.meinedomain.de
resolvectl query <subdomain>.meinedomain.de
```

### Erwartung in dieser Architektur

Die interne Domain sollte auf die interne IP des Docker-/NPM-Hosts zeigen, z. B.:

```text
192.168.88.106
```

### Typische Befunde

#### Befund A: Auflösung auf öffentliche WAN-IP

Dann greift intern keine saubere Split-DNS-Auflösung. Der Client versucht den Umweg über die öffentliche Adresse. Das führt oft zu Hairpin-/Loopback-Problemen.

**Nächster Schritt:** MikroTik-DNS prüfen.

#### Befund B: Auflösung auf interne IP

Dann ist DNS wahrscheinlich nicht mehr das Hauptproblem. Als Nächstes NPM und den Zielservice prüfen.

#### Befund C: Unterschiedliche Ergebnisse je nach Tool

Dann Resolver-Caches, unterschiedliche Anfragepfade oder lokale Overrides prüfen.

---

## 2. Prüfen, welchen DNS-Server der Client wirklich nutzt

### Linux-Client

```bash
resolvectl status
cat /etc/resolv.conf
```

### Ziel

Klären, welcher DNS-Server tatsächlich antwortet.

In dieser Architektur sollte ein interner Client typischerweise den MikroTik verwenden, z. B.:

```text
192.168.88.1
```

### Wichtige Erkenntnis

Solange nicht klar ist, wer die DNS-Antwort liefert, ist jede weitere Analyse potenziell schief.

---

## 3. Lokale Overrides ausschließen

### Linux-Client

```bash
grep -n "<subdomain>.meinedomain.de" /etc/hosts
```

### Zweck

Ausschließen, dass ein einzelner Client die Domain lokal anders auflöst als der Rest des Netzes.

---

## 4. MikroTik-DNS prüfen

### Bestehende statische Einträge prüfen

```rsc
/ip dns static print where name~"meinedomain.de"
```

### DNS-Cache prüfen

```rsc
/ip dns cache print where name~"meinedomain.de"
```

### Typische Befunde

#### Befund A: Kein statischer Eintrag vorhanden

Dann wird die Domain wahrscheinlich öffentlich aufgelöst. Wenn interne Clients dieselbe Domain intern nutzen sollen, fehlt ein Split-DNS-Eintrag.

#### Befund B: Einträge nur für einen Teil der Domains

Dann entsteht das klassische uneinheitliche Verhalten: manche Dienste funktionieren intern, andere nicht.

#### Befund C: Eintrag vorhanden, aber falsche IP

Dann zeigt die Domain intern auf das falsche Ziel. Korrigieren.

---

## 5. Statische DNS-Einträge anlegen oder korrigieren

### Beispiel

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

### Danach Caches leeren

MikroTik:

```rsc
/ip dns cache flush
```

Linux-Client:

```bash
resolvectl flush-caches
```

### Danach erneut testen

```bash
nslookup <subdomain>.meinedomain.de
getent hosts <subdomain>.meinedomain.de
```

---

## 6. Prüfen, ob die Domain intern im Browser oder per HTTP(S) erreichbar ist

Wenn DNS jetzt auf die interne NPM-IP zeigt, aber der Dienst weiterhin nicht funktioniert, als Nächstes die HTTP-Ebene prüfen.

### Vom Client oder Host aus

```bash
curl -I https://<subdomain>.meinedomain.de
curl -vk https://<subdomain>.meinedomain.de
```

### Ziel

Prüfen, ob:

* überhaupt eine Antwort kommt
* TLS funktioniert
* Redirects auftreten
* HTTP-Statuscodes sinnvoll sind
* die Domain wirklich beim Reverse Proxy ankommt

---

## 7. Prüfen, ob das Problem nur im Browser nicht, aber in Integrationen oder Containern besteht

Wenn ein Dienst im Browser funktioniert, eine Integration aber nicht, liegt das Problem oft **nicht mehr auf DNS-Ebene des Clients**, sondern im Laufzeitkontext der Anwendung.

Typische Beispiele:

* BarcodeBuddy kann Grocy nicht erreichen
* Container A erreicht Dienst B nicht über dieselbe URL wie der Browser
* API-Aufrufe schlagen fehl, obwohl die Weboberfläche geht

### Docker-Container prüfen

Zuerst Container identifizieren:

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```

Dann in den relevanten Container wechseln:

```bash
docker exec -it <containername> sh
```

oder:

```bash
docker exec -it <containername> bash
```

Von dort testen:

```bash
wget -S -O - https://<subdomain>.meinedomain.de/api
```

oder:

```bash
curl -I https://<subdomain>.meinedomain.de/api
curl -vk https://<subdomain>.meinedomain.de/api
```

### Ziel

Prüfen, ob die Anwendung oder der Container dieselbe URL wirklich erreichen kann.

---

## 8. Wenn eine Integration auf eine Domain zeigt, die intern selbst nicht funktioniert

Dann ist die Integration nicht das Problem, sondern nur der Überbringer der schlechten Nachricht.

Beispiel:

* BarcodeBuddy nutzt `https://grocy.meinedomain.de/api`
* `grocy.meinedomain.de` funktioniert intern selbst noch nicht
* dann scheitert zwangsläufig auch BarcodeBuddy

**Vorgehen:** Erst die Domain selbst intern funktionsfähig machen, dann die Integration erneut testen.

---

## 9. NPM prüfen, wenn DNS korrekt ist, aber der Dienst weiter scheitert

Wenn die Domain intern sauber auf die NPM-IP zeigt und der Dienst trotzdem nicht funktioniert, liegt das Problem wahrscheinlich in einer dieser Ebenen:

* falscher NPM-Proxy-Host
* falsches Upstream-Ziel
* falsches Protokoll (`http` vs. `https`)
* Redirect-/Base-URL-Probleme
* Header-/Websocket-Themen
* Trusted-Proxy- oder Host-Header-Probleme der Anwendung

### Was dann prüfen

* Ziel-IP / Ziel-Port in NPM
* verwendetes Schema (`http` oder `https`)
* Websocket-Support
* individuelle App-Anforderungen
* Logs von NPM und Zielservice

---

## Typische Root Causes in dieser Umgebung

### 1. Fehlende Split-DNS-Einträge auf dem MikroTik

Der häufigste Fall. Einige Domains zeigen intern auf die öffentliche IP statt auf den Docker-/NPM-Host.

### 2. Unvollständige Domainliste

Ein Teil der NPM-Domains ist statisch eingetragen, andere fehlen. Ergebnis: manche Dienste funktionieren intern, andere nicht.

### 3. Falsche Ziel-IP in statischen DNS-Einträgen

Der Eintrag zeigt auf einen alten oder falschen Host.

### 4. Resolver-Cache

Die Konfiguration ist bereits korrekt, aber Client oder MikroTik liefern noch alte Antworten aus dem Cache.

### 5. Integration oder Container nutzt eine URL, die intern selbst nicht funktioniert

Dann ist nicht die Integration kaputt, sondern die Ziel-Domain intern noch nicht sauber hergestellt.

---

## Entscheidungslogik in Kurzform

### Fall 1

**Domain löst intern auf öffentliche IP auf**

→ Split-DNS auf dem MikroTik fehlt oder ist unvollständig.

### Fall 2

**Domain löst intern auf interne NPM-IP auf, Browser geht, Integration nicht**

→ Integration / Container / API / Zertifikat / Auth prüfen.

### Fall 3

**Domain löst intern auf interne NPM-IP auf, Browser geht nicht**

→ NPM-Proxy-Host, Upstream, TLS oder App-Konfiguration prüfen.

### Fall 4

**Einige Domains gehen, andere nicht**

→ Domainliste auf dem MikroTik auf Vollständigkeit prüfen.

---

## Bewährte Minimal-Checks

### Client

```bash
nslookup <subdomain>.meinedomain.de
getent hosts <subdomain>.meinedomain.de
resolvectl status
resolvectl query <subdomain>.meinedomain.de
```

### MikroTik

```rsc
/ip dns static print where name~"meinedomain.de"
/ip dns cache print where name~"meinedomain.de"
/ip dns cache flush
```

### Host / Container

```bash
curl -I https://<subdomain>.meinedomain.de
curl -vk https://<subdomain>.meinedomain.de

docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
docker exec -it <containername> sh
```

---

## Was man in dieser Situation bewusst nicht zuerst tun sollte

* nicht sofort Hairpin NAT bauen
* nicht sofort an Docker-Netzwerken herumändern
* nicht sofort mehrere NPM-Hosts blind umbauen
* nicht gleichzeitig DNS, NAT und Container-Config ändern

Sonst weiß man am Ende nicht mehr, was das Problem gelöst oder verschlimmert hat.

---

## Empfehlung für den Betrieb

Bei jeder neu veröffentlichten NPM-Domain prüfen:

1. Soll sie intern unter derselben Domain erreichbar sein?
2. Verwenden interne Clients den MikroTik als DNS?
3. Gibt es dafür bereits einen statischen DNS-Eintrag?
4. Löst die Domain intern auf die NPM-IP auf?

Wenn diese vier Punkte sauber beantwortet werden, spart das später viel Fehlersuche.

---

## Verwandte Dokumente

* [Architekturentscheidung für Split-DNS via MikroTik statt Hairpin NAT](docs/01-architecture/internal-split-dns-via-mikrotik.md)
* [Incident-Dokumentation zur internen Nichterreichbarkeit von NPM-Domains](docs/04-incidents/npm-internal-dns.md)
* [Topologie- und Betriebsdokumentation des Heimnetzes](docs/overview)
