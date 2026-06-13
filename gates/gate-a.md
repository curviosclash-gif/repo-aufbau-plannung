# Gate A — vor dem ersten Commit (E1–E6)

Fällig: Phase 0, bevor das neue Repo angelegt wird. Faktenbasis: [`../befund.md`](../befund.md).
Du füllst nur die Zeile **Entscheidung** je Block aus — oder antwortest im Chat, die AI trägt nach.
Beim Anlegen des neuen Repos (Phase 1) werden die sechs Entscheidungen dort als ADR 001–006 übertragen.

---

## E1 — Ziel-Targets und ein Start-Target

**Frage:** Welche Auslieferungen soll das neue Repo am Ende haben — und mit welcher **einen** wird gestartet?

**Fakten:**
- Das Altrepo bediente ~7 Lanes: Desktop-Electron, Browser, Android-Classic, Android-Map-Tools, Editor, LAN-Multiplayer, Trainings-Stack. Jede Lane = eigene Build-/Test-Strecke (Befund §4.1).
- Der Spielkern ist Web-Technik (Vite + three.js). Der Browser-Build ist der kürzeste Weg zu „jede Session spielbar"; Electron und Android sind Schalen darüber.
- Der Produktfokus des Altrepos war Desktop-first.
- Die Android-Tilt-Steuerung wurde zuletzt gehärtet (V131, 2026-06-11) — bezahltes Wissen; die Mobile-Lane kostet aber manuelle Device-Smokes.

**Optionen:**
1. **Browser zuerst** — schnellste Iteration, jede Session direkt im Browser verifizierbar, einfachste CI. Desktop-Eigenheiten (Fenster, Dateipfade, Save-Verzeichnis) kommen erst mit der Electron-Schale in Phase 6.
2. **Desktop-Electron zuerst** — entspricht dem alten Produktfokus; dafür Schale und Kern gleichzeitig, langsamere Schleife, Packaging-Themen ab Tag 1.
3. **Android zuerst** — schwerste Lane (Geräte-Tests manuell); spricht nichts dafür.

**Empfehlung:** Browser als Start-Target; Electron als erste Schale in Phase 6, Android danach. Der Spielkern ist in allen Fällen identisch — entschieden wird nur die Reihenfolge. Die End-Target-Liste kannst du hier grob notieren; final beschnitten wird sie in Gate D.

**Entscheidung:** ____ (Datum; Start-Target + grobe End-Target-Liste)

---

## E2 — Stack: TS/JS und three.js-Version

**Frage:** Bekommen neue Module TypeScript? Bleiben migrierte Module JS? Wird three.js gehoben?

**Fakten:**
- Altbestand: 574 JS-Dateien (~98k Zeilen), plain JS; three.js `^0.160.0` (Stand ~Ende 2023, mehrere Releases hinter aktuell).
- Learnings: Maschinelle Gates sind die erste Review-Instanz für AI-Code (L6); migrierte Module sollen unveränderte Verhaltensanker bleiben (L5). Typen sind kostenlose maschinelle Verifikation für die AI-Schleife.
- Weil die Simulation renderer-unabhängig ist (Teil-1-Prinzip), gefährdet ein three.js-Hub den Determinismus-Anker aus Phase 2 nicht — Breaking Changes treffen nur die Render-Schicht.

**Optionen:**
1. **Neue Module TS strict, migrierte JS 1:1, Konversion später pro Modul** — Typen für neuen Code gratis, Migration bleibt Verhaltensanker. Kostet eine bewusste `tsconfig`-Festlegung (`allowJs: true`, kein `checkJs`-Zwang für Altmodule).
2. **Alles JS + JSDoc** — homogen, aber schwächere maschinelle Prüfung für neuen Code.
3. **Alles TS sofort (auch Migration)** — jede Migration wird Teil-Neuschreibung; Verhaltensanker verloren.

three.js: **einmalig heben in Phase 1** (Breaking Changes zahlen, solange fast kein Code da ist) vs. **alt pinnen** (maximale Verhaltensgleichheit, aber die Schuld wächst und wird später teurer).

**Empfehlung:** Option 1 + three.js einmalig in Phase 1 heben. Die Typecheck-Abdeckung wird im ADR explizit festgehalten (was prüft `tsc` ab Tag 1, was erst nach Konversion).

**Entscheidung:** ____ (Datum; TS-Regel + three.js-Entscheid)

---

## E3 — Migrationsregel

**Frage:** Gilt „Migrieren als Default, Neuschreiben nur mit Begründung"?

