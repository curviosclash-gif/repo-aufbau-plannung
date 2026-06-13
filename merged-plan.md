# Merged-Plan: Neuaufbau CurviosClash

Stand: 2026-06-13. Führt `claude-plan.md` (recherchierter Begründungsplan) und `codex-plan.md`
(operativer Arbeitsplan) zu einem Plan zusammen und löst die drei Diskrepanzen zwischen beiden auf.

Tragende Korrektur gegenüber beiden Einzelplänen: **Wissensgraph und RAG sind keine geplanten Phasen,
sondern verdiente Konditionalfeatures** — das hält den Plan konsistent mit der Kernlehre des Altrepos
("der Meta-Apparat hat das Projekt erstickt"). Entscheidungen, die nur der User treffen kann, sind als
**D1–D11** markiert.

---

# Teil B — Diskrepanzen aufgelöst (Entscheidungsvorlage)

## B.1 Die drei echten Konflikte zwischen den Plänen

**Konflikt 1 — Wissensgraph & RAG: geplante Phase vs. verbotenes Meta-Tooling**
- `codex-plan` baut sie als feste Phasen 3 (Wissensgraph v0) und 4 (RAG v0) ein.
- `claude-plan` verbietet beides per Stop-Regel ("Kein Meta-Tooling … ohne expliziten User-Auftrag",
  "Kein Code-RAG").
- **Auflösung:** Claudes Haltung gewinnt als Default — genau das ist die Lehre des Altrepos. Codex'
  *Start-Gates* werden dabei nicht verworfen, sondern zur **Mindestbedingung** (nicht zum Auslöser).
  Heißt: Graph/RAG sind **keine geplanten Phasen**, sondern stehen in der Trigger-Tabelle (A.2). Gebaut
  wird erst, wenn **(1)** ein echter, zweifach beobachteter Schmerz vorliegt **und (2)** das jeweilige
  Gate erfüllt ist **und (3)** der User explizit zustimmt. Default bleibt: kein Graph, kein RAG, nur
  agentische Suche (grep/glob).

**Konflikt 2 — "E1–E10" meint in beiden Plänen etwas anderes**
- Beide nummerieren E1–E10, aber claude-E3 (JS/TS) ≠ codex-E2, claude-E6 (Altrepo-Schicksal) ≠ codex-E5
  usw. ADR-Referenzen würden aneinander vorbeilaufen.
- **Auflösung:** Eine kanonische Tabelle **D1–D11** (B.2). Die alten E-Nummern werden darin nur noch als
  Herkunftsnachweis geführt.

**Konflikt 3 — Repo-Ort: explizit vs. implizit**
- Nur `codex-plan` macht es zur Entscheidung (neues Repo / neuer Ordner / Bestand umbauen). `claude-plan`
  setzt "neues Repo" stillschweigend voraus (E8 = nur Name).
- **Auflösung:** Wird zur expliziten frühen Entscheidung **D2**. Empfehlung: **neues Repo** (beide Pläne
  brauchen das Altrepo parallel-lauffähig als Verhaltensreferenz — das geht im selben Ordner nicht sauber).

## B.2 Kanonische Entscheidungstabelle (vor Phase 1 mindestens D1–D9 beantworten)

