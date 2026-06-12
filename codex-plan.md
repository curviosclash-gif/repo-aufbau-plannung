# Codex-Plan: Neuaufbau CurviosClash

Stand: 2026-06-12. Zweck: Arbeitsplan fuer Codex-Sessions beim Neuaufbau. Dieser Plan ist bewusst operativ: welche Entscheidungen zuerst fallen, wie das Repo mitlernt, wann Wissensgraph/RAG/Second Brain entstehen und wann Governance schaerfer wird.

## Grundsatz

CurviosClash wird neu aufgebaut, aber nicht blind neu erfunden. Das Altrepo bleibt Steinbruch und Referenz. Das neue Repo startet klein, spielbar und mit leichtem Gedaechtnis.

Leitsatz:

```text
Second Brain sofort.
Wissensgraph nach stabilem Kern.
RAG nach echtem Doku-Wissen.
Governance aus wiederholten echten Problemen.
```

## Vor dem ersten Code: Entscheidungen E1-E10

Diese Entscheidungen sind User-owned. Codex darf sie vorbereiten, aber nicht stillschweigend treffen.

| ID | Entscheidung | Zu klaeren | Ergebnis |
| --- | --- | --- | --- |
| E1 | Start-Target | Browser, Desktop-Electron oder Android zuerst? | ADR 001 |
| E2 | Stack | Three.js + Vite JS, Three.js + Vite TS strict oder anderer Stack? | ADR 002 |
| E3 | Repo-Ort | neues Repo, neuer Ordner oder Bestand umbauen? | ADR 003 |
| E4 | Migrationsregel | Altcode 1:1 migrieren, neu schreiben oder je Modul entscheiden? | ADR 004 |
| E5 | Uebernahme | Assets, Maps, Balancing, Tests, Bot-Wissen, Plattformen? | `docs/memory/harvest-log.md` |
| E6 | MVP-Grenze | Was ist der kleinste spielbare Kern? | `docs/roadmap.md` |
| E7 | Second-Brain-Regeln | Wie viel Doku pro Aenderung ist Pflicht? | `docs/governance/principles.md` |
| E8 | Wissensgraph-Trigger | Wann ist genug echte Modul-Wahrheit vorhanden? | ADR 005 |
| E9 | RAG-Rolle | Suchhilfe, Erklaerhilfe, nie Entscheider? | ADR 006 |
| E10 | Bot-Training | einfrieren, spaeter migrieren oder aufgeben? | ADR 007 |

Codex-Regel: Vor Phase 1 muessen mindestens E1-E7 beantwortet sein.

## Phase 0: Second Brain v0

Codex legt zuerst nur ein leichtes Repo-Gedaechtnis an:

```text
docs/
  README.md
  vision.md
  roadmap.md
  architecture.md
  module-map.md
  testing.md
  glossary.md

  adr/
    000-template.md

  memory/
    findings.md
    harvest-log.md

  governance/
    principles.md
    rule-candidates.md
    active-rules.md
```

Pflege-Regeln:

```text
Neue Architekturentscheidung -> ADR.
Neues dauerhaftes Modul -> module-map.md.
Neue Kernmechanik -> Test oder Smoke.
Wiederholtes Problem -> findings.md.
Uebernahme aus Alt-Repo -> harvest-log.md.
```

Nicht erlaubt in Phase 0:

```text
kein Wissensgraph
kein RAG
keine Lock-Registry
kein Plan-Autopilot
keine langen Evidence-Formate
keine Generated-Artefakt-Flut
```

## Phase 1: Walking Skeleton

Ziel: Das neue Repo hat eine duenne, echte, dauerhaft spielbare Scheibe.

Codex baut in dieser Reihenfolge:

1. Repo initialisieren.
2. Vite + Three.js + Test-Runner einrichten.
3. CI mit nur drei Checks: Lint, Tests, Build.
4. Canvas startet.
5. Game Loop tickt.
6. Ein Objekt ist sichtbar.
7. Input bewegt das Objekt.
8. Ein Round-State kann starten und enden.
9. Ein Smoke-Test beweist: App startet.

Done-Kriterien:

```text
npm run dev zeigt ein steuerbares Objekt.
npm test laeuft.
npm run build laeuft.
docs/module-map.md nennt die ersten Module.
ADR 001-007 existieren.
```

## Phase 2: Spielkern

Ziel: Der kleinste echte CurviosClash-Kern laeuft.

Reihenfolge:

1. `config`: Balancing-Minimum.
2. `input`: Tastatur/Controller-Grundlage.
3. `render`: Szene, Kamera, einfache Arena.
4. `simulation`: Bewegung und Timestep.
5. `entities`: Player und Dummy/Gegner.
6. `collision`: Arena-Grenzen und Treffer.
7. `round-state`: Start, Ende, Reset.
8. `ui`: minimale Anzeige fuer Zustand.

Pro Modul arbeitet Codex nach diesem Takt:

