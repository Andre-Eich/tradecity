# TradeCity

## Projektbeschreibung
TradeCity ist ein browserbasiertes Handels-RTS-Spiel. Das Projekt zeigt eine interaktive Weltkarte mit einer zentralen Hauptstadt, KI-Städten, Dörfern und Handelsrouten. Spielerinteraktionen erfolgen über Maus-Hover und Klicks auf Kartenpositionen, wodurch Städte ausgewählt und Informationen zu Wirtschaft, Bevölkerung und militärischen Aktionen angezeigt werden.

## Dateistruktur
- `index.html` — Haupt-HTML-Datei mit Spieloberfläche, Karten-Canvas, SVG-Ebenen und Panel-Layout.
- `style.css` — Stile für die Karte, Panels, City-Hover, Rahmen, Layout und UI-Elemente.
- `game.js` — komplette Spiel-Engine, Stadt- und Kartenlogik, Render-Pipeline, KI-Initialisierung, Auswahl / Interaktion, Handels- und Angriffsmechaniken.
- `map.png` — externe Karten-Hintergrundgrafik, die als Spielwelt-Visual verwendet wird.
- `city-placer.html` — Hilfstool zur interaktiven Platzierung von Städten auf der Map und Generierung von XR/YR-Werten.
- `README.md` — kurze Projektbeschreibung.
- `tradecity_v7(5).html` — ursprüngliche oder archivierte HTML-Version des Projekts.
- `.git/` — Git-Repository-Metadaten.
- `Bilder etc/` — zusätzlicher Medienordner für Assets.

## Spielmechaniken

### Karte & Städte
- Die Karte wird als statisches Hintergrundbild mit einer Overlay-Ebene aus Stadt-Hotspots dargestellt.
- Städte haben Eigenschaften wie `population`, `tradePotential`, `isEnemy`, `isBase` und relative Karten-Koordinaten (`xr`, `yr`).
- Städte werden in verschiedene Kategorien eingeteilt: Basis, große KI-Städte, mittlere KI-Städte und Dörfer.
- Interaktion erfolgt per Klick: Städte werden ausgewählt, Informationen angezeigt und UI-Panels geöffnet.
- Es gibt Logik für Straßenverbindungen (`state.roads`) sowie berechnete Kurvenpunkte für die Visualisierung von Routen.
- Das Spiel verwaltet Handelswaren, Gold, Bevölkerung, Armee, Angriffe, Verstärkungen und einzelne Spieler-/KI-Zustände.

### Bevölkerungs- und Schichten-System
Drei unabhängige Bevölkerungsschichten, jede mit eigenem Einkommen, Wachstum und Steuersatz:
- **Einfache Leute** (`state.schichten.einfache`): Arbeiter, Bauern, Knechte. Stellen Bauarbeiter, Minenarbeiter und Bauern.
- **Mittelschicht** (`state.schichten.mittel`): Händler und Handwerker. Benötigen Ressourcen (Holz, Werkzeuge) zum Produzieren von Handelswaren.
- **Oberschicht** (`state.schichten.ober`): Großhändler, Akademiker, Adlige. Höchste Steuereinnahmen.

Jede Schicht hat: `pop`, `income` (G/min, wird pro Tick berechnet), `taxLevel` (0=Niedrig / 1=Mittel / 2=Hoch).  
Steuern werden alle 5 Minuten abgerechnet (`TAX_INTERVAL = 300`).

Steuersätze (`TAX_RATES`):
- Einfache Leute: 0.5% / 1.0% / 2.0%
- Mittelschicht: 2.0% / 3.5% / 5.5%
- Oberschicht: 5.0% / 12.0% / 22.0%

### Arbeiter-System (Einfache Leute)
Die Einfachen Leute verteilen sich auf Rollen in `state.workers`:
- `farmers` — Bauern (produzieren Nahrung, nur auf Ackerplätzen 100% produktiv)
- `miners` — Minenarbeiter (produzieren Rohstoffe)
- `builders` — Bauarbeiter (auf Baustellen aktiv)
- `idle` — Jobsuchende (Startzustand für alle neuen Bürger)

