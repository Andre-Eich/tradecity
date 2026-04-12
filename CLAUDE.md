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
- `farmers` — Bauern (produzieren Nahrung)
- `miners` — Minenarbeiter (produzieren Rohstoffe)
- `builders` — Bauarbeiter (auf Baustellen aktiv)
- `idle` — ohne Job

Verteilung läuft über `updateWorkerDistribution()` mit sanfter Lerp-Annäherung (8%/sek).

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

### Gebäude-Stufen mit Bauaufwand und Kosten

#### 🪚 Sägewerk (neu – noch nicht implementiert)
| Stufe | Kosten | Bauaufwand | Produktion | Arbeiter |
|---|---|---|---|---|
| 1 | 50G · 20 Stein | 30 | +5 Holz/min | 5 |
| 2 | 100G · 30 Stein | 60 | +12 Holz/min | 10 |
| 3 | 200G · 50 Stein | 120 | +25 Holz/min | 20 |

#### ⛏ Mine (Umbau – derzeit nur Rohstoffe, künftig Stein & Erz)
| Stufe | Kosten | Bauaufwand | Produktion | Arbeiter |
|---|---|---|---|---|
| 1 | 100G | 40 | +5 Stein/min | 5 |
| 2 | 150G · 20 Holz | 80 | +12 Stein/min | 10 |
| 3 | 250G · 40 Holz | 160 | +25 Stein/min | 20 |

> Stufe 1 benötigt nur Gold (Early-Game ohne Holz/Stein-Voraussetzung)

#### 🔨 Schmiede (neu – noch nicht implementiert)
| Stufe | Kosten | Bauaufwand | Produktion | Arbeiter |
|---|---|---|---|---|
| 1 | 100G · 20 Holz · 30 Stein | 50 | +3 Werkzeuge/min | 3 |
| 2 | 200G · 40 Holz · 60 Stein | 100 | +8 Werkzeuge/min | 6 |
| 3 | 350G · 80 Holz · 100 Stein | 200 | +15 Werkzeuge/min | 10 |

#### Bestehende Gebäude (Kosten-Referenz für neue Ressourcen-Anforderungen)
| Gebäude | Kosten | Bauaufwand |
|---|---|---|
| ⛪ Kirche | 150G · 30 Holz · 50 Stein | 60 |
| ⚔ Kaserne | 200G · 20 Holz · 80 Stein | 80 |
| 🌾 Lagerausbau | 200G · 40 Holz · 20 Stein | 70 |
| 🏪 Marktplatz | 100G · 30 Holz · 30 Stein | 50 |

### Einfache Leute – Arbeitszuweisung
- Neue Bürger starten als Bauern mit 20% Produktivität und werden als "jobsuchend" markiert
- Bauarbeiter werden zuerst abgezogen (fest bis Bau fertig, dann zurück zu jobsuchend)
- Rest wird fließend (1/sek) nach Platz-Verhältnis verteilt:
  - Bauern = Verfügbare × (Ackerplätze / Gesamtplätze)
  - Miner = Verfügbare × (Mineplätze / Gesamtplätze)
  - Säger = Verfügbare × (Sägewerkplätze / Gesamtplätze)
- Überschuss (mehr Leute als Plätze) → jobsuchend, 20% Produktivität
- Priorität: Spieler kann pro Gebäude +30% Arbeiteranteil aktivieren (im Gebäude-Popup)

### Ackerfelder
- 3 Felder, je 3 Stufen ausbaubar
- Stufe 0: 20 Plätze · Stufe 1: 50 · Stufe 2: 100 · Stufe 3: 150
- Gesamt max: 3 × 150 = 450 Bauern-Plätze
- Baukosten je Stufe: 30G · 10s Bauzeit

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
1. `state.wood` und `state.tools` in game.js einführen (Lager, Produktion, Verbrauch)
2. Sägewerk-Gebäude in `queueBuild`, `completeBuild` und Render-Logik einbauen
3. Schmiede-Gebäude einbauen (Input: Stein+Erz → Output: Werkzeuge)
4. Mine auf Stein+Erz-Output umstellen
5. Handwerker-Produktion an Holz+Werkzeuge-Verfügbarkeit koppeln
6. UI-Ressourcenkarten für Holz und Werkzeuge aktivieren (Platzhalter bereits in index.html)

## Aktuelle Version
- **Status:** Entwicklungsversion / kein formaler Release-Tag
- **Version:** `0.1-dev`
- **Bemerkung:** Das Projekt ist aktiv in Arbeit und wird momentan über `index.html`, `game.js` und `style.css` gepflegt.
