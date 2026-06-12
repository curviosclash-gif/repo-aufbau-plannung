# Befund-Bericht: Neuaufbau CurviosClash (Analyse, 2026-06-12)

Reine Bestandsaufnahme, keine Empfehlung. Repo-Alter: ~3,5 Monate (erster Commit 2026-02-21), 1666 Commits.
Vorversion dieses Berichts gesichert als `tmp/analyse-neuaufbau.prev.md`; deren Graph-Evidence ist hier eingearbeitet.

## 1. Warum ging der Überblick verloren?

**Code-Ebene — gesund.** Der Code funktioniert und ist strukturiert: `src/` hat 574 JS-Dateien (~98k Zeilen)
mit klarer Modulgliederung (`src/core/`, `src/modes/` mit Strategy-Pattern, `src/entities/`, `src/ui/`,
`src/shared/contracts/`, `src/mobile-classic/`). Größte Datei: `src/core/arcade/ArcadeRunRuntime.js`
(1492 Zeilen) — keine Monster-Files (God-File-Abbau V140 lief). Verifikation heute: `npm run test:contract`
lief 80+ Tests grün, **blieb dann im Graph-RAG-/Tooling-Segment hängen** (3 node-Prozesse 19 Min. idle,
Abbruch); alle produktnahen Tests (Ghost, Blueprint, Plan-Validator) bestanden. Graph-Checks aus der
Vorversion: `graph:check` PASS, kritische Flows (spawn, combat-hit, round-end, settings) grün, Coverage
adjusted 79,2 %. Featurearbeit gelang bis zuletzt (V130 Map-Pack, V131 Mobile, V132 Android). Hotspots sind
dokumentiert und begrenzt (`docs/prozess/Open_Findings.md`: P45 `UIStartSyncController.js` 772 Zeilen,
P47/P48 Boundary-/Recorder-Drift).

**Prozess-Ebene — reparabel, aber stark gedriftet.** Von 64 Dateien in `docs/plaene/aktiv/` sind laut
`docs/generated/plan-index.json` nur **14 echt offen** (`planned`); 34 sind `done`, liegen aber weiter in
`aktiv/`; 15 V-Dateien (V64–V98, V141) haben keine Master-Zeile mehr ("protected-dependency-source"). Von
den 14 offenen Blöcken sind nur 5 Produkt-/Architekturarbeit (V96, V106, V113, V118, V146) — **9 sind
Meta-Tooling** (Agent-Memory V122, Graph V124/V127, Knowledge-Core V136, Drift-Guard V142, Agent-Skills
V143, JSON-SoT V144, Autopilot V147, Test-Automation V148). Vier parallele Wahrheitsquellen
(`docs/Umsetzungsplan.md`, `plan-index.json`, `knowledge-graph.json` mit 4113 Knoten/7112 Kanten,
`docs/plaene/CHANGELOG.md`, 898 Zeilen) plus Lock-Status-JSONs; ihr Abgleich ist ein eigener wiederkehrender
Arbeitstyp ("Archivierungsabgleich", "Statusdrift synchronisiert"). Aktueller Zustand: `npm run plan:check`
FAIL am Evidence-Format in `docs/plaene/aktiv/V131.md`, `docs:check` fällt mit. Gesamt: 168 Plan-Dateien
(64 aktiv + 69 alt + 35 neu) in 3,5 Monaten.

**Governance-Ebene — verloren (selbstblockierend).** 594 von 1666 Commits (36 %) sind reine `docs:`-Commits,
plus 275 `chore:` — weniger als die Hälfte ist Produktarbeit. `package.json` führt ~150 npm-Scripts, davon
23 `check-*`-Gates; `scripts/` umfasst 103 Dateien / 36k Zeilen (gut ein Drittel des `src/`-Umfangs nur für
Tooling). Jeder Commit durchläuft `.husky/pre-commit` (guard:main, plan:check, staged architecture guard)
und den `agent:commit`-Envelope mit Pflicht-Trailern (Decision-Klasse, Evidence, Not-checked,
Residual-risk). Beleg für Selbstblockade: Die reine *Anlage* des Plans V148 (docs-only) musste 7+ Dateien
anfassen (Master, Changelog, plan-index, knowledge-graph + coverage, Overlap-Metadaten in 3 fremden Plänen)
und wurde trotzdem von den Gates gestoppt
(`docs/Fehlerberichte/2026-06-11_v148-commit-blocked-by-parallel-plan-state.md`). **5 der letzten 6
Fehlerberichte dokumentieren Governance-Blockaden, keine Spielfehler.** Die Contract-Suite testet
überwiegend die Governance selbst (agent-context, commit-envelope, diff-audit, Graph-RAG). Frühere
Reparaturversuche (V109 "Entschlackung", V116 "Kontext-Reduktion",
`docs/plaene/neu/Handlungsempfehlungen_Meta_Quote_Reduktion.md`) erzeugten ihrerseits neue Pläne und Gates.