**Alle neuen Bürger starten als `idle`.** Verteilung läuft über `updateWorkerDistribution()`:
- Caps werden enforzt: Überschuss-Arbeiter → `idle`
- 1/sek: 1 Jobsuchender sucht die Stelle mit den meisten freien Plätzen
- Bei >10 freien Stellen in einem Beruf: 3–5 Jobsuchende/sek statt 1
- `state._jobSeekTimer` akkumuliert Sekunden für den Jobsuche-Tick

### Bauarbeiter-Mechanik (implementiert)
- **Bauleistung**: 1 Bauarbeiter = 1 Bauleistung/min
- **Bauzeit** = Bauaufwand / Anzahl aktiver Bauarbeiter (in Minuten)
- **Draw-Mechanik**: Wenn ein Bau in Auftrag gegeben wird, werden jede Sekunde 1 Arbeiter aus den Einfachen Leuten (erst `idle`, dann `farmers`) in `workers.builders` verschoben und in `item.reservedWorkers` erfasst.
- **Rückgabe**: Nach Fertigstellung eines Baus werden alle `item.reservedWorkers` zurück nach `workers.idle` überführt und suchen sich automatisch einen neuen Job (via `updateWorkerDistribution`).
- **Fortschritt**: `remaining -= (effWorkers * xpFactor / 60) * seconds`

### Ressourcen (aktuell in game.js)
- **Gold** (`state.gold`) — Hauptwährung
- **Nahrung** (`state.food`) — Versorgung für Bevölkerung und Armee
- **Rohstoffe** (`state.goods`) — produziert durch Minen/Bauern; max. durch Lager begrenzt
- **Handelswaren** (`state.wealth`) — produziert durch Handwerker/Händler
- **Holz** (`state.wood`) — noch nicht in game.js implementiert (UI-Platzhalter vorhanden)
- **Werkzeuge** (`state.tools`) — noch nicht in game.js implementiert (UI-Platzhalter vorhanden)

## Technische Besonderheiten

### Rendering & Layout
- Die Karte nutzt eine Kombination aus Canvas und SVG-Overlays (`#mapCanvas`, `#territory`, `#roads`) für dynamische Darstellung und Interaktion.
- Externe Map-Grafik (`map.png`) wird im Hintergrund geladen; bei Erfolg wird eine spezielle CSS-Klasse `body.map-image-loaded` gesetzt.
- Die Stadt-Platzierung verwendet relative Koordinaten (`xr`, `yr`) und wandelt sie in Pixel um, abhängig von der Containergröße.

### UI-Panel-Struktur
- **Panel 1 (Dein Reich)**: 65% der Gesamthöhe, fest – kein Accordion-Toggle. Enthält 4 Stat-Karten (Einkommen, Einwohner, Gebäude, Land & Vorräte) + KI-Städte-Grid.
- **Panel 2 (Bauwesen)**: Im DOM vorhanden (für JS-Kompatibilität), aber `display:none`.
- **Panel 3 (Fremde Städte)**: 35% der Gesamthöhe, fest.
- Höhenaufteilung via `#panel1.expanded { flex: 65 1 0 }` / `#panel3.expanded { flex: 35 1 0 }` (ID-Spezifität schlägt Klassen-Spezifität).

### CSS-Besonderheiten
- **`:has()` Selector**: Gebäude-Kacheln (`.bldg-ic-cell`) sind standardmäßig `display:none`; werden sichtbar wenn `.b-status.done` enthalten ist oder `.in-queue` gesetzt ist.
- **`data-lvl` Attribut**: Steuersatz-Badges nutzen `[data-lvl="0/1/2"]` für farbliche Kodierung (grün/gold/rot).
- **Ressourcen-Grid**: 2×2-Grid für Rohstoffe, Handelswaren, Holz (inaktiv), Werkzeuge (inaktiv) via CSS Grid.

### Hilfstool
- `city-placer.html` erzeugt per Klick XR/YR-Koordinaten und exportierbare Stadtdaten.

## Geplante Änderungen

### Neue Ressourcen-Logik
Ziel: Produktionskette aus Rohstoffen → Zwischenprodukte → Handelswaren

| Gebäude | Input | Output | Produzent |
|---|---|---|---|
| Mine | — | Stein + Erz | Einfache Leute (Minenarbeiter) |
| Sägewerk | — | Holz | Einfache Leute (Waldarbeiter) |
| Schmiede | Stein + Erz | Werkzeuge | Einfache Leute (Schmiede) |
| Handwerker (Mittelschicht) | Holz + Werkzeuge | Handelswaren | Mittelschicht |

