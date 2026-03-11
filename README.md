
# AlarmMonitor Horoskop, Autobahn-Sperrungen & NINA-Warnungen

Eine Web-App für die Anzeige auf einem Alarmmonitor einer Rettungswache (DIVERA 24/7 Umgebung). Die Seite zeigt täglich um 05:00 Uhr ein neues, zufälliges Horoskop aus einer CSV-Datei, aktuelle Sperrungen auf ausgewählten Autobahnen und NINA-Warnungen für die konfigurierte Region. Zusätzlich wird ein QR‑Code für eine Feedback-Seite angezeigt.

Hinweis: Dieses Projekt ist ein Spaßprojekt, um Kolleg:innen mit etwas Humor und aktuellen Verkehrs- und Warninformationen durch den oft fordernden Dienstalltag zu begleiten.


## Features
- Tägliche, deterministische Zufallsauswahl aus `horoskop.csv`
- Tageswechsel um 05:00 Uhr (lokale Zeit); automatischer Reload genau zu diesem Zeitpunkt
- Dynamischer Wachenname per URL-Parameter (`?wachenname=...`)
- QR‑Code neben dem Horoskop, verlinkt auf `feedback.html`
- Feedback-Seite öffnet das E‑Mail‑Programm mit vorbefülltem Betreff und Nachricht
- **Anzeige aktueller Sperrungen auf den Autobahnen A3, A5, A60, A67, A671**
- **Filterung der Sperrungen nach relevanten Orten (z. B. Mainz, Wiesbaden, Rüsselsheim, etc.)**
- **NINA-Warnungen** für die konfigurierte Region (Standard: Kreis Groß-Gerau), mit automatischer Aktualisierung alle 2 Minuten und Detail-Lazy-Loading


## Projektstruktur
```
.
├── index.php          # Startseite (Horoskop + QR-Code + Autobahn-Sperrungen + NINA-Warnungen)
├── index.css          # Styles
├── index.js           # Logik: CSV laden, Auswahl, Auto-Reload, QR setzen, Sperrungen abrufen, NINA-Warnungen
├── horoskop.csv       # Datenquelle (Überschrift;Text)
├── feedback.html      # Feedback-Formular (mailto:)
└── feedback.css       # Styles für Feedback-Seite
```

## Wachenname per URL-Parameter

Die Startseite (`index.php`) akzeptiert einen GET-Parameter `wachenname`, um den Namen der Rettungswache dynamisch anzuzeigen:

```
https://example.com/index.php?wachenname=Gustavsburg
```

## Sperrungsanzeige & Filterung

Die App ruft aktuelle Sperrungen von der offiziellen Autobahn-API für die Autobahnen A3, A5, A60, A67 und A671 ab. Es werden nur Sperrungen angezeigt, die für die Rettungswache relevant sind. Die Relevanz wird anhand von Ortsnamen und Anschlussstellen im Titel, Untertitel oder der Beschreibung der Sperrung geprüft.

**Relevante Orte:**

```
const relevanteStichwoerter = [
  "Mainz", "Rüsselsheim", "Wiesbaden", "Darmstadt", "Raunheim", "Groß-Gerau",
  "Büttelborn", "Bischofsheim", "Mörfelden-Walldorf", "Mörfelden", "Walldorf",
  "Langen", "Weiterstadt", "Ginsheim-Gustavsburg", "Mainz -> Wiesbaden"
];
```

Nur Sperrungen, die einen dieser Begriffe enthalten, werden angezeigt. Die Anzeige erfolgt je Autobahn in einem eigenen Bereich.

**Angezeigte Informationen pro Sperrung:**
- Titel und Untertitel der Sperrung
- Beginn und Ende (aus der Beschreibung, hervorgehoben)

## NINA-Warnungen

Die App zeigt NINA-Warnungen (Bevölkerungsschutz) für die konfigurierte Region an. Die Daten werden über einen lokalen Proxy-Server (`/nina/...`) abgerufen, um CORS-Probleme zu umgehen.

- **Standard-Region:** Kreis Groß-Gerau (ARS: `064330000000`)
- **Aktualisierung:** alle 2 Minuten automatisch
- **Details:** Lazy-Loading per Klick auf "Details"
- **Konfiguration:** ARS kann per `localStorage` (`nina_ars`) angepasst werden

## CSV-Format
- Trennzeichen: Semikolon `;`
- Erste Zeile (Header) wird erkannt und übersprungen
- Jede weitere Zeile: `Überschrift;Text`

Beispiel:
```
Überschrift;Text
Tageshoroskop;Heute ist euer Tag! Zumindest bis der Melder geht.
Spruch des Tages;Ein guter Tag, um den Kollegen daran zu erinnern, dass 'gleich zurück' relativ ist.
```

## Funktionsweise der Auswahl
- Die Auswahl ist „willkürlich", aber pro Tag stabil (deterministisch)
- Der Tageswechsel findet um 05:00 Uhr lokaler Zeit statt (nicht um Mitternacht)
- Ein Timer lädt die Seite genau um 05:00 neu, sodass automatisch das neue Horoskop erscheint

## Voraussetzungen
- **PHP** (für `index.php` mit dem `wachenname`-Parameter)
- **Node.js** (empfohlen, für den NINA-Proxy-Server)

## Lokal starten

### NINA-Proxy (empfohlen)
Für die NINA-API wird ein lokaler Proxy-Server benötigt, um CORS-Probleme zu umgehen:

```bash
node server.js
```
Öffne dann: http://localhost:8000

### Alternativen (ohne NINA)
- Python (3.x):
  ```bash
  python3 -m http.server 8000
  ```
- Node (serve):
  ```bash
  npx serve
  ```

## Feedback-Konfiguration
- In `feedback.html` wird ein `mailto:` Link erzeugt
- Passe die Zieladresse bei Bedarf in `feedback.html` an (Variable `to`, aktuell `feedback@example.com`)

## Anpassungen
- QR‑Bildgröße: `index.php` (`#feedback-qr` width/height)
- Tageswechselzeit (aktuell 05:00): `dayKeyWithCutoff(5)` und `scheduleDailyReload(5)` in `index.js`
- Relevante Autobahnen und Orte: `showCurrentClosures()` Aufrufe und `relevanteStichwoerter` in `index.js`
- NINA-Region (ARS): `NINA_DEFAULT_ARS` in `index.js`
- Layout: Bootstrap-Grid in `index.php`

## Lizenz
MIT License – siehe [LICENSE](./LICENSE).

## Hinweis / Disclaimer
- Dieses Projekt dient ausschließlich der Unterhaltung. Es stellt keine medizinische Beratung dar und ist nicht für den klinischen Einsatz geeignet.
- Inhalte können ungenau, satirisch oder unvollständig sein.
- Keine Verbindung zu DIVERA 24/7: Dieses Projekt steht in keinem Zusammenhang mit, wird nicht unterstützt, gesponsert oder autorisiert von DIVERA 24/7 oder deren Betreiber:innen.
- Alle genannten Namen und Marken sind Eigentum der jeweiligen Rechteinhaber:innen.

English:
- This project is for entertainment purposes only. It does not constitute medical advice and is not intended for clinical use.
- Content may be inaccurate, satirical, or incomplete.
- No affiliation with DIVERA 24/7: This project is not affiliated with, endorsed by, sponsored by, or authorized by DIVERA 24/7 or its operators.
- All product names, logos, and brands are property of their respective owners.
