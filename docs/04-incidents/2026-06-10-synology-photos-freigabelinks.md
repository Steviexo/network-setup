# 2026-06-10: Synology Photos Freigabelinks mit falscher Domain

## TL;DR

Freigabelinks von Synology Photos nutzten `username.synology.me` statt der gewünschten Domain `photos.meinedomain.de`.  
Die Ursache war eine **inkonsistente NPM-Konfiguration**: Im **Details-Tab** war `Scheme: http` für Port 5001 eingestellt, obwohl DSM auf Port 5001 **HTTPS** erwartet.  
Nach Korrektur auf `Scheme: https` und Eintragung der Domain in DSM generiert Synology Photos nun Freigabelinks mit `photos.meinedomain.de`.

---

## Kontext

### Netzwerkaufbau

```
Client → MikroTik (Routing, Firewall) → FritzBox (WAN-Gateway) → ISP
                    ↓
              HP EliteDesk (NPM, Docker) → Synology NAS (192.168.88.32)
```

- **NPM (Nginx Proxy Manager)** leitet externe Anfragen an `photos.meinedomain.de` an das NAS weiter.
- **DSM (Synology DiskStation Manager)** generiert Freigabelinks basierend auf der hinterlegten externen Adresse (`username.synology.me`).
- **Ziel:** Freigabelinks sollen automatisch `photos.meinedomain.de` nutzen, um SSL-Zertifikate (Wildcard `*.meinedomain.de`) zu nutzen.

---

## Symptome

- Externe Nutzer*innen erhalten beim Aufruf von `https://username.synology.me/photo/mo/sharing/...` den Fehler:  
`**err_ssl_unrecognized_name_alert**`.
- Freigabelinks enthalten **nicht** die gewünschte Domain `photos.meinedomain.de`.
- `curl -v https://photos.meinedomain.de/photo/mo/sharing/...` funktioniert (HTTP/2 200), aber nur mit manueller Portangabe (`:5001`).

---

## Hypothesen

1. **DNS-Problem:** `photos.meinedomain.de` zeigt nicht auf die richtige IP.
  - ✅ **Widerlegt:** DNS löst korrekt auf (CNAME → `username.synology.me` → öffentliche IP).
2. **SSL-Zertifikat:** Wildcard-Zertifikat (`*.meinedomain.de`) ist ungültig.
  - ✅ **Widerlegt:** `curl`-Test mit `photos.meinedomain.de` zeigt gültiges Zertifikat.
3. **NPM-Konfiguration:** Falsche Weiterleitung (HTTP vs. HTTPS).
  - ✅ **Bestätigt:** Scheme im **Details-Tab** war `http`, aber DSM erwartet **HTTPS** auf Port 5001.
4. **DSM-Einstellungen:** Falsche Domain hinterlegt.
  - ✅ **Bestätigt:** DSM nutzte `username.synology.me` als Standard-Domain für Freigabelinks.

---

## Untersuchung

### 1. `curl`-Test für `photos.meinedomain.de`

```bash
curl -v https://photos.meinedomain.de/photo/mo/sharing/12ab34cd
```

**Ergebnis:**

- `HTTP/2 200` → NPM und NAS funktionieren.
- SSL-Zertifikat (`*.meinedomain.de`) wird akzeptiert.

### 2. `curl`-Test für `username.synology.me`

```bash
curl -v https://username.synology.me/photo/mo/sharing/12ab34cd
```

**Ergebnis:**

- `TLSv1.3 alert, unrecognized name (624)` → Zertifikat passt nicht zu `username.synology.me`.

### 3. NPM-Konfiguration prüfen

- **Details-Tab:**
  - Scheme: `http` (❌ **Falsch!** DSM erwartet HTTPS auf Port 5001).
  - Forward Hostname/IP: `192.168.88.32`
  - Forward Port: `5001`
- **Custom Locations:**
  - `/photo/` → `https://192.168.88.32:5001/photo/` (✅ Korrekt)
  - `/mo/` → `https://192.168.88.32:5001/mo/` (✅ Korrekt)

**→ Inkonsistenz:** Details-Tab nutzt `http`, Custom Locations nutzen `https`.

---

## Root Cause

**NPM leitete Anfragen im Details-Tab per HTTP an Port 5001 weiter, obwohl DSM auf diesem Port HTTPS erwartet.**

- DSM generierte Freigabelinks mit `username.synology.me`, weil:
  - Die **externe Adresse in DSM** nicht auf `photos.meinedomain.de` gesetzt war.
  - Die **NPM-Konfiguration** inkonsistent war (HTTP vs. HTTPS).

---

## Lösung

### 1. NPM-Konfiguration korrigieren

- **Details-Tab:**
  - **Scheme:** `https` (nicht `http`)
  - Forward Hostname/IP: `192.168.88.32`
  - Forward Port: `5001`
- **Custom Locations:** Unverändert (bereits korrekt).

### 2. Domain in DSM eintragen

- **Pfad:** Systemsteuerung → Anmeldeportal → Anwendungen → Synology Photos bearbeiten
- **Domain:** `photos.meinedomain.de` eintragen (trotz Fehlermeldung "Ungültiger Domainname" speichern).
  - *Hinweis:* DSM prüft DNS nur beim Speichern. Danach ist die Domain hinterlegt.

---

## Test

1. **Neuen Freigabelink erstellen** in Synology Photos.
2. **Link prüfen:** Sollte jetzt `https://photos.meinedomain.de/photo/mo/sharing/...` enthalten.
3. **Extern testen:**
  ```bash
   curl -v https://photos.meinedomain.de/photo/mo/sharing/12ab34cd
  ```
  &nbsp;

---

## Lessons Learned

1. **Scheme und Port in NPM müssen konsistent sein:**
  - DSM erwartet auf Port 5001 **HTTPS**, nicht HTTP.
2. **DSM akzeptiert Domains trotz Fehlermeldung:**
  - DNS-Prüfung erfolgt nur beim Speichern. Danach bleibt die Domain hinterlegt.
  - Die benutzerdefinierte Domain allein genügt nicht. Zusätzlich muss der NPM-Proxyhost korrekt über HTTPS auf DSM-Port 5001 weiterleiten.
    Fehlerhafte Kombination: http → 5001
    Funktionierende Kombination: https → 5001
3. **Freigabelinks werden von DSM generiert:**
  - Nicht von Synology Photos selbst, sondern von DSM-Einstellungen. Nach Eintragen der benutzerdefinierten Domain erzeugt Synology Photos Links der Form:
    ```
    https://photos.meinedomain.de/mo/sharing/...
    ```

---

## Präventive Maßnahmen

- **Dokumentation:** NPM-Konfiguration (Scheme, Ports, Custom Locations) in `reverse-proxy.md` festhalten.
- **Test nach Änderungen:** Immer `curl -v` nutzen, um Weiterleitung und SSL zu prüfen.
- **DNS-Monitoring:** Regelmäßig prüfen, ob `photos.meinedomain.de` auf die aktuelle öffentliche IP zeigt.

---

## Verwandte Dokumente

- [Reverse Proxy Konfiguration](../02-services/reverse-proxy.md)
- [Netzwerk-Topologie](../00-overview/network-topology.md)
