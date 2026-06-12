# Best-Practice-Plan: Neuaufbau CurviosClash

Stand: 2026-06-12. Ersetzt `tmp/neuaufbau-anleitung.md` (v1). Basis: Befund `tmp/analyse-neuaufbau.md` + Online-Recherche (Quellen am Ende).
Entscheidungen, die nur du treffen kannst, sind als **E1–E10** markiert.

## Die fünf recherchierten Prinzipien hinter dem Plan

1. **Kein Big-Bang-Rewrite.** Joel Spolskys Klassiker nennt den Komplett-Neuschrieb "den schlimmsten strategischen
   Fehler" (Netscape 4→6: 3 Jahre ohne Release). Kernsatz: "Old code is hairy for a reason — jedes Haar ist ein
   gefixter Bug." Das deckt sich mit dem Befund: die Ghost-/Tilt-/Orientation-Edge-Cases im Altcode sind bezahltes Wissen.
   Ausnahme laut AWS/Swimm: Rewrite lohnt nur, wenn man das Altsystem "in wenigen Wochen vollständig verstehen" kann —
   bei ~98k Zeilen nicht gegeben.
2. **Strangler-Fig-Migration.** Microsoft/AWS/Fowler: Altsystem bleibt lauffähig, Neues wächst Modul für Modul daneben,
   Architektur entsteht aus realen statt theoretischen Constraints. Bekannte Falle (Future Processing): Migrationen
   versanden ohne Fokus → braucht eine feste Kadenz-Regel (siehe Phase 2).
3. **Walking Skeleton zuerst.** Henrico Dolfing/Code Climate: Als Erstes die dünnste durchgehende, *produktionsechte*
   Scheibe bauen — inklusive Build, Tests und Automation ab Tag 1. Kein Prototyp: jede Zeile bleibt.
4. **Gedächtnis = ADRs + kompaktes Agent-Briefing, nicht Apparat.** ADR-Best-Practice (Nygard-Format
   Context/Decision/Consequences, im Repo unter `docs/adr/`, "pithy", nur architektur-relevante Entscheidungen,
   Supersede statt Editieren). Claude-Code-Best-Practice: `CLAUDE.md` unter ~200 Zeilen als "operational brief",
   verschachtelte `CLAUDE.md` pro Teilbereich, stabile Regeln in die Datei statt in den Chat.
5. **Kein Code-RAG.** Mehrere unabhängige Quellen (Claude-Code-Engineering, Amazon-Science-Paper 02/2026): agentische
   Suche (grep/glob + Reasoning-Loop) schlägt Vektor-RAG auf Code deutlich; Embeddings veralten bei jedem Edit.
   RAG bleibt sinnvoll für große *Prosa*-Wissensbasen — das ist der spätere, definierte Trigger (E5).
6. **AI-geschriebener Code gilt als unverifiziert, bis Maschinen ihn geprüft haben.** Branchenkonsens 2026
   (Checkmarx, StackHawk, Graphite, CodeScene): AI-Code nie ungeprüft übernehmen; Bug-Dichte ohne Review-Disziplin
   messbar höher (+23 % in einer zitierten Erhebung). Konsequenz: maschinelle Gates (Typecheck, Lint, Tests, Build)
   sind die erste Review-Instanz, der Mensch reviewt Verhalten und Entscheidungen — nicht jede Zeile.
   Zusatzbefund: AI-*generierte* Tests sind oft schwach (~20 % Mutation-Score) — die kampferprobten Alt-Tests aus
   `tests/` sind deshalb wertvoller als neu generierte und werden bevorzugt migriert.

---

## Rollenmodell: AI schreibt den Code, du steuerst

Das Altrepo ist der Beweis, dass dieser Punkt den Ausschlag gibt: Auch dort schrieb AI den Code — und baute ohne
Grenzen den Meta-Apparat, der das Projekt erstickte. Die Stop-Regeln in `CLAUDE.md` richten sich deshalb an die AI.

