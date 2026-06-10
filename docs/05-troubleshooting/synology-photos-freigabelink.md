# Synology Photos: Freigabelinks – Diagnose und Lösungen

## Problem: Freigabelinks nutzen falsche Domain oder Scheme

Freigabelinks von Synology Photos enthalten entweder:

- Die falsche Domain (`username.synology.me` statt `photos.meinedomain.de`).
- Das falsche Scheme (`http` statt `https`).

---

## Diagnose

### 1. NPM-Konfiguration prüfen

- **Scheme:** Muss zu DSM-Port passen.
  - Port 5001 → `**https**` (DSM erwartet HTTPS).
  - Port 5000 → `http` (falls HTTP genutzt wird).
- **Test:**
  ```bash
  curl -v https://photos.meinedomain.de/photo/mo/sharing/...
  ```
  - **Erwartet:** `HTTP/2 200` (kein 400 Bad Request).
  - **Fehler 400?** → Scheme in NPM prüfen (wahrscheinlich `http` statt `https`).

### 2. DSM-Einstellungen prüfen

- **Pfad:** Systemsteuerung → Anmeldeportal → Anwendungen → Synology Photos
- **Domain:** Sollte `photos.meinedomain.de` enthalten.
  - Falls Fehlermeldung "Ungültiger Domainname": 
    - Temporär **A-Record** für `photos.meinedomain.de` auf die aktuelle öffentliche IP setzen.
    - Domain in DSM speichern.
    - DNS-Eintrag zurück auf CNAME (`username.synology.me`) ändern.

### 3. DNS-Eintrag prüfen

- **A-Record oder CNAME?**
  - `photos.meinedomain.de → CNAME → username.synology.me` (empfohlen, da dynamisch).
  - `photos.meinedomain.de → A → <öffentliche-IP>` (nur falls CNAME nicht funktioniert).
- **Test:**
  ```bash
  nslookup photos.meinedomain.de
  ```
  → Sollte auf die öffentliche IP der FritzBox zeigen.

---

## Lösungen

### Lösung 1: NPM-Konfiguration korrigieren

- **Details-Tab:**
  - Scheme: `**https**` (wenn Port 5001 genutzt wird).
  - Forward Hostname/IP: `192.168.88.32` (NAS-IP).
  - Forward Port: `5001`.
- **Custom Locations:**
  - `/photo/` → `https://192.168.88.32:5001/photo/`
  - `/mo/` → `https://192.168.88.32:5001/mo/`

### Lösung 2: Domain in DSM eintragen

1. Gehe zu **Anmeldeportal → Anwendungen → Synology Photos bearbeiten**.
2. Trage `**photos.meinedomain.de**` als Domain ein.
3. Speichere **trotz Fehlermeldung** (DNS-Prüfung nur beim Speichern).

### Lösung 3: Manuelle Link-Anpassung (Workaround)

- Ersetze in Freigabelinks manuell:
  ```bash
  https://username.synology.me/photo/mo/sharing/... 
  → https://photos.meinedomain.de/photo/mo/sharing/...
  ```
  - *Hinweis:* Nicht nachhaltig, aber schnell umsetzbar.

---

## Häufige Fehler und Ursachen


| Fehler                            | Ursache                                                  | Lösung                                               |
| --------------------------------- | -------------------------------------------------------- | ---------------------------------------------------- |
| `400 Bad Request`                 | NPM nutzt `http` für Port 5001 (DSM erwartet `https`).   | Scheme in NPM auf `https` ändern.                    |
| `err_ssl_unrecognized_name_alert` | Zertifikat passt nicht zu `username.synology.me`.        | Nutze `photos.meinedomain.de` (Wildcard-Zertifikat). |
| DNS-Fehler                        | `photos.meinedomain.de` zeigt nicht auf die richtige IP. | DNS-Eintrag prüfen (CNAME oder A-Record).            |


---

## Verwandte Incidents

- [2026-06-10: Synology Photos Freigabelinks mit falscher Domain](./2026-06-10-synology-photos-freigabelinks.md)

---

## Test-URLs

- [https://photos.meinedomain.de/](https://photos.meinedomain.de/)
- [https://photos.meinedomain.de/photo/](https://photos.meinedomain.de/photo/)
- [https://photos.meinedomain.de/mo/](https://photos.meinedomain.de/mo/)
