# Aufbau-Plan: Neues CurviosClash-Repo

Stand: 2026-06-12. **Kanonischer Plan.** Konsolidiert aus drei Vorlagen: `claude-plan.md` und `codex-plan.md` (beide ersetzt, in der Git-Historie erhalten) sowie dem Playground-Entwurf (`Documents/Playground/repo-aufbau-plannung.md`).

Zweck dieses Repos: den Aufbau des neuen Spiel-Repos Schritt für Schritt zu planen. Dieser Plan ist die Grundlage für **alle** Entscheidungen — von der ersten Grundsatzfrage bis zum Release.

**Aufbau dieses Repos (Entscheidungshilfe):**
- [`plan.md`](plan.md) — dieser Plan: Index, Phasen, Status. Eine Wahrheitsquelle.
- [`befund.md`](befund.md) — read-only Faktenbasis (Altrepo-Analyse), auf die sich alle Empfehlungen berufen.
- [`gates/`](gates/) — ein Entscheidungsblatt pro Gate (A–F) mit Fakten, Optionen, Empfehlung und Entscheidungszeile. Hier triffst du die Entscheidungen.

Die Grundlage des Plans ist zweiteilig:

1. **Die Spielidee** wird aus dem Altrepo übernommen (Teil 1). Sie steht nicht zur Debatte.
2. **Die Learnings aus dem Altrepo** (Teil 2) entscheiden über alles Weitere. Stack, Struktur, Tooling, Gedächtnis und Governance werden neu entschieden (Teil 4) — nichts davon wird unreflektiert aus dem Altrepo kopiert.

---

## Teil 1 — Die Spielidee (gesetzt)

CurviosClash ist ein 3D-Fahrzeug-Arena-Spiel (three.js) mit drei Modi:

- **Classic:** rundenbasierter Arena-Kampf (Start → Runde → Ende).
- **Hunt:** Jagd-Modus.
- **Arcade:** Parcours-Läufe mit Checkpoints, Splits, Minimap, Penalty, XP, Leaderboard und Ghost-Selbstduell gegen die eigene beste Spur pro Route.

Dazu gehören: Fahrzeuge mit Hangar-/Blueprint-/Tier-System und Balancing-Contracts, Items/Powerups, kuratierte Maps (15 JS-authored Presets, davon 6 shipping-fähige Parcours), LAN-Multiplayer, Bot-Gegner (PPO-trainierbar) und Touch-/Tilt-Steuerung für Android.