**Deine Aufgaben pro Slice (Review-Einheit = Verhalten, nicht Codezeilen):**
- Auftrag geben (ein Modul/Slice pro Session — kleine Scopes verhindern Drift) und E-Fragen beantworten.
- Am Slice-Ende: Changelog-Satz + Landkarten-Diff lesen (≤ 2 Minuten) und an den Phasen-Checkpoints spielen (Playtest).
- D3-artige Entscheidungen bleiben bei dir: Löschen, Plattform-/Stack-Wechsel, neue Pflicht-Mechanismen.

**Pflichten der AI (in `CLAUDE.md` festschreiben):**
- Jeder Slice endet mit selbst ausgeführter Verifikation: Lint + Typecheck + Tests + Build grün, *bevor* "fertig"
  gemeldet wird; die Abschlussmeldung nennt in je einem Satz, was geprüft wurde und was nicht.
- Verhaltens-Tests werden aus `tests/` migriert, nicht neu generiert; neue Tests prüfen Verhalten, nicht Implementierung.
- Kein neues Modul ohne Landkarten-Zeile, keine Architekturänderung ohne ADR-Vorschlag.
- Session-Scope einhalten: keine "Beifang"-Refactorings außerhalb des beauftragten Slices.

**Optional ab Phase 3:** Zweit-Review durch separate AI-Session am Phasenende (frischer Kontext findet, was die
Schreib-Session übersieht — leichtgewichtige Variante der "validation chains" aus den Quellen).

---

## Phase 0 — Grundsatzentscheidungen (1 Gespräch, Ergebnis = ADR 001–007)

| Nr. | Entscheidung | Empfehlung (begründet) |
| --- | --- | --- |
| E1 | Targets: welche Auslieferungen, welches **eine** Start-Target? | Eines zuerst (Browser oder Desktop); Rest nach lauffähigem Kern |
| E2 | Migrieren als Default, Neuschreiben nur mit Begründung? | Ja (Spolsky/Strangler-Fig; Ausnahmen: P45-/P48-Hotspots) |
| E3 | JS behalten oder TS? three.js heben? | **Aktualisiert (AI schreibt den Code):** Typen sind kostenlose maschinelle Verifikation für die AI-Schleife. Empfehlung: neue Module direkt TS strict; migrierte Module zunächst JS 1:1 (Verhaltensanker), Konversion pro Modul später als eigener Schritt. three.js einmalig heben |
| E4 | Gedächtnis-Format bestätigen (Set siehe Phase 1, Schritt 3) | Minimal-Set; Deckel: ≤ 2 Min. Pflege pro Commit |
| E5 | RAG-Trigger | Kein Code-RAG (Quellenlage); Neubewertung nur falls eine große Prosa-Wissensbasis entsteht |
| E6 | Altrepo: read-only Referenz? Schicksal der ~390 MB `archive/`? | Read-only behalten; Archiv auslagern oder löschen |
| E7 | Bot-Training: einfrieren + sichern, mitnehmen, aufgeben? | Einfrieren + sichern (`python/`, Checkpoints, `Bot_Trainingsplan.md`); Reaktivierung frühestens nach Phase 3 |

**Done:** Alle sieben als je ein ADR im Nygard-Format (Status: Accepted) im neuen Repo.

## Phase 1 — Walking Skeleton (Ziel: 1–2 Sitzungen)

Best Practice: nicht "Repo aufsetzen, dann irgendwann Spiel", sondern die dünnste durchgehende spielbare Scheibe
inklusive aller Automation — Ende-zu-Ende, produktionsecht.

1. Repo anlegen (**E8: Name**), `.nvmrc`, Vite, three.js (Version laut E3), ESLint Standard, `node --test`.
2. **CI ab Tag 1** (Lehre aus Walking-Skeleton-Praxis, bewusst klein): ein GitHub-Actions-Workflow mit
   Lint + Unit-Tests + Build. Keine weiteren Gates, keine Pre-Commit-Hooks außer Lint.
