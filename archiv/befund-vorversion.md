# Analyse Neuaufbau CurviosClash

Stand: 2026-06-12. Scope: read-only Analyse plus dieses D1-Report-Artefakt. Keine Code-, Plan- oder Governance-Aenderungen.

## 1. Warum ging der Ueberblick verloren?

**Code-Ebene:** Der Code wirkt gross, aber nicht verloren: `src/` hat 574 Dateien, groesste Hotspots sind u. a. `src/core/arcade/ArcadeRunRuntime.js` (1492 Zeilen), `src/core/MediaRecorderSystem.js` (1320), `src/ui/UIStartSyncController.js` (772); gleichzeitig sind die Architekturgrenzen dokumentiert (`docs/referenz/ai_architecture_context.md`, `docs/referenz/architektur_ausfuehrlich.md`) und kritische Graph-Flows sind gruen (`spawn`, `combat-hit`, `round-end`, `settings`). `npm run graph:check` PASS, `coverage-report` PASS mit adjusted 79.2%, aber kein frischer Build-/Playtestlauf in dieser Analyse.

**Prozess-Ebene:** `docs/plaene/aktiv/` enthaelt 64 Markdown-Dateien (63 V-Dateien plus `README.md`), aber Master/Planindex kennen nur 48 kanonische Block-IDs: 34 `done`, 14 `planned` (`docs/Umsetzungsplan.md`, `docs/generated/plan-index.json`). Echt offen sind damit 14 Master-Bloecke; 14 V-Dateien liegen im Aktivordner ohne Master-Zeile (`V64`, `V71`, `V72`, `V76`, `V77`, `V82`, `V87`, `V88`, `V93`, `V94`, `V95`, `V97`, `V98`, `V141`). `npm run plan:check` scheitert aktuell an Evidence-Format in `docs/plaene/aktiv/V131.md`, wodurch `npm run docs:check` mitfaellt; `docs/plaene/CHANGELOG.md` dokumentiert denselben Blocker beim V148-Intake.

**Governance-Ebene:** Die Pflegekosten pro Aenderung sind der eigentliche Ueberblicksverlust: `docs:check` verkettet Freshness, `plan:check`, Agent-/Gemini-/Drift-Checks (`package.json`), aktive Plaene verlangen DoD/Evidence/Risiko/Phasen (`.agents/rules/planning_and_governance.md`), und 10 der 14 offenen Master-Bloecke liegen in AI/Graph/Repo-Governance statt direkt im Spiel (`docs/generated/plan-index.json`). `scope-collisions` meldet Ueberlappungen gerade zwischen Governance-/Graph-Bloecken (`V142`/`V144`, `V143`/`V144`/`V147`, `V148`/`V96`).

Fazit Code: **gesund** - gross und mit Legacy-Surfaces, aber lauffaehige Kernpfade und Tests/Graph-Evidence sind vorhanden.  
Fazit Prozess: **reparabel** - Master/Planindex liefern eine Wahrheit, aber `aktiv/` und Evidence-Format driften.  
Fazit Governance: **verloren im aktuellen Modus** - Schutzmechanismen dominieren die Arbeit und erzeugen selbst neue Arbeit.

## 2. Was ist erhaltenswert?

**Kronjuwelen:** Der Spielkern liegt in `src/core/main.js`, `src/core/GameBootstrap.js`, `src/core/GameLoop.js`, `src/core/PlayingStateSystem.js`, `src/entities/**`, `src/state/RoundState*.js` und `src/shared/contracts/**`; Graph-Queries belegen validierte Flows fuer Spawn, Combat-Hit, Round-End und Settings. Balancing-/Produktdaten sind wertvoll in `src/shared/contracts/GameplayConfigContract.js`, `src/shared/contracts/FightHangarBalanceContract.js`, `src/entities/ai/BotTuningConfig.js`, `src/entities/GeneratedVehicleConfigs.js` und den Map-Presets unter `src/core/config/maps/**`, besonders `src/core/config/maps/presets/parcours_pack_v130.js`. Assets sind substanziell: `assets/` enthaelt u. a. 120 `.glb`, 43 `.obj`, 9 `.blend`, 9 `.fbx`; V130 dokumentiert sechs shipping-faehige Parcours-Maps (`docs/plaene/aktiv/V130.md`).

**Bot-Training:** Erhaltenswert als Sidecar-Wissen, nicht als fertiger Runtime-Pfad: `docs/bot-training/Bot_Trainingsplan.md` ist laut eigener Regel alleinige aktive Quelle, nennt Survival-First, PPO-Validate-Luecken und verbietet bis BT95 produktive PPO-Umschaltung; dazu gehoeren `src/entities/ai/**`, `src/state/training/**`, `src/shared/contracts/TrainingRuntimeContract.js`, `python/**`, `data/**` und Trainings-/Bridge-Tests.

