# repo-aufbau-plannung

Planungs-Repo für den Neuaufbau von **CurviosClash**. Job dieses Repos: die Grundsatz- und
Umsetzungsentscheidungen (E1–E15) entscheidungsreif machen. **Gebaut wird im neuen Spiel-Repo**, nicht hier.

## Einstieg

1. [`plan.md`](plan.md) — der **kanonische Plan**: Spielidee (gesetzt), Learnings aus dem Altrepo (L1–L8),
   Arbeitsmodell, Bauphasen 0–9, Governance. Die einzige Plan-Datei (L7).
2. [`befund.md`](befund.md) — read-only Faktenbasis (Altrepo-Analyse), auf die sich alle Empfehlungen berufen.
3. [`gates/`](gates/) — ein Entscheidungsblatt pro Gate (A–F) mit Fakten, Optionen, Empfehlung und
   Entscheidungszeile. **Hier triffst du die Entscheidungen.**

| Gate | Entscheidungen | Fällig |
| --- | --- | --- |
| [A](gates/gate-a.md) | E1–E6: Target, Stack, Migrationsregel, Struktur, Gedächtnis, Name | vor dem ersten Commit |
| [B](gates/gate-b.md) | E7: Contracts 1:1 | vor der Kern-Migration |
| [C](gates/gate-c.md) | E8: Übernahme-Umfang (Modi/Maps/Assets) | vor Modi & Content |
| [D](gates/gate-d.md) | E9–E10: Multiplayer, Schalen-Reihenfolge | vor Schalen/Multiplayer |
| [E](gates/gate-e.md) | E11–E12: Wissensgraph, RAG — **Default: nein** | nur bei Schmerz-Trigger |
| [F](gates/gate-f.md) | E13–E15: Bot-Training, Altrepo-Erbe, Release 1.0 | Erbe ab Migration, Release vor Phase 9 |

## Prinzip

Eine Wahrheitsquelle pro Thema (L2): `plan.md` ist der Plan, `befund.md` die Fakten, `gates/` die
Entscheidungen — keine Parallelkopien. Sobald eine Entscheidung fällt, wird sie im neuen Spiel-Repo als ADR
abgelegt und in `plan.md` Teil 4 mit der ADR-Nummer markiert.