- Mine produziert künftig **Stein** und **Erz** (derzeit nur generische Rohstoffe)
- Sägewerk ist ein neues Gebäude (noch nicht in game.js)
- Schmiede ist ein neues Gebäude (noch nicht in game.js)
- Handwerker der Mittelschicht benötigen Holz + Werkzeuge im Stadtlager

### Bausystem
**Formel**: Bauzeit (min) = Bauaufwand / Bauleistung/min  
**Bauleistung/min** = Anzahl zugewiesener Bauarbeiter (1 Arbeiter = 1 Bauleistung/min)  
**Draw-Rate**: 1 Arbeiter/Sekunde aus Einfachen Leuten → Baustelle (bereits implementiert)

### Gebäude-System (finale Version)

Einheitlicher Bauaufwand: Stufe 1 = 50, Stufe 2 = 150, Stufe 3 = 250. Einzelgebäude = 50.  
Lagerausbau wurde gestrichen.

#### 🪚 Sägewerk (neu)
| Stufe | Kosten | Bauaufwand | Produktion | Arbeiter |
|---|---|---|---|---|
| 1 | 50G · 20 Stein | 50 | +5 Holz/min | 5 |
| 2 | 100G · 30 Stein | 150 | +12 Holz/min | 10 |
| 3 | 200G · 50 Stein | 250 | +25 Holz/min | 20 |

#### ⛏ Mine (Stein & Erz)
| Stufe | Kosten | Bauaufwand | Produktion | Arbeiter |
|---|---|---|---|---|
| 1 | 100G | 50 | +5 Stein & Erz/min | 5 |
| 2 | 150G · 20 Holz | 150 | +12 Stein & Erz/min | 10 |
| 3 | 250G · 40 Holz | 250 | +25 Stein & Erz/min | 20 |

#### 🔨 Schmiede (neu)
| Stufe | Kosten | Bauaufwand | Produktion | Arbeiter |
|---|---|---|---|---|
| 1 | 100G · 20 Holz · 30 Stein | 50 | +3 Werkzeuge/min | 3 |
| 2 | 200G · 40 Holz · 60 Stein | 150 | +8 Werkzeuge/min | 6 |
| 3 | 350G · 80 Holz · 100 Stein | 250 | +15 Werkzeuge/min | 10 |

#### ⛪ Kirche (3 Stufen)
| Stufe | Kosten | Bauaufwand | Effekte |
|---|---|---|---|
| 1 | 150G · 30 Holz · 50 Stein | 50 | +Ruf · +Wachstum Einfache Leute |
| 2 | 200G · 50 Holz · 80 Stein | 150 | +Ruf · +Wachstum Einfache Leute |
| 3 | 300G · 80 Holz · 120 Stein | 250 | +Ruf · +Wachstum Einfache Leute |

#### 🏪 Marktplatz (3 Stufen)
| Stufe | Kosten | Bauaufwand | Effekte |
|---|---|---|---|
| 1 | 100G · 30 Holz · 30 Stein | 50 | +Händler-Einkommen · +Max Mittelschicht |
| 2 | 150G · 50 Holz · 50 Stein | 150 | +Händler-Einkommen · +Max Mittelschicht |
| 3 | 250G · 80 Holz · 80 Stein | 250 | +Händler-Einkommen · +Max Mittelschicht |

#### ⚔ Kaserne (3 Stufen)
| Stufe | Kosten | Bauaufwand | Armee-Kapazität |
|---|---|---|---|
| 1 | 200G · 20 Holz · 80 Stein | 50 | 50 Soldaten |
| 2 | 250G · 40 Holz · 100 Stein | 150 | 100 Soldaten |
| 3 | 350G · 60 Holz · 120 Stein | 250 | 200 Soldaten |

#### 🌾 Ackerfelder
3 Felder · je 3 Stufen · Stufe 0: 20 Plätze · Stufe 1: 50 · Stufe 2: 100 · Stufe 3: 150 · Baukosten je Stufe: 30G · Aufwand 50