3. Gedächtnis-Grundgerüst:
   - `CLAUDE.md` ≤ 1 Seite: Commands, Architektur-Kurzbild, Stop-Regeln (s. u.), Verweis auf Landkarte.
   - `docs/landkarte.md`: eine Zeile pro Modul (Pfad, Zweck, Abhängigkeiten, Test). Pflicht bei jedem neuen Modul.
   - `docs/adr/`: Nygard-Template (`NNN-titel.md`: Status / Kontext / Entscheidung / Konsequenzen, 5–15 Zeilen).
     Nicht editieren — supersedieren. Nur architektur-relevante Entscheidungen.
   - `docs/changelog.md`: ein Satz pro abgeschlossenem Arbeitspaket.
   - Später, wenn Teilbereiche wachsen: verschachtelte `CLAUDE.md` in `src/<bereich>/` (Claude-Code-Pattern)
     statt einer wachsenden Root-Datei.
4. Stop-Regeln in `CLAUDE.md` (Verfassung, aus dem Altrepo-Befund):
   - Kein Meta-Tooling (Graph, Index, Autopilot, Lock-Registry, Generated-Artefakte) ohne expliziten User-Auftrag.
   - Eine Wahrheitsquelle pro Thema; keine generierten Zweitkopien.
   - Doku pro Änderung: Landkarten-Zeile + Changelog-Satz, nicht mehr.
   - Max. eine aktive Plan-Datei gleichzeitig.
5. **Skeleton-Slice:** leere three.js-Szene → ein steuerbares Objekt → Game-Loop-Tick → ein Rundenende-Event →
   ein Unit-Test darauf → CI grün. Dafür die ersten Alt-Module dünn anziehen
   (`src/core/GameLoop.js`, Teile von `src/core/renderer/`, `src/core/input/`).

**Done:** `npm run dev` zeigt steuerbares Objekt; CI grün; ADR 001–008 und Landkarte mit ersten Zeilen committed.

## Phase 2 — Strangler-Fig-Migration des Spielkerns

Quellpfade = Altrepo (bleibt parallel lauffähig als Verhaltensreferenz — Kern des Strangler-Patterns).

Reihenfolge: 1. benötigte Contracts aus `src/shared/contracts/` (**E9:** 1:1 oder vereinfacht? Empfehlung 1:1 —
"replacement lightweight & disposable", Vereinfachung erst, wenn der Konsument migriert ist) →
2. Bootstrap/Loop (`main.js`, `GameBootstrap.js`, `AppInitializer.js`) → 3. Renderer komplett (ohne Recording-Pfad,
P48-Hotspot bleibt zurück) → 4. Input Desktop → 5. `ClassicModeStrategy` + `GameModeRegistry` →
6. Entities + State (`src/entities/`, `src/state/`).

**Fester Takt pro Modul** (gegen das "Versanden", die dokumentierte Strangler-Fig-Falle):
kopieren → Imports anpassen → zugehörige Alt-Tests aus `tests/` mitnehmen → Lint/Typecheck/Tests/Build selbst
ausführen → Landkarten-Zeile → Commit mit Ein-Satz-Meldung (geprüft / nicht geprüft).
Verhaltensvergleich gegen das Altrepo bei Physik/Determinismus (gleiche Seeds, gleiche Ergebnisse).
**Kadenz-Regel:** Jede Arbeitssitzung endet mit einem spielbaren Build — nie zwei Sitzungen "rot".
**Session-Regel (AI):** Eine Session = ein Modul/Slice; kein Beifang-Refactoring.

**Done:** Classic-Runde im Browser spielbar (Start→Runde→Ende), migrierte Tests grün, jede Datei in der Landkarte.
**Checkpoint mit dir:** Playtest, erst dann Phase 3.

## Phase 3 — Modi und Content

1. Hunt (`HuntModeStrategy.js`, `src/hunt/`); 2. Arcade inkl. Ghost (`src/core/arcade/` —
`ArcadeRunRuntime.js` beim Umzug in 2–3 Module schneiden; Ghost-Edge-Case-Tests zwingend mitnehmen);
3. Content 1:1: Map-Presets (15), `data/maps/`, `data/vehicles/`, Balancing-Contracts;
4. Assets selektiv per Skript (nur referenzierte der 252 MB). **E10:** alle 3 Modi / 6 Parcours behalten?