**Prinzip (übernommen als „Punkt 1: Das Spiel"):** Das Spiel ist deterministisch, testbar und sauber getrennt in Simulation, Rendering, UI, Assets und Content. **Die Simulation darf nie vom Renderer abhängen** — der Renderer zeigt nur an, was die Simulation vorgibt.

Das Altrepo (`Desktop/CurviosCLash`) bleibt die Verhaltensreferenz: Was dort spielbar und richtig ist, ist die Spezifikation. Welche Teile im Detail übernommen werden, entscheidet E8.

---

## Teil 2 — Learnings aus dem Altrepo (Verfassung)

Befundlage (Analyse 2026-06-12): 1666 Commits in 3,5 Monaten. Code gesund — 574 Dateien, klare Modulstruktur, produktnahe Tests grün. Governance selbstzerstörerisch. Daraus acht bindende Lehren:

- **L1 — Regel folgt Schmerz, nie umgekehrt.** 36 % aller Commits waren reine Doku, dazu ~150 npm-Scripts und 23 Check-Gates; am Ende blockierten die Gates sogar docs-only-Commits. Neue Pflichtregeln entstehen nur noch aus wiederholten, echten Problemen (Teil 6).
- **L2 — Eine Wahrheitsquelle pro Thema.** Vier parallele Planquellen (Master, generierter Index, Wissensgraph, Changelog) machten den Abgleich zum eigenen Arbeitstyp. Keine generierten Zweitkopien, keine Schatten-Wahrheiten.
- **L3 — Kein Meta-Tooling ohne expliziten Auftrag.** Wissensgraph (4113 Knoten), Graph-RAG, Plan-Autopilot, Lock-Registry im Single-Agent-Betrieb: alles technisch funktionsfähig — und zusammen die Hauptlast des Projekts. Im Neuaufbau sind Graph und RAG **Entscheidungen mit Default „nein"** (E11/E12), keine geplanten Phasen.
- **L4 — Alt-Tests sind bezahltes Wissen — aber nur geprüft.** Die produktnahen Tests (Ghost, Blueprint, Physics, Smokes) werden migriert, nicht neu generiert; AI-generierte Tests sind messbar schwächer als kampferprobte. Ein migrierter Test gilt jedoch erst als Orakel, wenn er einen injizierten Bug fängt (Orakel-Check, Phase 2); sonst tritt der Golden-Master aus dem laufenden Altverhalten an seine Stelle.
- **L5 — Migrieren als Default.** Die Edge-Cases im Altcode (Ghost-Persistenz, Tilt-Kalibrierung, Orientation-Handling) sind teuer bezahlte Bugs. Neuschreiben nur mit Begründung; dokumentierte Ausnahmen: P45 (`UIStartSyncController.js`, Listener-Duplikation) und P48 (Recording-Pfad).
- **L6 — AI baut ohne Grenzen Apparat.** Auch im Altrepo schrieb die AI den Code — und errichtete nebenbei den Meta-Apparat. Die Stop-Regeln (Teil 3) richten sich deshalb an die AI und stehen in der `CLAUDE.md` des neuen Repos.
- **L7 — Eine aktive Plan-Datei.** 168 Plan-Dateien in 3,5 Monaten; 34 erledigte Pläne lagen weiter im „aktiv"-Ordner. Dieser Plan hier ist der einzige Plan. Er wird fortgeschrieben, nicht vervielfacht.
- **L8 — Doku ist gedeckelt.** Gedächtnispflege ≤ 2 Minuten pro Commit: eine Landkarten-Zeile + ein Changelog-Satz. Wenn es länger dauert, ist das Format falsch — nicht die Disziplin.

---

## Teil 3 — Arbeitsmodell: AI baut, du steuerst

**Deine Aufgaben** (Review-Einheit ist Verhalten, nicht Codezeilen):

- Auftrag geben: ein Modul/Slice pro Session.
- Die E-Entscheidungen (Teil 4) beantworten, wenn sie fällig werden.
- Am Slice-Ende Changelog-Satz + Landkarten-Diff lesen (≤ 2 Minuten).
- An den Phasen-Checkpoints spielen (Playtest) — dein Urteil entscheidet, ob die Phase zu ist.
- Bei dir bleiben: Löschen, Plattform-/Stack-Wechsel, neue Pflicht-Mechanismen, Release.

**Pflichten der AI** (stehen wortgleich in der `CLAUDE.md` des neuen Repos):

- Jeder Slice endet mit selbst ausgeführter Verifikation (Lint + Typecheck + Tests + Build grün), **bevor** „fertig" gemeldet wird.
- Abschlussmeldung je Slice in fünf Punkten: Was wurde gebaut? Welche Dateien sind dauerhaft wichtig? Welche Checks liefen? Was wurde bewusst nicht geprüft? Welche Entscheidung liegt beim User?
- Kein neues Modul ohne Landkarten-Zeile (inkl. Herkunft: `alt-migriert` oder `neu`). Keine Architekturänderung ohne ADR-Vorschlag.
- Verhaltens-Tests aus dem Altrepo migrieren statt neu generieren; neue Tests prüfen Verhalten, nicht Implementierung.
- Session-Scope einhalten: kein Beifang-Refactoring.

**Stop-Regeln (Verfassung, nicht verhandelbar durch die AI):**

1. Kein Meta-Tooling (Graph, Index, RAG, Autopilot, Lock-Registry, Generated-Artefakte) ohne expliziten User-Auftrag.
2. Eine Wahrheitsquelle pro Thema; keine generierten Zweitkopien.
3. Doku pro Änderung: Landkarten-Zeile + Changelog-Satz, nicht mehr.
4. Max. eine aktive Plan-Datei (diese).
5. Keine neue Pflichtregel ohne dokumentiert wiederholtes Problem (Teil 6).

**Optional ab Phase 3:** Zweit-Review durch eine separate AI-Session am Phasenende — frischer Kontext findet, was die Schreib-Session übersieht.

---

## Teil 4 — Alle Entscheidungen (E1–E15)

Jede Entscheidung ist User-owned. Die AI bereitet Optionen vor, entscheidet aber nie stillschweigend. Die **ausgearbeiteten Entscheidungsblätter** liegen je Gate in `gates/` (Frage, Fakten aus [`befund.md`](befund.md), Optionen mit Konsequenzen, Empfehlung, Entscheidungszeile). Dieser Abschnitt ist nur der Index. Sobald eine Entscheidung gefallen ist, wird sie im neuen Repo als ADR abgelegt und hier in der Status-Spalte mit ADR-Nummer markiert.

**Entscheidungsprozess für größere Fragen:** Optionen aufschreiben → Kriterien benennen (Gameplay-Fit, Komplexität, Wartbarkeit, Testbarkeit) → bei Unsicherheit Mini-Prototyp statt Diskussion → Ergebnis als ADR → nach 2–4 Wochen prüfen, ob die Entscheidung noch passt (supersedieren statt editieren).

| Gate | Blatt | Fällig | Entscheidungen | Status |
| --- | --- | --- | --- | --- |
| A | [`gates/gate-a.md`](gates/gate-a.md) | vor dem ersten Commit (Phase 0) | E1 Targets/Start-Target · E2 Stack & three.js · E3 Migrationsregel · E4 Repo-Struktur · E5 Gedächtnis-Format · E6 Name/Hosting | offen |
| B | [`gates/gate-b.md`](gates/gate-b.md) | vor der Kern-Migration (Phase 2) | E7 Contracts 1:1 oder vereinfacht | offen |
| C | [`gates/gate-c.md`](gates/gate-c.md) | vor Modi & Content (Phase 3) | E8 Übernahme-Umfang (Modi/Maps/Assets) | offen |
| D | [`gates/gate-d.md`](gates/gate-d.md) | vor Schalen & Multiplayer (Phase 6/7) | E9 Multiplayer im Scope · E10 Reihenfolge der Schalen | offen |
| E | [`gates/gate-e.md`](gates/gate-e.md) | nur bei Schmerz-Trigger (L3) | E11 Wissensgraph · E12 RAG — **Default beider: nein** | offen |
| F | [`gates/gate-f.md`](gates/gate-f.md) | Erbe ab Migrationsstart, Release vor Phase 9 | E13 Bot-Training · E14 Altrepo/Erbe/P21 · E15 Release-Definition 1.0 | offen |

**ADR-Format** (im neuen Repo, `docs/adr/NNN-titel.md`, 5–15 Zeilen): Status / Kontext / **Optionen** / Entscheidung / Konsequenzen. Nicht editieren — supersedieren. Nur architektur- und produktrelevante Entscheidungen.

---

## Teil 5 — Bauphasen bis zum Release

Jede Phase endet mit einem Done-Kriterium und — wo markiert — einem Playtest-Checkpoint mit dir. **Kadenz-Regel:** Jede Arbeitssitzung endet mit einem spielbaren Build; nie zwei Sessions hintereinander „rot".

### Phase 0 — Grundsatzentscheidungen

Gate A beantworten (E1–E6).
**Done:** Sechs ADRs (001–006) liegen im neuen Repo.

### Phase 1 — Walking Skeleton (1–2 Sitzungen)

Die dünnste durchgehende, produktionsechte spielbare Scheibe — kein Prototyp, jede Zeile bleibt.

1. Repo anlegen (laut E6), `.gitignore`, `.nvmrc`, Vite, three.js (Version laut E2), ESLint Standard, `node --test`. LICENSE + Dependency-Policy spätestens vor dem ersten Deploy (Phase 9).
2. CI ab Tag 1, bewusst klein: ein Workflow mit Lint + Tests + Build. Keine weiteren Gates, keine Pre-Commit-Hooks außer Lint.
3. Gedächtnis-Grundgerüst laut E5 anlegen; Stop-Regeln (Teil 3) in die `CLAUDE.md`.
4. Skeleton-Slice: leere Szene → ein steuerbares Objekt → Game-Loop-Tick → ein Rundenende-Event → ein Test darauf → CI grün. Dafür die ersten Alt-Module dünn anziehen (`GameLoop.js`, Teile von Renderer und Input). Das Skeleton bleibt **contract-frei**: `GameLoop.js` und `src/core/input/` importieren keine `shared/contracts` (verifiziert), und der minimale Renderer-Schnitt braucht die contract-ziehenden Render-Dateien (CameraRig/Shadow/Recording) nicht. So wird E7/Gate B nicht schon in Phase 1 vorentschieden — die Gameplay-Contracts kommen erst in Phase 2.

**Done:** `npm run dev` zeigt ein steuerbares Objekt; CI grün; ADRs und Landkarte mit ersten Zeilen committed.

### Phase 2 — Spielkern migrieren (Strangler-Takt)

Gate B (E7) vorab beantworten. Quellpfade = Altrepo; es bleibt parallel lesbar als reine read-only-Referenz (ohne Live-Routing — „Strangler" hier im übertragenen Sinn; präziser: inkrementelle Modul-Migration).

**Optionaler Orakel-Spike (empfohlen, vorgezogen):** Bevor der Kern auf das Orakel gebaut wird, an *einem* edge-case-schweren Modul (z. B. Ghost, sonst Phase 3) einmalig prüfen: Hält die Determinismus-Annahme (gleiche Seeds → reproduzierbare Trace)? Fangen die zugehörigen Alt-Tests einen injizierten Bug? Bestätigt sich das nicht, ist das ein Befund für die erste Session, nicht für Phase 5 — kostet ~1 Session und ersetzt die Grundsatzdebatte Migration-vs-Rewrite durch Daten.

Reihenfolge: benötigte Contracts (`src/shared/contracts/`) → Bootstrap/Loop (`main.js`, `GameBootstrap.js`, `AppInitializer.js`) → Renderer (**ohne** Recording-Pfad, P48 bleibt zurück) → Input Desktop → `ClassicModeStrategy` + `GameModeRegistry` → Entities + State.

**Fester Takt pro Modul:** kopieren → Imports anpassen → zugehörige Alt-Tests aus `tests/` mitnehmen → **Orakel-Check** (ein bewusst injizierter Bug muss von den migrierten Tests gefangen werden — sonst gelten sie nicht als Orakel und werden durch einen Golden-Master ersetzt) → Lint/Typecheck/Tests/Build selbst ausführen → Landkarten-Zeile (mit Herkunft) → Commit mit Ein-Satz-Meldung.

**Determinismus-Anker:** gleiche Seeds, gleiche Ergebnisse gegen das Altrepo; die migrierten Contract-Tests und die Ghost-Traces des Altrepos sind die Fixtures dafür. **Wenn migrierte Tests sich als schwach erweisen** (Orakel-Check schlägt fehl), wird das Orakel als **Golden-Master** direkt aus dem laufenden Altverhalten erzeugt (Seed → Trace/Score/Ghost-Spur) statt aus der Alt-Suite — das hängt nur davon ab, dass der Altcode läuft, nicht von der Test-Qualität. Golden-Master gilt für die **Simulation**; **Rendering/Optik** prüft der Playtest (Simulation ⊥ Renderer).

**Go/No-Go (Timebox):** Ist die Classic-Runde nach ~3 Arbeitssitzungen nicht spielbar, wird der Migrationsansatz bewusst neu bewertet — adressiert das Risiko „Migration wird nie fertig" (befund.md §3). Die Playtest-Checkpoints prüfen Qualität; dieser Punkt prüft, ob die Migration aus dem Ruder läuft. (Sitzungszahl anpassbar.)

**Done:** Eine Classic-Runde ist im Browser spielbar (Start → Runde → Ende), migrierte Tests grün, jede Datei in der Landkarte. **Playtest-Checkpoint.**

### Phase 3 — Modi & Content

Gate C (E8) vorab beantworten.

1. Hunt-Modus (`HuntModeStrategy.js`, `src/hunt/`).
2. Arcade inkl. Ghost (`src/core/arcade/`): `ArcadeRunRuntime.js` (1492 Zeilen) beim Umzug in 2–3 Module schneiden — einziger geplanter Zuschnitt; Ghost-Edge-Case-Tests zwingend mitnehmen. (Ghost = migrierte **Simulations**-Spur fürs Selbstduell — abzugrenzen vom Render-Recording-Pfad P48 in Phase 5.)
3. Content laut E8: Map-Presets, `data/maps/`, `data/vehicles/`, Balancing-Contracts 1:1.
4. Assets selektiv per Skript — `scripts/export-game-only-repo.mjs` aus dem Altrepo ist die Blaupause für die Auswahl.

**Done:** Alle gewählten Modi spielbar, Maps laden, zugehörige Tests grün. **Playtest-Checkpoint.**

### Phase 4 — Settings, Profile, UI

1. `src/core/settings/` + `SettingsManager.js` migrieren (der Mutationsvertrag ist erhaltenswert).
2. UI/HUD/Menü selektiv aus `src/ui/`; den P45-Hotspot (`UIStartSyncController.js`) **neu schreiben** statt migrieren.

**Done:** Spiel per Menü konfigurier- und startbar; Settings überleben den Reload.

### Phase 5 — Subsystem-Komplettierung

1. Items/Powerups/Waffen-Feinschliff laut E8.
2. Recording/Replay als Neuschreibung (P48-Wiedereinstieg — bewusst erst jetzt, mit stabilem Renderer). Dies ist der **Render**-Recording-Pfad, nicht die Ghost-Trace aus Phase 3.

**Done:** Zielumfang laut E8 vollständig spielbar. **Playtest-Checkpoint.**

### Phase 6 — Plattform-Schalen (laut E1/E10)

Je Schale ein eigener, einzeln abgeschlossener Slice: Electron frisch mit aktuellem Major (erledigt P21 — aber nur, falls Phase 6 vor der Frist 2026-07-11 erreicht wird; sonst Security-Bump ins Altrepo oder bewusste Verschiebung laut E14); Android/Capacitor mit migrierter Tilt-Steuerung; Editor/Map-Tools nur falls Target.

**Done je Schale:** eigener Build + ein Smoke-Test.

### Phase 7 — Multiplayer (nur falls E9 = ja)

`server/`-Signaling + `src/network/` migrieren; ein Playwright-Smoke für den Verbindungsweg.

### Phase 8 — Bot-Training (nur falls E13 ≠ aufgeben)

Bridge (`WebSocketTrainerBridge.js`), `python/`, Checkpoints. Voraussetzung: deterministischer Kern aus Phase 2/3. Sowohl Option 1 „einfrieren + sichern" (empfohlen — Reaktivierung erfolgt hier) als auch Option 2 „mitnehmen" führen in diese Phase; nur Option 3 „aufgeben" streicht sie.

### Phase 9 — Release & Altrepo-Stilllegung

1. E15 final beantworten; Feature-Freeze.
2. Stabilisierung: nur noch Bugfixes; voller Playtest aller E8-Inhalte durch dich.
3. Release-Artefakte je Target bauen (Installer / Browser-Deploy / APK), Version 1.0 taggen.
4. Altrepo laut E14: letzten Stand taggen, Abschlussnotiz „Nachfolger: <neues Repo>", read-only schalten, Ballast erst nach deiner Bestätigung löschen.

**Done:** 1.0 veröffentlicht laut E15; Altrepo stillgelegt.

---

## Teil 6 — Governance-Ausbaupfad (Regel folgt Schmerz)

Pipeline für jede neue Regel: Beobachtung → Finding (Changelog-Notiz) → beim **zweiten** Auftreten Regel-Kandidat → erst Warn-Check, dann blockierend.

| Auslöser (konkret beobachtet) | Dann — und erst dann — nachrüsten |
| --- | --- |
| Gleiche Bug-Klasse zum 2. Mal | Gezielter Test/Check für genau diese Klasse |
| Landkarte zum 2. Mal veraltet erwischt | Leichter CI-Check: neue `src/`-Datei braucht Landkarten-Zeile |
| Architekturgrenze (z. B. Simulation ⊥ Renderer) zum 2. Mal verletzt | ESLint-Boundary-Regel für genau diese Grenze |
| Merge bricht Spielbarkeit zum 2. Mal | Playwright-Smoke in CI |
| Datei wächst > ~500 Zeilen | Zuschnitt-ADR, kein automatisches Gate |
| AI meldet zum 2. Mal „fertig" ohne grünen Lauf | Pre-Push-Skript: Typecheck + Tests + Build als ein Befehl |
| AI verlässt zum 2. Mal den Session-Scope | Scope-Regel in `CLAUDE.md` schärfen; Slice-Definition in den Auftrag |
| Große Prosa-Wissensbasis und Suche reicht nicht | E12 neu bewerten — nur für Doku, nie für Code |

Eine Regel wird nur aktiv, wenn alle vier Punkte stimmen: Sie verhindert einen echten wiederholten Fehler. Sie ist automatisch oder sehr einfach prüfbar. Sie kostet weniger Pflege, als sie spart. Sie kann wieder entfernt werden.

---

## Teil 7 — Pflege dieses Plans

- Dieser Plan ist die **einzige** Planquelle (L7). Änderungen direkt hier, per Commit nachvollziehbar — keine Kopien, keine Parallelversionen.
- Entscheidungen: Status-Spalte in Teil 4 von „offen" auf „entschieden (ADR NNN)" setzen, sobald der ADR im neuen Repo liegt.
- Phasenfortschritt: Tabelle unten aktualisieren — mehr Status-Buchhaltung gibt es nicht.
- **Lebensende dieses Repos:** Es ist erledigt, sobald E1–E15 als ADRs im neuen Spiel-Repo liegen. Dann wird es read-only/archiviert — keine Parallelpflege neben dem Spiel-Repo (L2/L7).

| Phase | Status |
| --- | --- |
| 0 — Grundsatzentscheidungen | offen |
| 1 — Walking Skeleton | offen |
| 2 — Spielkern | offen |
| 3 — Modi & Content | offen |
| 4 — Settings & UI | offen |
| 5 — Subsystem-Komplettierung | offen |
| 6 — Plattform-Schalen | offen |
| 7 — Multiplayer | offen |
| 8 — Bot-Training | offen |
| 9 — Release | offen |