### Einfache Leute – Arbeitszuweisung
- Neue Bürger starten als Bauern mit 20% Produktivität und werden als "jobsuchend" markiert
- Bauarbeiter werden zuerst abgezogen (fest bis Bau fertig, dann zurück zu jobsuchend)
- Rest wird fließend (1/sek) nach Platz-Verhältnis verteilt:
  - Bauern = Verfügbare × (Ackerplätze / Gesamtplätze)
  - Miner = Verfügbare × (Mineplätze / Gesamtplätze)
  - Säger = Verfügbare × (Sägewerkplätze / Gesamtplätze)
- Überschuss (mehr Leute als Plätze) → jobsuchend, 20% Produktivität
- Priorität: Spieler kann pro Gebäude +30% Arbeiteranteil aktivieren (im Gebäude-Popup)

### Mittelschicht – Wachstum & Jobs
- Start: 10 Mittelschicht-Bürger, alle als Händler
- Kein eigenes Gebäude für Mittelschicht
- Wachstum:
  - Nahrung > 50% → +1%/min Geburten
  - Einkommen > 50% des Schicht-Durchschnitts → zufällig +1 Zuzug alle 2–3 min
- Jobs: Händler und Handwerker, max 50/50 Aufteilung
- Rohstoffe (Holz oder Stein & Erz) vorhanden → Händler werden zu Handwerkern (6/min)
- Rohstoffe leer → Handwerker werden nach 1 min wieder zu Händlern
- Handwerker geben Einfachen Leuten 1–5 Arbeitsplätze

### Nächste Implementierungsschritte
1. Mine auf Stein+Erz-Output umstellen (derzeit noch Rohstoffe-Generik)
2. Sägewerk-Holzproduktion an Spieltick koppeln (Rate: 5/12/25 Holz/min je Stufe)
3. Schmiede-Produktion anschalten (Input: Stein+Erz → Output: Werkzeuge)
4. Handwerker-Produktion an Holz+Werkzeuge-Verfügbarkeit koppeln
5. UI-Ressourcenkarten für Holz und Werkzeuge aktivieren (Platzhalter bereits in index.html)

## Oberschicht-System

### Regionaler Ruf
- Neue Ressource, Startwert: 50/100
- Wird oben im Panel neben "Mein Reich" angezeigt
- Je höher der Ruf desto bessere Einheiten bewerben sich
- Beeinflusst Qualität (Sterne) der Bewerber

### Oberschicht
- Startet mit 0 Einheiten
- Zwei Einheitentypen: Großhändler und Ritter
- Qualität wird in Sternen angezeigt: 0.5 bis 5.0 (in 0.5 Schritten)
- Formel: `Sterne = Math.round((wert / 100) × 10) / 2`

### Großhändler
- Handelserfahrung: 0–100 (wird bei aktiven Handelsrouten gebunden)
  - Dorf: 0–25 XP gebunden (zufällig generiert)
  - Stadt: 25–50 XP gebunden (zufällig generiert)
  - Große Stadt: 50–100 XP gebunden (zufällig generiert)
  - Nicht genug XP → Handelsroute kann nicht gestartet werden
- Handelstrupps: 1–5
- Einkommen: 1000–5000 G/min (in Sternen angezeigt)

### Ritter
- Militärerfahrung: 0–100 (wird bei Angriffen gebunden)
  - Benötigte XP abhängig von Angriffsgröße und Zielgröße
  - Nicht genug XP → Angriff kann nicht gestartet werden
  - Mehrere Ritter können XP kombinieren
- Soldaten: 100–1000 (eigene Ritter-Armee, getrennt von Basis-Armee)
- Einkommen: 1000–5000 G/min (in Sternen angezeigt)
- Bei Angriff: Ritter-Armee + Basis-Armee kämpfen gemeinsam

### Bewerbersystem (Stadt-Popup)
- Rechte Box 200px breit im Stadt-Popup
- Stadtkarte wird an den linken Rand geschoben
- Obere Hälfte: Bewerberkarten
  - Neue Bewerberkarte alle 30–60 Sek (zufällig)
  - Max 4 Bewerberkarten gleichzeitig
  - Timer pausiert wenn 4 Karten vorhanden
  - Jede Karte bleibt 60–120 Sek, dann verschwindet sie
  - Spieler kann Annehmen oder Ablehnen
  - Bewerberkarte zeigt: Typ, alle 3 Werte in Sternen, Countdown