| ID | Entscheidung | Herkunft claude / codex | Empfehlung (aufgelöst) | Artefakt |
| --- | --- | --- | --- | --- |
| **D1** | **Start-Target** — welche *eine* Auslieferung zuerst? | E1 / E1 | Genau eines (Browser **oder** Desktop); Rest erst nach lauffähigem Kern | ADR 001 |
| **D2** | **Repo-Ort + Name** | E8 / E3 | Neues Repo (Altrepo bleibt parallel-lauffähig); Name = User | ADR 002 |
| **D3** | **Stack** — JS, TS strict, three.js-Bump? | E3 / E2 | Neue Module **TS strict** (Typen = gratis Maschinen-Verifikation); migrierte Module zunächst **JS 1:1** als Verhaltensanker, TS-Konversion später als eigener Schritt; three.js einmalig heben | ADR 003 |
| **D4** | **Migrationsregel** — migrieren / neu schreiben / pro Modul? | E2 / E4 | Migrieren als Default, Neuschreiben nur mit Begründung (Ausnahme: P45-/P48-Hotspots) | ADR 004 |
| **D5** | **MVP-Grenze** — kleinster spielbarer Kern? | – / E6 | Classic-Runde: Start → Steuern → Rundenende; alles andere später | `docs/roadmap.md` |
| **D6** | **Übernahme-Scope** — Assets, Maps, Balancing, Tests, Contracts, Modi, Plattformen | E9 (Contracts 1:1), E10 (Modi/Parcours) / E5 | Contracts **1:1** (Vereinfachung erst nach Migration des Konsumenten); nur referenzierte Assets per Skript; Modi/Parcours-Umfang = User (D5-abhängig) | `docs/memory/harvest-log.md` |
| **D7** | **Second-Brain-Regeln** — wie viel Doku pro Änderung ist Pflicht? | E4 / E7 | Minimal-Set: Landkarten-Zeile + Changelog-Satz; ADR nur bei Architekturentscheidung; Deckel **≤ 2 Min Pflege pro Commit** | `docs/governance/principles.md` |
| **D8** | **Altrepo-Schicksal** — read-only? 390 MB `archive/`? | E6 / (Teil E5) | Read-only als Referenz behalten; `archive/` auslagern oder nach Bestätigung löschen; am Ende stilllegen (Phase 8) | ADR 005 |
| **D9** | **Bot-Training** — einfrieren / mitnehmen / aufgeben? | E7 / E10 | Einfrieren + sichern (`python/`, Checkpoints, Plan); Reaktivierung frühestens nach stabilem deterministischem Kern | ADR 006 |
| **D10** | **Wissensgraph** — ob/wann überhaupt? | (Stop-Regel) / E8 | **Konditional, keine Phase.** Default nein. Bau nur bei Schmerz-Trigger + Gate + User-OK → A.2 | (entsteht nur bei Trigger) |
| **D11** | **RAG** — ob/wann/Rolle? | E5 / E9 | **Konditional, keine Phase.** Nie für Code, nur für große Prosa-Wissensbasis, nie Entscheider ("keine Quelle → keine Antwort; Datei schlägt RAG") → A.2 | (entsteht nur bei Trigger) |

---

# Teil A — Konsolidierter Neuaufbau-Plan

> Vereint Codex' operative Schritt-für-Schritt-Sequenz mit Claudes recherchierten Stop-Regeln.

## A.0 Leitsatz & Rollenmodell

```text
Kein Big-Bang-Rewrite. Strangler-Fig, Altrepo bleibt Referenz.
Walking Skeleton zuerst — produktionsecht, jede Zeile bleibt.
Leichtes Gedächtnis sofort. Graph/RAG nur aus echtem Schmerz.
Governance folgt Schmerz — nie umgekehrt.
AI schreibt den Code, der User steuert Verhalten.
```

**AI schreibt, User steuert.** Das Altrepo beweist: Auch dort schrieb AI den Code — und baute ungebremst
den Meta-Apparat, der das Projekt erstickte. Die Stop-Regeln richten sich deshalb an die AI.

- **User pro Slice:** Auftrag (ein Modul/Slice je Session), D-Fragen beantworten, am Slice-Ende
  Changelog-Satz + Landkarten-Diff lesen (≤ 2 Min), an Phasen-Checkpoints spielen. Eigentumsentscheidungen
  (Löschen, Stack-/Plattformwechsel, neue Pflichtregel) bleiben beim User.
- **AI-Pflichten:** kein "fertig" ohne selbst gelaufene Verifikation (Lint + Typecheck + Tests + Build
  grün); Abschlussmeldung nennt je einen Satz "geprüft / bewusst nicht geprüft"; Verhaltens-Tests aus
  `tests/` **migrieren statt generieren**; kein neues Modul ohne Landkarten-Zeile; keine
  Architekturänderung ohne ADR-Vorschlag; kein Beifang-Refactoring.

## A.1 Die Phasen

### Phase 0 — Entscheidungen + Second Brain v0 *(1 Gespräch)*