**Fazit:** Code: gesund. Prozess: reparabel. Governance: verloren — sie verbraucht die Mehrheit der Arbeit
und blockiert sich nachweislich selbst.

## 2. Was ist erhaltenswert?

**Kronjuwelen**
- Spiellogik komplett: `src/core/main.js`, `GameBootstrap.js`, `GameLoop.js`, `PlayingStateSystem.js`,
  `src/modes/` (Classic/Hunt/Arcade-Strategien), `src/core/arcade/` (Run-Runtime, Ghost-System),
  `src/entities/`, `src/state/`, `src/shared/contracts/`, `src/network/` + `server/` (LAN-Signaling).
- Balancing/Content: `src/shared/contracts/GameplayConfigContract.js`, `FightHangarBalanceContract.js`,
  `src/entities/ai/BotTuningConfig.js`, `src/core/config/maps/presets/` (15 JS-authored Maps inkl.
  `parcours_pack_v130.js`, sechs shipping-fähige Parcours laut `docs/plaene/aktiv/V130.md`),
  `data/maps/`, `data/vehicles/`; V76-Blueprint-/Tier-Regeln (Tests grün).
- Assets: `assets/` 252 MB — u. a. 120 `.glb`, 43 `.obj`, 9 `.blend`, 9 `.fbx` (models, items, portals, trails, ui).
- Bot-Training (als Sidecar-Wissen): `python/` (PPO `train.py`/`eval.py`), `trainer/`, `dev/scripts/`,
  `src/entities/ai/training/WebSocketTrainerBridge.js`, Checkpoints in `data/training/` (~4 MB je),
  `data/bot_validation_report.json`, fachliche Doku `docs/bot-training/Bot_Trainingsplan.md`
  (PPO-Stand 2026-05-08; produktive PPO-Umschaltung bis BT95 gesperrt).
- Tests: die produktnahen der 145 Testdateien (Ghost, Blueprint, Physics, Playwright-Smokes; V112 belegt
  585 grüne Contract-Tests) — nicht die Governance-Selbsttests.
- Plattform-Schalen, sofern Targets bleiben: `electron/`, `android-classic/`, `editor/`, `android-map-tools/`.
- Blaupause: `scripts/export-game-only-repo.mjs` definiert bereits den minimalen Spielumfang
  (dist-app, electron, server, data, shared/contracts).

**Toter Ballast / Kandidaten**
- `archive/` 227 MB (1144 Dateien, Snapshot `root-2026-04-20_220003`) + `docs/archive/` 161 MB ≈ 390 MB.
- `prototypes/powerup-lab/`, `.codex_tmp/`, `.fallow/`, `.vs/`, `dist/`, `test-results/`, `logs/`,
  `videos/`, tmp-Logs im Root. Achtung: `prototypes/vehicle-lab/` ist **nicht** pauschal tot —
  `src/shared/vehicle-lab/ModularVehicleMeshBridge.js` und `tests/vehicle-lab-*.contract.test.mjs` zeigen darauf.
- Der Governance-Apparat als Ganzes: Knowledge-Graph + Coverage, plan-index, Lock-Registry (obwohl
  Single-Agent-Betrieb), Graph-RAG + Viewer, Plan-Autopilot (eingefroren), agent-commit-Envelope,
  23 check-Scripts, 26 Workflows in `.agents/workflows/`. Technisch funktionsfähig, aber Hauptlast (siehe 1).

## 3. Wo startet man am besten?

