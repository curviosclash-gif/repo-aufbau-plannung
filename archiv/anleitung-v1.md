# Schritt-für-Schritt-Anleitung: Neuaufbau CurviosClash

Stand: 2026-06-12. Basis: Befund in `tmp/analyse-neuaufbau.md`.
Ansatz: **neues Repo, Code modulweise migrieren statt neu schreiben**, Gedächtnis wächst pro Migrationsschritt mit.
Entscheidungen sind als **E1–E10** markiert; wo ich eine begründete Empfehlung habe, steht sie dabei — entschieden wird von dir.

---

## Phase 0: Grundsatzentscheidungen (vor dem ersten Commit)

**E1 — Targets:** Welche Auslieferungen soll das neue Repo am Ende haben?
Altbestand: Desktop-Electron, Browser, Android-Classic, Android-Map-Tools, Editor, LAN-Multiplayer, Trainings-Stack.
Jedes Target kostet eine eigene Build-/Test-Lane. *Empfehlung: mit genau einem Start-Target beginnen (Browser ODER Desktop), Rest erst nach lauffähigem Kern entscheiden.*

**E2 — Migrieren vs. Neuschreiben pro Modul:** Default laut Befund: migrieren (Code ist gesund).
Neuschreiben nur gezielt für die dokumentierten Hotspots (`UIStartSyncController.js`, Recorder-Pfad P48).
*Hier entscheiden: Gilt "migrieren als Default, Neuschreiben nur mit Begründung"? Ja/Nein.*

**E3 — Sprache/Stack:** Plain JS beibehalten (Altbestand: 574 JS-Dateien) oder Umstieg auf TypeScript?
TS bringt Struktur-Sicherheit, macht aber aus jedem Migrationsschritt eine Teil-Neuschreibung.
*Empfehlung: JS beibehalten, ggf. JSDoc-Typen + `checkJs` für neue Dateien; TS-Frage nach Phase 2 neu bewerten.*
Dazu: three.js-Version — alt ist `^0.160.0` (Stand ~Ende 2023). Beim Neuaufbau direkt auf aktuelle Version heben (Breaking Changes einmal bezahlen) oder pinnen? *Empfehlung: einmalig heben, in Phase 1, solange noch wenig Code da ist.*

**E4 — Gedächtnis-Format (das Kernziel):** Was ist das Pflicht-Gedächtnis pro Änderung?
Vorschlag (bewusst minimal, Lehre aus dem Altrepo):
- `CLAUDE.md` — Arbeitsregeln, max. 1 Seite.
- `docs/landkarte.md` — eine Zeile pro Modul: Pfad, Zweck, Abhängigkeiten, Test. Pflicht bei jedem neuen Modul.
- `docs/adr/NNN-titel.md` — kurze Entscheidungsnotiz (5–10 Zeilen) nur bei echten Architekturentscheidungen.
- `docs/changelog.md` — ein Satz pro abgeschlossenem Arbeitspaket.
Harte Obergrenze: Gedächtnispflege ≤ 2 Minuten pro Commit, sonst ist das Format falsch.
*Hier entscheiden: reicht dieses Set, oder willst du zusätzlich etwas (z. B. Skizzen, Balancing-Journal)?*

**E5 — RAG:** Jetzt nicht bauen. Trigger-Kriterium festlegen, ab wann neu bewertet wird
(Vorschlag: erst wenn Repo > ~100k Zeilen oder direkte Suche nachweislich nicht mehr reicht).
*Hier entscheiden: Trigger akzeptiert oder anderes Kriterium?*

**E6 — Altrepo-Schicksal:** bleibt read-only als Referenz liegen (Empfehlung), wird archiviert oder gelöscht?
Was passiert mit `archive/` + `docs/archive/` (~390 MB) und den 168 Plan-Dateien? Git-Historie wird nicht migriert
(Neues Repo startet bei Commit 1; Altrepo bleibt als Nachschlagewerk inkl. Historie).

