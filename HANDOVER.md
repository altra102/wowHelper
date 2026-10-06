# Handover — wowHelper (Stand 2026-10-06)

## Kurzfassung

- **Live:** https://altra102.github.io/wowHelper/ (Startseite) · https://altra102.github.io/wowHelper/sszorak/
- **Repo:** https://github.com/altra102/wowHelper (öffentlich, Branch `main`, GitHub Pages aus Root)
- **Stand:** Die Sszorak-Übung „Tempest“ ist spielbar. Rückmeldung des Users: „bis her ist das spiel gut“.
- Alles ist committet und gepusht, es gibt keine offenen Änderungen.

## Was das Spiel kann (`sszorak/index.html`)

**Ablauf**
1. Startbildschirm mit Rollenwahl: Nahkampf oder Fernkampf. Die Leertaste startet mit der zuletzt gewählten Rolle.
2. Nach 3 s castet Sszorak **Tempest** (2 s). Es erscheinen eine Castleiste und ein Ring am Boden, der sich zum Boss zusammenzieht.
3. 10 Tornados spawnen als **gleichmäßiger Ring mit festen Winkeln**. Sie fliegen alle gerade nach außen, mit 7 Yards/s.
4. Am Arenarand prallen sie in eine **zufällige Richtung** zurück (bis ±70° um die Richtung zur Mitte). Erst danach driften sie zufällig. Sie bleiben 16 s, also bis zum übernächsten Cast, dadurch sind bis zu 20 gleichzeitig unterwegs.
5. Alle 8 s kommt ein neuer Cast.
6. 5 Herzen, jeder Treffer kostet eins, danach 1,5 s unverwundbar. Game Over zeigt die überlebte Zeit und die Anzahl der Tempests.
7. Im Start- und Game-Over-Bildschirm laufen Tornados als Hintergrund weiter.

**Steuerung (WoW-Standard, vom User ausdrücklich gewünscht)**

| Eingabe | Wirkung |
|---|---|
| W / S | vor / zurück in Blickrichtung |
| A / D | drehen; mit gedrückter rechter Maustaste seitwärts |
| Q / E | seitwärts |
| Rechte Maustaste halten | Figur dreht mit der Maus, Kamera hinter der Figur |
| Linke Maustaste halten | nur Kamera |
| Beide Maustasten | vorwärts laufen |
| Mausrad | Zoom |
| 1–4 | Fähigkeiten |

**Figuren.** Je nach Rolle gibt es eine eigene Figur, gebaut aus Grundformen (`warrior` / `mage` mit `warriorRig` / `mageRig`):
- Nahkampf: Schwertkämpfer in Plattenrüstung mit gehörntem Helm, leuchtendem Visier, Runenschwert, Rundschild und rotem Umhang.
- Fernkampf: Zauberer in violetter Kapuzenrobe mit leuchtenden Augen, Stab mit Kristall und drei schwebenden Runensteinen.
- `animatePlayer(dt)` animiert Laufen (Tempo aus der Positionsänderung), Atmen und den Umhang. Beim Schlag holt der Krieger aus, bei Klingensturm dreht er sich. Der Zauberer hebt beim Zaubern den Stab, die Leuchtkugel (`castOrb`) sitzt an der Stabspitze.
- Ein kleines Punktlicht vor der Figur sorgt dafür, dass sie in der dunklen Arena lesbar bleibt.

**Fähigkeiten.** Sie treffen nur den Boss und haben **keine Wirkung**, nur Effekte. Das ist so gewollt, sie dienen nur zum Üben.

| Taste | Nahkampf | Fernkampf |
|---|---|---|
| 1 | Schlag, sofort, 5 yd | Arkanlanze, 1,2 s Zauberzeit (Vorgabe des Users) |
| 2 | Klingensturm, sofort, 8 s Abklingzeit | Arkansplitter, 3 Geschosse, 6 s Abklingzeit |
| 3 | Ansturm, 8–25 yd, 12 s Abklingzeit | Sternsturz, 2,5 s Zauberzeit, 12 s Abklingzeit |
| 4 | Sprint, +70 % für 4 s, 20 s Abklingzeit | Blinzeln, 15 yd in Blickrichtung, 15 s Abklingzeit |

Regeln:
- 1 s globale Abklingzeit. Ausnahmen sind Sprint und Blinzeln.
- Laufen unterbricht Zauber („Unterbrochen“).
- Der Boss muss vor der Figur stehen („Ziel muss vor dir sein“). Angriffe drehen die Figur nicht automatisch, nur Ansturm tut das.
- Weitere Fehlermeldungen: „Außer Reichweite“, „Zu nah“, „Noch nicht bereit“. Die Aktionsleiste zeigt Abklingzeiten an und färbt Felder außer Reichweite rot.

## Code-Aufbau

Alles steckt in einer Datei (`sszorak/index.html`, ca. 1415 Zeilen) und ist in Abschnitte mit `// ---------- Name ----------` gegliedert:
Werte, Renderer/Szene/Nachbearbeitung, Licht, Arena, Boss, Spieler (Figuren + Animation), Tornados, Effekte, Fähigkeiten, Eingabe & Kamera, HUD, Spielzustand, Loop.