**Fakten:**
- Der Altcode ist gesund (Befund §1); die Edge-Cases (Ghost-Persistenz, Tilt-Kalibrierung, Orientation) sind teuer bezahlte Bugs (L5).
- Dokumentierte Ausnahmen, bei denen Neuschreiben geplant ist: P45 (`UIStartSyncController.js`, 772 Zeilen, Listener-Duplikation P14 — Phase 4) und P48 (Recording-Pfad — Phase 5).
- Branchenbeleg gegen Rewrites und für Strangler-Migration steht im Plan (Teil 2, L5) und in der Historie der Quellpläne.

**Optionen:**
1. **Migrieren als Default**, Neuschreiben nur mit Begründung (Ausnahmen P45/P48 vorab genehmigt; jede weitere braucht einen Satz Begründung im Changelog).
2. **Pro Modul frei entscheiden** — flexibler, aber jede Session beginnt mit einer Grundsatzdiskussion und der Rewrite-Anteil wächst schleichend.

**Empfehlung:** Option 1.

**Entscheidung:** ____ (Datum)

---

## E4 — Repo-Struktur

**Frage:** Flache `src/`-Gliederung oder Packages-Monorepo?

**Fakten:**
- Die flache Alt-Struktur (`src/core`, `src/modes` mit Strategy-Pattern, `src/entities`, `src/state`, `src/shared/contracts`, `src/ui`) war laut Befund ausdrücklich gesund — das Problem lag nie in der Code-Struktur.
- Der Monorepo-Vorschlag (`packages/game-core`, `packages/game-renderer` …) stammt aus dem generischen Playground-Plan und kostet Workspace-Tooling ab Tag 1.
- Die Architekturregel „Simulation darf nie vom Renderer abhängen" gilt unabhängig von der Ordnerform; durchgesetzt wird sie laut Teil 6 erst nach der zweiten Verletzung (dann als gezielte ESLint-Boundary-Regel).

**Optionen:**
1. **Flach starten:** `src/core`, `src/render`, `src/input`, `src/modes`, `src/entities`, `src/state`, `src/contracts`, `src/ui` + `data/`, `assets/`. Monorepo erst, wenn ein zweites echtes Artefakt (z. B. Editor) die Trennung zweimal schmerzhaft macht (L1).
2. **Packages-Monorepo ab Tag 1:** erzwingt die Grenzen über Paket-Abhängigkeiten, kostet aber sofort Workspace-Konfiguration und macht jeden Migrationsschritt schwerer.

**Empfehlung:** Option 1.

**Entscheidung:** ____ (Datum)

---

## E5 — Gedächtnis-Format

**Frage:** Reicht das Minimal-Set als Pflicht-Gedächtnis?

**Fakten:**
- Altrepo: vier parallele Wahrheitsquellen + Evidence-Pflicht pro Checkbox → 36 % reine Doku-Commits, Abgleich als eigener Arbeitstyp (L2, L8).
- Minimal-Set: `CLAUDE.md` (≤ 1 Seite: Commands, Architektur-Kurzbild, Stop-Regeln), `docs/landkarte.md` (eine Zeile pro Modul: Pfad, Zweck, Abhängigkeiten, Test, Herkunft alt/neu), `docs/adr/` (Nygard + Optionen-Abschnitt), `docs/changelog.md` (ein Satz pro Arbeitspaket).
- Harter Deckel: Gedächtnispflege ≤ 2 Minuten pro Commit — sonst ist das Format falsch.

**Optionen:**
1. **Minimal-Set** wie beschrieben; Playtest-Notizen als datierter Abschnitt im Changelog.
2. **Minimal-Set + Extras** (eigenes Playtest-Journal, Balancing-Journal, Skizzenordner) — nur wählen, wenn du es wirklich pflegen willst; jedes Extra ist eine weitere Quelle.

**Empfehlung:** Option 1.

**Entscheidung:** ____ (Datum; bei Option 2: welche Extras)

---

## E6 — Repo-Name und Hosting

**Frage:** Wie heißt das neue Repo und wo liegt es?

**Fakten:**
- CI ab Tag 1 (Phase 1) braucht ein Hosting mit Actions; dieses Planungs-Repo liegt bereits unter `github.com/curviosclash-gif`.
- Der Altrepo-Ordner heißt `CurviosCLash` (uneinheitliche Schreibweise) — ein konsistenter neuer Name spart dauerhafte Pfad-/Suchverwirrung.

**Optionen:**
1. **GitHub (privat) unter `curviosclash-gif`** — CI, Mehrgeräte-Zugriff, gleiche Organisation wie dieses Repo.
2. **Nur lokal** — kein CI-ab-Tag-1; widerspricht Phase 1.

**Empfehlung:** Option 1. Namensvorschläge ohne Präferenz: `curviosclash`, `curviosclash-game`, `cc-game`.

**Entscheidung:** ____ (Datum; Name + Hosting)