**Tests:** Behalten: 145 Testdateien, `package.json` mit `test:contract`, Desktop-/Browser-/Android-/Smoke-Skripten, V112-Evidence fuer 585 Contract-Tests plus Desktop-Smoke (`docs/plaene/aktiv/V112.md`). `untested-systems` listet vor allem Tool-/Viewer-Flaechen (`tools/agent-map/viewer.js`, `tools/plan-map/viewer.js`, `electron/map-tools/**`), nicht den Match-Kern.

**Ballast/Kandidaten:** `archive/` (1144 Dateien), `docs/archive/**`, `docs/plaene/alt/**` (87 md) und `docs/plaene/neu/**` (35 md) sind Historie/Intake, nicht taeglicher Kontext. `prototypes/powerup-lab/**` wirkt als Prototype-Altpfad; `prototypes/vehicle-lab/**` ist dagegen nicht pauschal tot, weil `src/shared/vehicle-lab/ModularVehicleMeshBridge.js` und `tests/vehicle-lab-*.contract.test.mjs` darauf zeigen.

## 3. Wo startet man am besten?

Befund zum kleinsten lauffaehigen Kern: Entry und Loop laufen ueber `index.html`, `vite.config.js`, `src/core/main.js`, `src/core/GameBootstrap.js`, `src/core/GameLoop.js`, Renderer/Input, `PlayingStateSystem`, Entity-/Player-/Projectile-/Collision-Systeme, RoundState und Shared Contracts. Danach folgen beobachtbar: Settings/RuntimeConfig (`src/core/SettingsManager.js`, `src/core/RuntimeConfig.js`), HUD/Menu/Start-Setup (`src/ui/**`), Arcade/Maps/Hangar (`src/core/arcade/**`, `src/core/config/maps/**`, `src/ui/arcade/**`, `src/ui/hangar/**`), dann Desktop-Shell/Multiplayer/Recording/Mobile (`electron/**`, `src/network/**`, `src/platform/**`, `android-classic/**`), zuletzt Bot-Training/AI-Rollout.

Moegliche Wege ohne Empfehlung:
- Neues Repo: plus sauberer Kontext und kleiner Start; minus Git-Archaeologie, Assets, Graph-/Testwissen und Plan-Evidence muessen bewusst migriert werden.
- Aufraeumen im Bestand: plus Tests, Assets, History und Tools bleiben sofort nutzbar; minus `aktiv/`-/Governance-Laerm bleibt anfangs sichtbar.
- Schrittweise migrieren: plus jeder Kronjuwel-Pfad kann mit Tests belegt werden; minus laengere Phase mit zwei Wahrheiten und hoeherer Disziplinlast.

## 4. Vor dem Aufbau zu entscheiden

- Welche Targets bleiben wirklich: Desktop-App (`electron/**`), Browser-Demo, Android Classic/Arcade (`android-classic/**`, `package.json`), Map-Tools (`electron/map-tools/**`)?
- Git-Historie behalten, neu starten oder nur relevante History ueber Tags/Archivpfade sichern?
- Welche Gedaechtnisquelle ist kuenftig kanonisch: Markdown-Master (`docs/Umsetzungsplan.md`), generierter Planindex (`docs/generated/plan-index.json`), Graph (`docs/generated/knowledge-graph.json`) oder bewusst weniger?
- Welche Governance war Hilfe: Graph-Queries, `plan:check`, Contract-Tests, DoD-Notizen; welche war Last: Evidence pro Checkbox, breite Changelog-Pflicht, viele parallele Meta-Bloecke?
- Bot-Training: Sidecar-Wissen behalten, einfrieren oder als Produktziel wieder priorisieren?
- Archive/Prototypes: nur als Historie behalten, aktiv migrieren oder bewusst aus Standardkontext entfernen?
- Testziel fuer den Neustart: Headless-Kernel, Browser-Smoke, Desktop-Smoke, Android-Smoke oder zuerst nur Contracts?

Ausgefuehrte Checks: `npm run graph:check` PASS; `critical-path-health` OK; `coverage-report` PASS; `scope-collisions` mit 10 Konflikten; `npm run plan:check` FAIL wegen `docs/plaene/aktiv/V131.md`; `npm run docs:check` FAIL via `plan:check`. Not checked: Build, Playwright, Desktop-App, Android-Device, Vollsuite.