Wichtige Punkte:
- **Konstanten** stehen oben unter „Werte“: Tornado-Anzahl, Tempo, Lebensdauer, Cast-Timer, Reichweiten usw.
- **Three.js 0.170.0** wird per CDN über die Importmap geladen. Bloom läuft über `three/addons` (EffectComposer, UnrealBloomPass, OutputPass).
- **Tornado-Optik**:
  - Der Trichter ist eine LatheGeometry mit eigenem Shader. Im Shader stehen in x nur ganzzahlige Frequenzen, damit die Naht unsichtbar bleibt.
  - Trümmerteile und Sporen sind `Points`, die komplett im Vertex-Shader animiert werden.
  - Alle Shader teilen sich die Uniform `time`. Das Trichterprofil (`FUNNEL_*`) wird per Template-String in den Trümmer-Shader eingesetzt und muss zum Trichter passen.
- **Effekte**:
  - `effects` ist eine Liste von Funktionen `dt => weiterlaufen?`.
  - Hilfsfunktionen sind `addTimedFx`, `addBurst`, `swingFx`, `boltFx`, `beamFx` und `addShockwave`.
  - Effekte, die während eines Durchlaufs neu entstehen, werden angehängt und nicht verworfen. Das war früher ein Bug.
- **Fähigkeiten** sind datengetrieben in `ABILITIES.melee` und `ABILITIES.ranged`. Felder: `name, icon, cast, cd, range, minRange, offGcd, moves, use`.
- **Richtungen**:
  - vorwärts = `(sin yaw, cos yaw)`, rechts = `(-cos yaw, sin yaw)`, wobei `yaw = player.rotation.y`
  - Kamera: `camYaw = yaw + π + camOffset`
- **Spielschleife**: `update(dt)` läuft nur beim Spielen. `updateWorld(dt)` läuft immer und kümmert sich um Tornados, Effekte und Kulisse.

## Annahmen, nicht verifiziert

- Laut Quellen (Mythic Trap, Icy Veins) schießt Tempest Tornados vom Boss, macht viel Schaden plus DoT und kommt einmal pro „Apex Predator“-Sequenz.
- **Alles Weitere ist geschätzt oder vom User vorgegeben, nicht im Spiel gemessen.** Das betrifft Anzahl, Tempo, Abprallen, Driften, Arenagröße (Radius 30 yd) und Timer.
- Einen DoT gibt es nicht. Ein Treffer kostet einfach ein Herz.

## Bekannte Einschränkungen

- Kein Pointer Lock: Der Mauszeiger bleibt sichtbar und stoppt am Bildschirmrand. Mit Pointer Lock würde Chrome bei jedem Mal einen Hinweis „Esc zum Beenden“ einblenden.
- Die Maussteuerung ist nicht automatisch getestet, nur die Tastatur.
- Die Startposition `(0, 0, 12)` liegt genau auf einer Tornado-Bahn (Winkel 0). Man muss beim ersten Cast ausweichen.
- Es gibt keine automatisierten Tests, weil jede Seite eine einzelne HTML-Datei sein soll. Geprüft wird per Playwright im Browser.
- Die Performance mit Bloom und bis zu 10–20 Tornados ist auf schwachen Rechnern nicht getestet.

## Testen

- Playwright darf kein `file://` öffnen. Deshalb `cd sszorak && python3 -m http.server 8765` starten und `http://localhost:8765/index.html` aufrufen. Danach den Server mit `pkill -f "http.server 8765"` stoppen.
- Tasten simulieren: `window.dispatchEvent(new KeyboardEvent('keydown', {code: 'KeyE'}))` und dann `keyup`.
- Eine Rolle starten: `document.querySelector('button[data-role=melee]').click()`.
- Modul-Variablen sind von außen nicht erreichbar. Zum Debuggen vorübergehend `window.__dbg = () => ({ ... })` einbauen und danach **wieder entfernen**.
- Screenshots kommen in `.playwright-mcp/` (steht in `.gitignore`) und werden danach gelöscht.

## Deploy

- Wenn `main` gepusht ist, baut GitHub Pages die Seite in etwa einer Minute neu.
- Status prüfen: `gh api repos/altra102/wowHelper/pages/builds/latest -q '.status + " " + .commit'`
- Der User hat erlaubt, nach dem Testen **direkt zu pushen**.

## Lokale Einstellungen (nicht im Repo)

- `.claude/settings.local.json` sperrt die Skills `claude-md` und die großen `threejs-game-*` (game-director, aaa-graphics-builder, 3d-, image- und audio-generator, qa-release).
- In `~/.claude/skills/` sind zusätzlich installiert: `threejs-fundamentals`, `threejs-animation`, `threejs-interaction` (aus CloudAI-X/threejs-skills).

## Mögliche nächste Schritte (nicht beauftragt)

- Pointer Lock für die Maus
- Startposition zwischen zwei Tornado-Bahnen, oder den Ring bei jedem Cast anders drehen
- DoT-Debuff nach einem Treffer, wie im echten Kampf
- Rückwärtslaufen langsamer wie in WoW, oder Schwierigkeitsstufen
- Weitere Mechaniken oder Bosse: dafür einen neuen Ordner anlegen und ihn in der Root-`index.html` und der Tabelle in `CLAUDE.md` eintragen