**E7 — Bot-Training:** mitnehmen (eigene Phase 7), einfrieren (nur `python/`, Checkpoints und
`docs/bot-training/Bot_Trainingsplan.md` als Kopie sichern) oder aufgeben?
*Empfehlung: einfrieren + sichern; Reaktivierung erst nach Phase 3, weil Training deterministischen Spielkern braucht.*

**Done-Kriterium Phase 0:** E1–E7 sind beantwortet und als erste ADRs (`docs/adr/001…007`) im neuen Repo notiert.

---

## Phase 1: Neues Repo aufsetzen (1 Sitzung)

1. Neues Repo anlegen (Name entscheiden — **E8**), Node-LTS via `.nvmrc`, `npm init`, Vite, three.js (Version laut E3).
2. Tooling bewusst klein: ESLint (Standard-Config), `node --test` für Unit-/Contract-Tests, Playwright erst ab Phase 2.
   **Keine** Pre-Commit-Gates außer Lint. Keine Generated-Artefakte (kein plan-index, kein Graph).
3. Gedächtnis-Grundgerüst aus E4 anlegen: `CLAUDE.md`, `docs/landkarte.md` (leer mit Kopfzeile), `docs/adr/`, `docs/changelog.md`.
4. Zielstruktur als leere Ordner + eine Zeile je Ordner in der Landkarte, angelehnt an die gesunde Alt-Struktur:
   `src/core/` (Loop, Runtime), `src/render/`, `src/input/`, `src/modes/`, `src/entities/`, `src/state/`,
   `src/contracts/`, `src/ui/`, `data/`, `assets/`.
5. "Stop-Regeln" in `CLAUDE.md` festschreiben (Lehren aus dem Altrepo):
   - Kein Meta-Tooling (Graph, Index, Autopilot, Lock-Registry) ohne expliziten User-Auftrag.
   - Doku-Pflicht pro Änderung: genau Landkarten-Zeile + Changelog-Satz, nicht mehr.
   - Eine Wahrheitsquelle pro Thema; keine generierten Zweitkopien.
   - Plan-Dateien: max. eine aktive Plan-Datei gleichzeitig.

**Done-Kriterium:** `npm run dev` zeigt eine leere three.js-Szene im Browser; Landkarte + ADRs 001–007 committed.

---

## Phase 2: Spielkern migrieren (das Herzstück)

Reihenfolge entlang des kleinsten lauffähigen Kerns (Quellpfade = Altrepo):

1. **Contracts zuerst:** `src/shared/contracts/` sichten; nur die übernehmen, die der Kern braucht
   (GameState, MatchLifecycle, GameplayConfig). **E9:** Contracts 1:1 übernehmen oder beim Umzug
   vereinfachen? *Empfehlung: 1:1 übernehmen, vereinfachen erst wenn der Konsument migriert ist.*
2. **Loop + Bootstrap:** `src/core/main.js`, `GameBootstrap.js`, `GameLoop.js`, `AppInitializer.js` —
   beim Umzug entkoppeln von Settings/UI (nur Stubs).
3. **Renderer:** `src/core/renderer/` (ohne Recording-Pfad! `MediaRecorderSystem`, `RecordingCapturePipeline`
   bleiben vorerst zurück — P48-Hotspot).
4. **Input:** `src/core/input/` (Desktop-Pfad; Touch erst mit Mobile-Target).
5. **Ein Modus:** `src/modes/GameModeContract.js`, `GameModeRegistry.js`, `ClassicModeStrategy.js`.
6. **Entities + State:** `src/entities/` (Player, Projectile, Collision, MapSchema), `src/state/`
   (RoundState, RoundStateController, RoundStateTickSystem).
7. **Pro migriertem Modul, immer gleicher Takt:** Datei(en) kopieren → Imports auf neue Struktur anpassen →
   zugehörige Tests aus `tests/` mitnehmen → Landkarten-Zeile schreiben → Commit.
   Erkenntnisse über Altlasten ("Modul X hing heimlich an Y") als ADR, wenn entscheidungsrelevant.