## Phase 4 — Settings, Profile, UI

`src/core/settings/` + Mutationsvertrag migrieren; `src/ui/` selektiv — **P45-Hotspot
(`UIStartSyncController.js`) neu schreiben** statt migrieren (dokumentierte Listener-Duplikation P14).

## Phase 5 — Plattform-Schalen (laut E1)

Desktop: `electron/` frisch mit aktuellem Major (erledigt Alt-Finding P21 nebenbei). Browser/Android/Editor je
nach E1 als eigene, einzeln abgeschlossene Slices.

## Phase 6 — Multiplayer (falls Target): `server/`-Signaling + `src/network/` + ein Playwright-Smoke.

## Phase 7 — Bot-Training (nur falls E7 = mitnehmen): Bridge + `python/` + Checkpoints; braucht deterministischen Kern.

## Phase 8 — Altrepo stilllegen (laut E6)

Letzten Stand taggen → Abschlussnotiz "Nachfolger: <neues Repo>" → read-only/archivieren → Ballast nach Bestätigung löschen.

---

## Governance-Ausbaupfad (Regel folgt Schmerz — nie umgekehrt)

| Auslöser (konkret beobachtet) | Dann — und erst dann — nachrüsten |
| --- | --- |
| Gleiche Bug-Klasse zum 2. Mal | Gezielter Check/Test für genau diese Klasse |
| Landkarte zum 2. Mal veraltet erwischt | Leichter CI-Check: jede neue `src/`-Datei braucht Landkarten-Zeile |
| Architekturgrenze zum 2. Mal verletzt | ESLint-Boundary-Regel für genau diese Grenze |
| Merge bricht Spielbarkeit zum 2. Mal | Playwright-Smoke in CI |
| Datei wächst > ~500 Zeilen | Zuschnitt-ADR, kein automatisches Gate |
| AI meldet zum 2. Mal "fertig" ohne grünen Verifikationslauf | Pre-Push-Skript: Typecheck + Tests + Build als ein Befehl |
| AI verlässt zum 2. Mal den Session-Scope | Scope-Erinnerung in `CLAUDE.md` schärfen; ggf. Slice-Definition in Auftrag aufnehmen |
| Große Prosa-Wissensbasis entstanden und Suche reicht nicht | RAG neu bewerten (E5) — nur für Doku, nicht für Code |

## Quellen

- Rewrite-Risiko: Spolsky, "Things You Should Never Do" (joelonsoftware.com, 2000); manuelmeyer.net; vibratingmelon.com
- Strangler Fig: learn.microsoft.com (Azure Architecture Center); docs.aws.amazon.com (Prescriptive Guidance);
  future-processing.com; swimm.io; altexsoft.com
- Walking Skeleton/Tracer Bullet: henricodolfing.com; codeclimate.com/blog; aihero.dev/tracer-bullets
- ADRs: adr.github.io; martinfowler.com/bliki/ArchitectureDecisionRecord; github.com/joelparkerhenderson/architecture-decision-record;
  AWS Architecture Blog; learn.microsoft.com (Well-Architected)
- CLAUDE.md/Agent-Memory: code.claude.com/docs/en/best-practices; anthropic.com/engineering/effective-context-engineering-for-ai-agents;
  datastudios.org (Claude Code Memory)
- RAG vs. agentische Suche: vadim.blog/claude-code-no-indexing; mindstudio.ai (2 Artikel); smartscope.blog;
  agents.siddhantkhare.com (RAG vs. Agentic Search); Amazon-Science-Paper 02/2026 (zitiert in mindstudio.ai)
- AI-geschriebener Code (Guardrails/Review/Tests): tfir.io (AI Code Quality 2026); stackhawk.com (4 Best Practices);
  graphite.com/guides (AI code review); codescene.com (AI Code Guardrails); dev.to/htekdev ("Tests Are Everything in
  Agentic AI"); dev.to/naelawadallah (TypeScript + AI-Agents); checkmarx.com (AI Security 2026)