- Untere Hälfte: Aktive Oberschicht-Einheiten als Karten
  - Großhändler-Karten mit allen Werten in Sternen
  - Ritter-Karten mit allen Werten in Sternen

## Aktuelle Version
- **Status:** Entwicklungsversion / kein formaler Release-Tag
- **Version:** `0.1-dev`
- **Bemerkung:** Das Projekt ist aktiv in Arbeit und wird momentan über `index.html`, `game.js` und `style.css` gepflegt.

## Nahrungslogik (Stand 2026-04-14)

### Startwerte
- Start-Nahrung: `300`
- Kennzahl für Versorgung und Wachstum: `foodPerCitizen() = state.food / totalCitizens()`

### Verbrauch
`foodConsumption()` → pro Sekunde (×60 = pro Minute):
- Einfache Leute: `1 Nahrung/min`
- Mittelschicht: `5 Nahrung/min`
- Oberschicht: `10 Nahrung/min`
- Armee: `1 Nahrung/min`

### Produktion
`foodProduction()` → pro Sekunde (×60 = pro Minute):
- Bauern ohne Acker: `1.5 Nahrung/min`
- Ackerfeld Stufe 0: `1.5 Nahrung/min` je Bauer
- Ackerfeld Stufe 1: `2.5 Nahrung/min` je Bauer
- Ackerfeld Stufe 2: `3.0 Nahrung/min` je Bauer
- Ackerfeld Stufe 3: `3.5 Nahrung/min` je Bauer
- Nur `workers.farmers` produzieren Nahrung; `jobsuchend` produziert nicht

### Versorgungsbilanz
- `foodStorageDeltaPerMin()` nutzt jetzt die echte Netto-Bilanz `foodBalance() * 60`
- Nahrung wird nicht mehr in groben Stufen `+10/-20` bewegt, sondern mit der realen Produktions-/Verbrauchsdifferenz

### Wachstumsschwellen
Kennzahl: `fpc = state.food / totalCitizens()` (Nahrung pro Bürger, alle 3 Schichten)
- Positiver Bereich: `fpc > 2.0`
- Negativer Bereich: `fpc < 2.0`
- Starker negativer Bereich: `fpc < 0.75`

### Wachstum Einfache Leute
- Geburten: `(random(1–5) + fpc) / min` wenn `fpc > 2`
- Steuermodifikator auf Geburten: Niedrig `×2.0` / Normal `×1.0` / Hoch `×0.5`
- Abwanderung: `−5%/min` bei `0.75 ≤ fpc < 2`
- Abwanderung: `−15%/min` bei `fpc < 0.75`

### Wachstum Mittelschicht
- Zuzug: `+1/min` wenn `fpc > 2`
- Abwanderung: `−10%/min` wenn `fpc < 0.75`

### Wachstum Oberschicht
- `state.schichten.ober.pop = state.nobles.length` (direkt aus Bewerbersystem)

## Ackerlogik (Stand 2026-04-14)

### Jobs pro Feld
- Startzustand: `15 Bauern-Jobs`
- Ausbau Stufe 1: insgesamt `25 Jobs` (`+10`)
- Ausbau Stufe 2: insgesamt `45 Jobs` (`+20`)
- Ausbau Stufe 3: insgesamt `75 Jobs` (`+30`)

### Wichtige Regel
- Acker-Ausbau gibt zusätzliche Jobs
- Die Arbeiterverteilung bleibt slot-basiert: nur verfügbare Jobs werden besetzt, der Rest bleibt `jobsuchend`

## Schmiede-Logik (Stand 2026-04-14)

### Schmiede Stufe 1
- `10 Schmied-Jobs`
- `50 Lagerplätze` für Werkzeuge
- `10 Werkzeuge/min` bei voller Versorgung

### Produktion & Fallback
- Schmiede nutzt eigene Arbeiterrolle `workers.smiths`
- Volle Produktion nur wenn Holz und Stein/Erz vorhanden sind
- Pro aktivem Schmied werden `1 Holz/min` und `1 Stein/Erz/min` verbraucht
- Ohne diese Rohstoffe produziert die ganze Schmiede nur `2 Werkzeuge/min`
- Werkzeugkarte nutzt jetzt das echte Schmiede-Lager statt festen UI-Werten