**Kleinster lauffähiger Kern** (Web-Build, ein Modus): `index.html` + `vite.config.js` →
`src/core/main.js` → `AppInitializer.js`/`GameBootstrap.js` → `GameLoop.js` +
`src/core/runtime/GameRuntimeCoordinator.js`, dazu Renderer (`src/core/renderer/`), Input
(`src/core/input/`), `src/modes/ClassicModeStrategy.js`, `src/entities/`, `src/state/` (RoundState),
`src/shared/contracts/`, minimale UI.

**Reihenfolge der Subsysteme:** 1) Core-Loop + Renderer + Input + Classic → 2) weitere Modi (Hunt,
Arcade inkl. Ghost) → 3) Settings/Profile (`src/core/settings/`, `SettingsManager.js`) → 4) HUD/Menü/
Start-Setup (`src/ui/`) → 5) Multiplayer (`server/`, `src/network/`) → 6) Plattform-Schalen (Electron,
Android-Capacitor) → 7) Editor/Map-Tools → 8) Bot-Training-Bridge.

**Wege (ohne Empfehlung):**
- **Neues Repo, Kern kopieren:** + sauberer Schnitt, Governance-Neustart, kein 390-MB-Ballast.
  − Verlust des Commit-Kontexts (Evidence verweist auf Hashes), Risiko stiller Funktionsverluste bei
  574 Dateien mit impliziten Kopplungen, Test-/Asset-Migration muss bewusst erfolgen.
- **Aufräumen im Bestand:** + Historie, grüne Tests, Tools sofort nutzbar, geringstes Funktionsrisiko.
  − Genau die Gates, die wegmüssten, blockieren das Aufräumen (Fehlerberichte belegen es); Meta-Lärm
  bleibt anfangs sichtbar; kein psychologischer Neuanfang.
- **Schrittweise Migration (neues Repo, Alt-Repo read-only als Referenz):** + ab Tag 1 lauffähig
  (`export-game-only-repo.mjs` als Startpunkt), jeder Kronjuwel-Pfad per Test belegbar, Risiko pro
  Schritt klein. − Übergangsphase mit zwei Wahrheiten, Doppelpflege, Gefahr "Migration wird nie fertig".

## 4. Was ist vor dem Aufbau zu entscheiden? (nur durch dich beantwortbar)

1. **Targets:** Welche der ~7 Auslieferungen bleiben? Desktop-Electron, Browser-Demo, Android-Classic,
   Android-Map-Tools, Editor, LAN-Multiplayer, Trainings-Stack. Jedes Target = eigene Build-/Test-Lane.
2. **Git-Historie:** behalten (1666 Commits, 36 % docs), frisch starten, oder nur Tags/Archivpfade sichern?
3. **Governance-Bilanz:** Was war Hilfe (Plan-Blöcke mit DoD, Contract-Tests, `plan:check`?), was Last
   (Evidence-Pflicht pro Checkbox, Knowledge-Graph, plan-index, Lock-Registry trotz Single-Agent,
   agent:commit-Envelope, 26 Workflows, Changelog-Pflicht)?
4. **Künftige Gedächtnisquelle:** Markdown-Master, generierter Index, Graph — oder bewusst weniger?
   Wie viel Doku pro Änderung soll Pflicht sein (ein Changelog-Satz vs. heutiges Evidence-Format)?
5. **Bot-Training:** weiterführen, einfrieren oder nur Checkpoints/Reports/Doku konservieren?
6. **Offene Sachthemen:** Electron-Major P21 (Wiedervorlage 2026-07-11, `docs/prozess/Open_Findings.md`)
   und Findings P14/P45–P48 — mitnehmen oder im Neuaufbau auflösen?
7. **Archiv-Schicksal:** `archive/` + `docs/archive/` (~390 MB) löschen, extern auslagern oder behalten?
8. **Testziel für den Neustart:** zuerst nur Contracts, oder Headless-Kernel/Browser-/Desktop-/Android-Smoke?

---
Ausgeführte Checks: `test:contract` 80+ grün, dann Hänger im Tooling-Segment (abgebrochen); Vorversion:
`graph:check` PASS, Coverage 79,2 %, `plan:check`/`docs:check` FAIL an V131-Evidence-Format.
Not checked: Build, Playwright-Suiten, Desktop-App, Android-Device, Vollsuite.