D1–D9 beantworten und als ADR 001–006 / `roadmap.md` / `harvest-log.md` / `principles.md` festschreiben.
Gleichzeitig das leichte Gedächtnis anlegen:

```text
docs/  README.md vision.md roadmap.md architecture.md module-map.md testing.md glossary.md
       adr/000-template.md   (Nygard: Status/Kontext/Entscheidung/Konsequenzen, 5–15 Z., supersede statt editieren)
       memory/findings.md harvest-log.md
       governance/principles.md rule-candidates.md active-rules.md
```

**Stop-Regeln (Verfassung in `CLAUDE.md`, ≤ 1 Seite):** kein Meta-Tooling (Graph, Index, Autopilot,
Lock-Registry, Generated-Flut) ohne expliziten Auftrag · eine Wahrheitsquelle pro Thema, keine generierten
Zweitkopien · Doku pro Änderung = Landkarten-Zeile + Changelog-Satz · max. **eine** aktive Plan-Datei.

**Done:** D1–D9 als ADRs/Docs committed; `docs/`-Gerüst steht.

### Phase 1 — Walking Skeleton *(1–2 Sitzungen)*

Dünnste durchgehende, produktionsechte Scheibe inkl. aller Automation:

1. Repo (D2), `.nvmrc`, Vite, three.js (D3), ESLint, `node --test`.
2. **CI ab Tag 1**, bewusst klein: **nur Lint + Tests + Build**. Keine weiteren Gates, kein Pre-Commit-Hook
   außer Lint.
3. Skeleton-Slice: leere three.js-Szene → ein steuerbares Objekt → Game-Loop-Tick → ein Rundenende-Event →
   ein Unit-Test darauf → CI grün. Dafür erste Alt-Module dünn anziehen (`src/core/GameLoop.js`, Teile
   `renderer/`, `input/`).

**Done:** `npm run dev` zeigt ein steuerbares Objekt; `npm test` & `npm run build` laufen; CI grün;
`module-map.md` mit ersten Zeilen.

### Phase 2 — Spielkern via Strangler-Fig

Quellpfade = Altrepo (bleibt parallel-lauffähig als Verhaltensreferenz). Reihenfolge:

1. benötigte Contracts aus `src/shared/contracts/` (1:1, D6) → 2. Bootstrap/Loop (`main.js`,
   `GameBootstrap.js`, `AppInitializer.js`) → 3. Renderer (ohne Recording-Pfad, P48-Hotspot bleibt zurück)
   → 4. Input Desktop → 5. `ClassicModeStrategy` + `GameModeRegistry` → 6. Entities + State.

**Takt pro Modul:** kopieren → Imports anpassen → Alt-Tests aus `tests/` mitnehmen →
Lint/Typecheck/Tests/Build selbst laufen lassen → Landkarten-Zeile → Commit mit Ein-Satz-Meldung.
Physik/Determinismus gegen Altrepo prüfen (gleiche Seeds → gleiche Ergebnisse).

**Kadenz-Regel:** Jede Sitzung endet mit spielbarem Build — **nie zwei Sitzungen rot.** Eine Session = ein
Modul.

**Done + Checkpoint:** Classic-Runde im Browser spielbar (Start→Runde→Ende), migrierte Tests grün; danach
Playtest mit dir, erst dann Phase 3.

### Phase 3 — Modi & Content

Hunt (`HuntModeStrategy.js`, `src/hunt/`); Arcade inkl. Ghost (`ArcadeRunRuntime.js` beim Umzug in 2–3
Module schneiden, Ghost-Edge-Case-Tests **zwingend** mitnehmen); Content 1:1 (Map-Presets, `data/maps/`,
`data/vehicles/`, Balancing-Contracts); Assets selektiv per Skript (nur referenzierte). Umfang Modi/Parcours
laut D5/D6.

### Phase 4 — Settings, Profile, UI

`src/core/settings/` + Mutationsvertrag migrieren; `src/ui/` selektiv — **P45-Hotspot
(`UIStartSyncController.js`) neu schreiben** statt migrieren (dokumentierte Listener-Duplikation P14).

### Phase 5 — Plattform-Schalen *(laut D1)*