## Aktuelle Session (April 2026, diese Session)

### Mittelschicht – idle-basiertes Jobsystem
- Alle Mittelschicht-Bürger starten als `idle` (neu: `state.schichten.mittel.idle`)
- Alle 10 Sek findet 1 Jobsuchender einen Job: Handwerker wenn Slots frei, sonst Händler
- Händler = pop − handwerker − idle (abgeleitet, nie gespeichert)
- Handwerker-Schonzeit: 60 Sek bevor Rohstoffmangel zur Entlassung führt (`_handwerkerGraceTimer`)
- Handwerker-Slots = floor(pop/2), nur wenn Holz > 0 UND Stein/Erz > 0

### Sägewerk (implementiert)
- `SAWMILL_WORKERS = [10, 20, 40]` Holzarbeiter je Stufe
- Holzarbeiter produzieren 2 Holz/min je Arbeiter
- Holzlager: 100 / 200 / 400 je Stufe
- Holz-Anzeige im Panel aktiv (woodCurrent, woodMax, woodFill, woodRate)

### Mine (implementiert)
- `MINE_WORKERS = [10, 20, 40]` Minenarbeiter je Stufe (war 5/10/20)
- `MINE_STORAGE = [100, 200, 400]` Lager-Max Stein/Erz je Stufe
- Produktion: 2 Stein/Erz je Minenarbeiter pro Ø 7.5 Sek
- Stein/Erz ist eine einzelne Ressource (`rohstoffLager.stein`)
- Rohstoff-Karte im Panel zeigt Stein/Erz + Rate

### Handwerker-Produktion (implementiert)
- Verbrauch: 3 Holz + 1 Stein/Erz → 1 Handelware pro Ø 7.5 Sek
- Engpass: bei Rohstoffmangel skaliert Produktion proportional
- Produktionsrate im Panel: +X.X/min (nur wenn Holz UND Stein/Erz vorhanden)
- `prodHandelswarenProMin()` und `prodSteinErzProMin()` als Hilfsfunktionen

## Tagesabschluss 2026-04-13

### Heute erledigt
- Startabbruch behoben: in `rohstoffTick()` wurde ein undefiniertes `min` verwendet
- Handwerker-Lagergrenze in `rohstoffTick()` auf explizite `50` gesetzt
- Minenproduktion auf `1.5 Stein/Erz` pro Minenarbeiter und Minute angepasst
- Panel-Label von `Rohstoffe` auf `Stein/Erz` umbenannt
- Handwerker-Freigabe auf Rohstoffsumme `Holz + Stein/Erz` umgestellt

### Offener Punkt
- Trotz ausreichender Rohstoffsumme (`Holz + Stein/Erz > 10`) entstehen im Spiel teils weiterhin keine Handwerkerjobs
- Naechster Einstieg: `wachstumMittelTick()`, `tickSchichten()` und Anzeige in `renderInfo()` zusammen pruefen

## TODO - Naechste Session

### Offene Designfragen aus dieser Session
- Schmiede-Stufen 2 und 3 sauber festlegen
  aktuell ist nur Stufe 1 inhaltlich bewusst geschärft; die höheren Stufen sind noch provisorisch
- Prüfen, ob Rohstoffkarten zusätzlich Brutto und Netto anzeigen sollen
- Optional: feste eigene Detailzeile für Schmiede-Arbeiter im Einwohnerpanel statt nur Zusammenfassungstext

### Gebaeude-Popup verbessern
- Breite auf `420px`
- Titel ueber Bild
- Effekte-Karte nach Bild
- `Ausbau`-Label ueber Optionen

### Ritter/Grosshaendler Annehmen-Button fixen
- Buttons funktionieren nicht - DOM-Event Problem
- Wenn Kapazitaet voll: Popup oeffnen, welche aktive Einheit ersetzt werden soll
- Kapazitaet: Ruf `0-50 = 1 Platz` / `51-80 = 2 Plaetze` / `81-100 = 3 Plaetze`

### Steuersatz-Bug fixen
- Niedrig und Normal haben gleiches EK - Faktoren pruefen
- Soll sein: Niedrig `50%` / Normal `100%` / Hoch `150%`