**Done-Kriterium:** Eine Classic-Runde ist im Browser spielbar (Start → Runde → Ende), migrierte
Contract-Tests grün, jede `src/`-Datei hat eine Landkarten-Zeile.
**Checkpoint mit dir:** kurzes Playtest-Feedback, bevor Phase 3 startet.

---

## Phase 3: Modi und Content

1. Hunt-Modus: `HuntModeStrategy.js`, `src/hunt/`.
2. Arcade-Modus inkl. Ghost: `src/core/arcade/` (`ArcadeRunRuntime.js` beim Umzug in 2–3 Module schneiden —
   einziger geplanter Zuschnitt, 1492 Zeilen), Ghost-Recorder/-Library samt der grünen Edge-Case-Tests.
3. Content 1:1: `src/core/config/maps/presets/` (15 Maps), `data/maps/`, `data/vehicles/`,
   Balancing-Contracts (`GameplayConfigContract`, `FightHangarBalanceContract`, `BotTuningConfig`).
   **E10:** Alle 6 Parcours-Maps und alle 3 Modi behalten, oder beim Umzug aussortieren?
4. Assets selektiv: nur referenzierte Dateien aus `assets/` (252 MB) kopieren; Skript dafür schreiben statt
   blind alles zu übernehmen.

**Done-Kriterium:** Alle gewählten Modi spielbar, Maps laden, zugehörige Tests grün.

---

## Phase 4: Settings, Profile, UI

1. `src/core/settings/` + `SettingsManager.js` (der Mutationsvertrag aus V103 ist erhaltenswert).
2. UI/HUD/Menü aus `src/ui/` — hier den P45-Hotspot **neu schreiben statt migrieren**
   (`UIStartSyncController.js`, Event-Listener-Duplikation P14 ist dokumentiert).

**Done-Kriterium:** Spiel per Menü konfigurier- und startbar; Settings überleben Reload.

## Phase 5: Plattform-Schalen (Umfang laut E1)

- Desktop: `electron/` neu aufsetzen — direkt mit aktuellem Electron-Major (löst Alt-Finding P21,
  Wiedervorlage 2026-07-11, nebenbei).
- Browser-Build, Android (`capacitor`), Editor/Map-Tools: je nach E1, jeweils eigene Phase.

## Phase 6: Multiplayer (falls Target)

- `server/lan-signaling.js`, `signaling-server.js`, `src/network/` migrieren; Playwright-Smoke dazu.

## Phase 7: Bot-Training (nur falls E7 = mitnehmen)

- `python/`, `trainer/`, `WebSocketTrainerBridge.js`, Checkpoints aus `data/training/`.
- Voraussetzung: deterministischer Kern aus Phase 2/3 steht.

## Phase 8: Altrepo stilllegen (laut E6)

1. Letzten Stand taggen, Repo read-only schalten/archivieren.
2. Kurze Abschlussnotiz im Altrepo: "Nachfolger ist <neues Repo>, Stand <Datum>".
3. Lokale Ballast-Ordner (`archive/`, `node_modules`, `dist*`, Logs) nach Bestätigung aufräumen.

---

## Übersicht der Entscheidungen

| Nr. | Entscheidung | Wann | Empfehlung vorhanden? |
| --- | --- | --- | --- |
| E1 | Welche Targets, welches Start-Target | Phase 0 | ja: eines zuerst |
| E2 | Migrieren als Default bestätigen | Phase 0 | ja: migrieren |
| E3 | JS vs. TS; three.js-Version | Phase 0 | ja: JS + Version heben |
| E4 | Gedächtnis-Format/-Umfang | Phase 0 | ja: Minimal-Set |
| E5 | RAG-Trigger-Kriterium | Phase 0 | ja: erst ab Schmerzgrenze |
| E6 | Altrepo + 390-MB-Archiv-Schicksal | Phase 0/8 | ja: read-only behalten |
| E7 | Bot-Training mitnehmen/einfrieren | Phase 0 | ja: einfrieren + sichern |
| E8 | Repo-Name | Phase 1 | nein |
| E9 | Contracts 1:1 oder vereinfacht | Phase 2 | ja: 1:1 |
| E10 | Welche Modi/Maps bleiben | Phase 3 | nein |
