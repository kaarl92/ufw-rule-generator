# UFW Rule Generator

[![Live-Demo](https://img.shields.io/badge/Live--Demo-GitHub%20Pages-4f9cf9)](https://kaarl92.github.io/ufw-rule-generator/)
[![License: MIT](https://img.shields.io/badge/License-MIT-3ad29f.svg)](LICENSE)
[![No dependencies](https://img.shields.io/badge/dependencies-none-8b93a3)](#)

Ein eigenständiges Web-Tool, das aus per Formular gewählten Diensten, Netzen und
Optionen ein fertiges **UFW-Firewall-Skript** erzeugt. Läuft komplett offline im
Browser — kein Server, keine Installation, keine externen Abhängigkeiten.

**[→ Live-Demo öffnen](https://kaarl92.github.io/ufw-rule-generator/)**

## Start

Entweder die [Live-Demo](https://kaarl92.github.io/ufw-rule-generator/) nutzen, oder
`docs/index.html` per Doppelklick lokal im Browser öffnen. Fertig.

Alle Eingaben (Netze, Dienstauswahl, eigene Dienste, Optionen) werden lokal im
Browser gespeichert und sind beim nächsten Öffnen wieder da.

## Bedienung

### 1. Quell-Netze (Profile)

Eigene Netze als wiederverwendbare Profile pflegen (z. B. `192.168.0.0/24`).

- **Häkchen** = Netz ist aktiv und gilt für alle gewählten Dienste.
- Der **Name** landet als Suffix im Regel-Kommentar (z. B. `SSH/SFTP LAN`).
- Ohne aktives Netz gilt jede Regel für `Anywhere`.

### 2. Dienste

Über 50 Standard-Dienste in Kategorien (Remote, Web, Datei, Netz-Infrastruktur,
Mail, Datenbanken, VPN, Virtualisierung, Media/Smart Home, Self-Hosted, Monitoring).
Ein Klick auf eine Kachel wählt den Dienst aus; er wird automatisch mit allen
aktiven Netzen kombiniert.

- **Suche** filtert nach Name, Port oder Kommentar (z. B. `ssh`, `8096`, `plex`).
- **Alle / Keine** wählt bzw. leert die komplette Liste.
- **Richtung** (Dropdown): `↓ Inbound` (Standard), `↑ Outbound`, `↕ Beides` —
  gilt global für alle gewählten Dienste.

#### Eigene Dienste

Unten Name, Port (oder Range/mehrere mit Komma: `80,443`), Protokoll und Kategorie
eintragen → **+ speichern**. Eigene Dienste bleiben dauerhaft erhalten (Browser-
Speicher) und lassen sich per 🗑 auf der Kachel wieder entfernen.

### 3. Einmalige Regeln

Für Sonderfälle ohne Speicherung. Pro Zeile: Richtung, Quelle/Ziel, Port, Protokoll,
Kommentar. Bei Richtung `Out` wird aus „Quelle" automatisch „Ziel" (UFW dreht die
Semantik um).

### 4. Skript-Optionen

| Option | Wirkung |
|--------|---------|
| ufw logging on | aktiviert UFW-Logging |
| Reset-Block | auskommentierter `ufw --force reset` zum sauberen Neustart |
| ufw enable + status | `ufw enable` und Status-Ausgaben am Ende |
| Abschnitts-Kommentare | `# SSH …`-Überschriften pro Dienst |
| Richtung explizit | schreibt `allow in …` statt nur `allow …` bei Inbound |
| Shebang + set -euo pipefail | nur beim `.sh`-Download vorangestellt |

### 5. Ausgabe

Rechts erscheint das Skript live. **Kopieren** legt es in die Zwischenablage,
**.sh laden** speichert es als `ufw-rules.sh` (optional mit Shebang).

## Richtungs-Logik (Inbound / Outbound)

- **Inbound (klassisch):** `sudo ufw allow from <netz> to any port <port> proto <proto>`
- **Inbound (explizit):** `sudo ufw allow in from <netz> to any port <port> …`
- **Outbound:** `sudo ufw allow out to <netz> port <port> proto <proto>`
  (das Netz ist hier das **Ziel**; ohne Netz → `to any`)
- **Beides:** erzeugt je eine In- und eine Out-Zeile

> Hinweis: UFW erlaubt standardmäßig (`default allow outgoing`) ausgehenden
> Verkehr bereits komplett. Explizite Outbound-`allow`-Regeln braucht man erst,
> wenn auf `default deny outgoing` umgestellt wird.

## Beispiel-Ausgabe

```bash
sudo ufw logging on
# SSH / SFTP
sudo ufw allow from 192.168.0.0/24 to any port 22 proto tcp comment 'SSH/SFTP LAN'
# HTTP / HTTPS
sudo ufw allow from 192.168.0.0/24 to any port 80 proto tcp comment 'HTTP LAN'
sudo ufw allow from 192.168.0.0/24 to any port 443 proto tcp comment 'HTTPS LAN'
...
# Aktivieren und prüfen
sudo ufw enable
sudo ufw status numbered
sudo ufw status verbose
```

## Anwenden auf dem Zielsystem

```bash
# Skript prüfen (immer erst lesen!)
less ufw-rules.sh

# ausführbar machen und starten
chmod +x ufw-rules.sh
sudo ./ufw-rules.sh
```

**Wichtig:** Vor `ufw enable` sicherstellen, dass eine SSH-Regel für dein
Admin-Netz drin ist — sonst sperrst du dich per SSH aus. Bei Remote-Zugriff
idealerweise eine zweite Sitzung offen halten.

## Hinweise & Grenzen

- Einige Dienste teilen sich Ports (z. B. Pi-hole/AdGuard → 53+80; Cockpit/
  Prometheus/Grafana → 9090/3000). Werden überlappende gleichzeitig gewählt,
  entstehen doppelte UFW-Regeln. UFW akzeptiert das und überspringt Duplikate
  mit einer Meldung.
- Ports basieren auf IANA-Standard und aktueller Homelab-Praxis. Eigene/geänderte
  Ports über „Eigene Dienste" oder „Einmalige Regeln" abbilden.
- Reine Client-App, keine Datenübertragung nach außen — alle Daten bleiben im
  Browser (`localStorage`).

## Projektstruktur

```
ufw-rule-generator/
├── docs/
│   └── index.html   # das Tool (alles in einer Datei) — wird per GitHub Pages ausgeliefert
├── LICENSE
├── README.md
└── .gitignore
```

## Mitwirken

Issues und Pull Requests sind willkommen — z. B. für weitere Standard-Dienste,
UI-Verbesserungen oder Bugfixes. Da das Tool bewusst eine einzelne,
abhängigkeitsfreie HTML-Datei bleibt, bitte keine Build-Tools oder Frameworks
einführen.

## Lizenz

[MIT](LICENSE)

---

Erstellt für Kaarl · Homelab (MS-A2 / Proxmox)