```text
1. Zweck in module-map.md nennen.
2. Altcode pruefen: migrieren oder neu schreiben?
3. Passenden Alt-Test suchen.
4. Mindestens ein Test oder Smoke.
5. Lint/Test/Build laufen lassen.
6. Changelog: ein Satz, wenn das Modul dauerhaft ist.
```

Stop-Regel: Nie zwei Sessions hintereinander mit nicht spielbarem Build enden.

## Phase 3: Wissensgraph v0

Der Wissensgraph kommt erst, wenn echte Modulstruktur existiert.

Start-Gate:

```text
Spielkern laeuft.
5-10 dauerhafte Module existieren.
docs/module-map.md ist aktuell.
Kern-Tests laufen.
Modulgrenzen sind erkennbar.
```

Graph v0 enthaelt nur einfache Fakten:

```text
module
owns
imports
depends_on
tested_by
described_by
decided_by
```

Codex darf den Graphen nutzen als:

```text
Radar fuer Abhaengigkeiten.
Hinweis auf fehlende Tests.
Orientierung fuer naechste Reads.
```

Codex darf den Graphen nicht nutzen als:

```text
alleinige Source of Truth.
Abschlussbeweis.
Freigabeersatz.
Grund fuer neue Governance ohne Finding.
```

## Phase 4: RAG v0

RAG kommt nach dem Wissensgraphen und nur fuer Prosa-Wissen.

Start-Gate:

```text
docs/architecture.md ist aktuell.
docs/adr/*.md enthalten echte Entscheidungen.
docs/module-map.md ist aktuell.
generated/knowledge-graph.json existiert.
Mindestens 20 Testfragen existieren.
Antworten muessen Quellen nennen.
```

RAG darf beantworten:

```text
Warum gibt es dieses Modul?
Welche ADR betrifft den Game Loop?
Welche Tests schuetzen Collision?
Welche Alt-Assets wurden uebernommen?
Welche Findings gab es schon zu Round-State?
```

RAG darf nicht entscheiden:

```text
Diese Regel ist bindend.
Dieser Plan ist abgeschlossen.
Dieser Code darf gemerged werden.
Diese Architektur ist richtig.
```

RAG-Regel:

```text
Keine Quelle -> keine Antwort.
Widerspruch zwischen RAG und Datei -> Datei gewinnt.
```

## Phase 5: Governance aus Gedaechtnis ableiten

Governance entsteht nicht aus Wunscharchitektur, sondern aus wiederholten Befunden.

Pipeline:

```text
Beobachtung -> Finding -> Rule Candidate -> Warn-Check -> Active Rule
```

Beispiel:

```text
Beobachtung:
UI greift mehrfach direkt in Simulation ein.

Finding:
UI/Simulation-Grenze ist wiederholt unscharf.

Rule Candidate:
UI darf Simulation nicht direkt mutieren.

Warn-Check:
CI warnt, blockiert aber noch nicht.

Active Rule:
Nach Stabilisierung blockiert CI neue Verletzungen.
```

Eine Regel wird nur aktiv, wenn alle Punkte stimmen:

```text
Sie verhindert einen echten wiederholten Fehler.
Sie ist automatisch oder sehr einfach pruefbar.
Sie kostet weniger Pflege, als sie spart.
Sie kann wieder entfernt werden.
```

## Phase 6: Subsysteme ausbauen

Erst nach stabilem Kern:

1. Waffen, Treffer, Schaden.
2. Items und Powerups.
3. mehrere Maps.
4. Menue und UI.
5. Arcade-Run und Progression.
6. Hangar und Fahrzeuge.
7. Recording und Replay.
8. Multiplayer.
9. Android.
10. Bot-Training und AI.

Jedes Subsystem braucht:

```text
ein klares Modulziel
einen Test/Smoke
einen Eintrag in module-map.md
bei Architekturentscheidung einen ADR
keine neue Governance ohne wiederholtes Problem
```

## Codex-Arbeitsregeln

Diese Regeln gelten in jeder Codex-Session:

```text
Ein Slice pro Session.
Kein Beifang-Refactoring.
Keine neue Pflichtregel ohne Finding.
Kein globales Runtime-Allzweckobjekt.
Kein Altcode-Import ohne Harvest-Log.
Kein "fertig" ohne gelaufene Verifikation.
Keine RAG-Indizes ueber Wunscharchitektur.
```

Abschlussmeldung je Slice:

```text
Was wurde gebaut?
Welche Dateien sind dauerhaft wichtig?
Welche Tests/Checks liefen?
Was wurde bewusst nicht geprueft?
Welche Entscheidung bleibt beim User?
```

## Erste Arbeitssequenz

Wenn E1-E7 beantwortet sind:

```text
1. Neues Spielrepo anlegen.
2. docs/ und ADR-Template anlegen.
3. Stack/Target-ADRs schreiben.
4. Vite/Three/Test/CI einrichten.
5. Leere Szene starten.
6. Steuerbares Objekt bauen.
7. Round-State-Minimum bauen.
8. Ersten Smoke-Test schreiben.
9. module-map.md aktualisieren.
10. Playtest: laeuft die duennste Scheibe?
```

Erst danach beginnt die gezielte Ernte aus dem Alt-Repo.
