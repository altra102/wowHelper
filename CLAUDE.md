# CLAUDE.md — wowHelper

Kleine interaktive Übungs-Seiten für World-of-Warcraft-Bossmechaniken, gebaut mit Three.js.
Jede Seite trainiert eine bestimmte Mechanik, nicht den ganzen Bosskampf.

## Struktur

- Pro Boss ein Unterordner, kleingeschrieben (z. B. `sszorak/`), darin eine `index.html`.
- Neue Bosse unten in die Tabelle und als Link in die `index.html` im Root eintragen (Startseite für GitHub Pages).

## Technik

- Jede Seite ist eine einzelne `index.html`, die per Doppelklick läuft: kein Build-Schritt, kein lokaler Server.
- Three.js wird per CDN über `<script type="importmap">` eingebunden, mit fest gepinnter Version.
- Weitere Abhängigkeiten nur, wenn es ohne sie wirklich nicht geht.

## Mechaniken

- Vor dem Bauen die Mechanik an einer Quelle prüfen (Wowhead, Icy Veins, Mythic Trap).
  Quelle und Schwierigkeitsgrad kommen als Kommentar oben in die Datei.
- Alle Werte (Timer, Radien, Geschwindigkeiten, Schaden) stehen als Konstanten oben im Skript,
  damit man sie nachjustieren kann, wenn Blizzard sie ändert.

## Bosse

| Ordner | Boss | Raid | Mechanik |
|---|---|---|---|
| `sszorak/` | Sszorak | Der Giftige Abgrund (Midnight S2) | Tempest (Tornados) |
