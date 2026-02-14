# Incident: FRITZ!Box 7530 – Boot-Hang nach Firmware-Update

Datum: 2026-02
Status: Gelöst

---

## TL;DR

Nach einem Firmware-Update befand sich die FRITZ!Box 7530 in einem inkonsistenten Boot-Zustand. LEDs signalisierten Betriebsbereitschaft, jedoch antwortete das Gerät weder auf ARP noch auf Ping. Ursache war sehr wahrscheinlich ein nicht vollständig abgeschlossener Firmware-Neustart. Ein vollständiger Power-Cycle führte zur sauberen Initialisierung des Systems und zur erfolgreichen Finalisierung von FritzOS 8.21.

---

## Kontext

HomeLab-Architektur zum Zeitpunkt des Vorfalls:

Client → MikroTik (Routing + DNS) → FRITZ!Box 7530 → ISP

* MikroTik übernimmt Routing und DNS
* FRITZ!Box fungiert als Internet-Gateway
* Clients befinden sich im 192.168.88.0/24 Netz
* FRITZ!Box LAN: 192.168.178.1

---

## Symptome

* Kein Internetzugriff hinter dem MikroTik
* Clients erhielten gültige IP-Adressen im MikroTik-Subnetz
* MikroTik erreichbar
* FRITZ!Box nicht erreichbar per Ping
* Kein Zugriff auf 192.168.178.1
* Kein Zugriff auf 169.254.1.1
* FRITZ!Box-WLAN nicht sichtbar
* ARP-Eintrag für 192.168.178.1 „unvollständig"
* LEDs: Power/DSL und Info leuchteten

---

## Erste Hypothesen

* ISP-Ausfall
* Routing-Fehler im MikroTik
* DNS-Problem
* Hardwaredefekt der FRITZ!Box

---

## Untersuchung (Schritt-für-Schritt)

1. Client → MikroTik: funktional
2. `traceroute 1.1.1.1` endete mit `!H` im FRITZ!Box-Netz
3. ARP-Check zeigte „unvollständig“ für 192.168.178.1
4. Direktanschluss per LAN an FRITZ!Box
5. Statische IP-Konfiguration gesetzt (192.168.178.2/24)
6. Während des Bootens kontinuierlicher Ping auf 192.168.178.1
7. Nach vollständigem Neustart antwortete die FRITZ!Box wieder

---

## Technische Artefakte

### ARP-Ausgabe

```
192.168.178.1  (unvollständig)
```

### Traceroute

```
1 192.168.88.1
2 192.168.178.49 !H
```

Interpretation:

* MikroTik routing funktional
* Keine Antwort von der FRITZ!Box auf Layer 2

---

## Root Cause

Sehr wahrscheinlich unvollständig abgeschlossener Firmware-Update-Prozess.

Nach Neustart wurde FritzOS 8.21 finalisiert. Vor dem vollständigen Power-Cycle befand sich das Gerät offenbar in einem inkonsistenten Boot-Zustand, bei dem der Netzwerk-Stack nicht initialisiert wurde.

Kein Hinweis auf Hardwaredefekt.

---

## Lösung

* MikroTik vom Strom getrennt
* Direktverbindung zur FRITZ!Box
* Statische IP gesetzt
* Vollständiger Neustart durchgeführt
* System bootete korrekt und installierte Firmware 8.21 vollständig

Internetverbindung danach stabil.

---

## Lessons Learned

* LEDs bedeuten nicht automatisch, dass der Netzwerk-Stack läuft
* ARP „unvollständig“ ist ein starkes Indiz für Layer-2-Probleme
* Firmware-Updates können inkonsistente Zwischenzustände erzeugen
* Debugging immer von innen nach außen (Layer 2 → 3 → 7)

---

## Präventive Maßnahmen

* Automatische Firmware-Updates auf „nur informieren“ gestellt
* Incident strukturiert dokumentiert
* Monitoring-Konzept für Gateway-Erreichbarkeit geplant

---

## Status

☑ Gelöst