Desktop: `electron/` frisch mit aktuellem Major (erledigt Alt-Finding P21 nebenbei). Browser/Android/Editor
je nach D1 als eigene, einzeln abgeschlossene Slices.

### Phase 6 — Multiplayer *(falls Target)*

`server/`-Signaling + `src/network/` + ein Playwright-Smoke.

### Phase 7 — Bot-Training *(nur falls D9 = mitnehmen)*

Bridge + `python/` + Checkpoints; braucht deterministischen Kern.

### Phase 8 — Altrepo stilllegen *(laut D8)*

Letzten Stand taggen → Abschlussnotiz "Nachfolger: <neues Repo>" → read-only/archivieren → Ballast nach
Bestätigung löschen.

## A.2 Governance-Ausbaupfad — Regel folgt Schmerz (inkl. Graph & RAG)

Pipeline für jede neue Regel: `Beobachtung → Finding → Rule Candidate → Warn-Check (CI warnt) →
Active Rule (CI blockt)`. Eine Regel wird nur aktiv, wenn sie einen echten wiederholten Fehler verhindert,
automatisch/einfach prüfbar ist, weniger Pflege kostet als sie spart, und wieder entfernbar ist.

| Auslöser (konkret 2× beobachtet) | Dann — und erst dann — nachrüsten |
| --- | --- |
| Gleiche Bug-Klasse zum 2. Mal | Gezielter Check/Test für genau diese Klasse |
| Landkarte zum 2. Mal veraltet erwischt | CI-Check: jede neue `src/`-Datei braucht Landkarten-Zeile |
| Architekturgrenze (z. B. UI mutiert Simulation) zum 2. Mal verletzt | ESLint-Boundary-Regel für genau diese Grenze |
| Merge bricht Spielbarkeit zum 2. Mal | Playwright-Smoke in CI |
| AI meldet zum 2. Mal "fertig" ohne grünen Lauf | Pre-Push-Skript: Typecheck + Tests + Build als ein Befehl |
| AI verlässt zum 2. Mal den Session-Scope | Scope-Erinnerung in `CLAUDE.md` schärfen / Slice in Auftrag definieren |
| **Orientierung im Modulnetz kostet wiederholt spürbar Zeit** UND Gate erfüllt (Kern läuft, 5–10 dauerhafte Module, `module-map.md` aktuell) UND User-OK | **Wissensgraph v0 (D10)** — nur Fakten `module/imports/depends_on/tested_by/decided_by`; Radar, **nie** Source-of-Truth/Abschlussbeweis/Freigabeersatz |
| **Große Prosa-Wissensbasis entstanden** UND Suche reicht nachweislich nicht UND Gate erfüllt (architecture.md + echte ADRs aktuell, ≥ 20 Testfragen, Antworten mit Quellenzwang) UND User-OK | **RAG v0 (D11)** — nur Doku, nie Code; nie Entscheider; "keine Quelle → keine Antwort", Datei schlägt RAG |

## A.3 AI-Arbeitsregeln & Abschlussmeldung

**Jede Session:** ein Slice · kein Beifang-Refactoring · keine neue Pflichtregel ohne Finding · **kein
globales Runtime-Allzweckobjekt** · kein Altcode-Import ohne `harvest-log.md`-Eintrag · kein "fertig" ohne
gelaufene Verifikation · keine RAG/Graph-Artefakte über Wunscharchitektur.

**Abschlussmeldung je Slice:** Was gebaut? · Welche Dateien dauerhaft wichtig? · Welche Tests/Checks liefen?
· Was bewusst nicht geprüft? · Welche Entscheidung bleibt beim User?

**Optional ab Phase 3:** Zweit-Review durch separate AI-Session am Phasenende (frischer Kontext findet, was
die Schreib-Session übersieht).

---

## Herkunft

- `claude-plan.md` — recherchierter Begründungsplan (Spolsky, Fowler, Microsoft/AWS, Walking Skeleton, ADRs,
  RAG-vs-agentische-Suche, AI-Code-Guardrails). Quellenliste dort.
- `codex-plan.md` — operativer Arbeitsplan (docs/-Baum, Phasen-Gates, Second-Brain-Pflegeregeln,
  Codex-Arbeitsregeln).
- Dieser Merge: 2026-06-13.
