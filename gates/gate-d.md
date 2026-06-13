# Gate D — Plattform-Schalen & Multiplayer (E9–E10)

Fällig: vor Phase 6/7. Faktenbasis: [`../befund.md`](../befund.md). Hängt an E1 (Start-Target).

---

## E9 — Multiplayer im Zielumfang?

**Frage:** Kommt der LAN-Multiplayer ins neue Repo (1.0) — oder erst nach Release?

**Fakten:**
- Kronjuwel vorhanden: `server/` (LAN-Signaling, `lan-signaling.js`, `signaling-server.js`) + `src/network/`. V99 hat Signaling/Connectivity gehärtet.
- Multiplayer ist eine eigene Phase (7) mit eigenem Playwright-Smoke für den Verbindungsweg — zusätzlicher Test- und Wartungsaufwand.
- Der Kern ist auch ohne MP voll spielbar (Classic/Hunt/Arcade sind Single-Device).

**Optionen:**
1. **MP in 1.0** — Phase 7 wird Teil des Release-Scopes.
2. **Singleplayer-Release, MP als 1.x** — schlankeres 1.0, MP migrieren, wenn der Kern steht.

**Empfehlung:** keine — abhängig davon, ob MP für dich Release-kritisch ist. Falls unsicher: Option 2 (kleineres 1.0, schnellerer Release).

**Entscheidung:** ____ (Datum)

---

## E10 — Reihenfolge der Plattform-Schalen

**Frage:** In welcher Reihenfolge entstehen die Schalen laut E1-End-Target-Liste?

**Fakten:**
- Electron frisch mit aktuellem Major aufgesetzt **erledigt Alt-Finding P21 nebenbei** (Electron-Security-Doppel-Major, Wiedervorlage 2026-07-11 — siehe E14). **Achtung Frist:** Phase 6 liegt hinter Phase 0–5; das neue Repo wird die Frist 2026-07-11 fast sicher nicht vorher erreichen. Für die Deadline trägt „Phase 6 erledigt P21" daher **nicht** — bis dahin muss der Security-Bump ins Altrepo oder die Frist bewusst verschoben werden (E14).
- Android/Capacitor: die zuletzt gehärtete Tilt-/Orientation-Steuerung (V131) ist bezahltes Wissen und wird migriert; kostet manuelle Device-Smokes.
- Editor/Map-Tools sind nur relevant, wenn sie in der End-Target-Liste (E1) stehen.

**Optionen:**
1. **Electron → Android → (Editor)** — Desktop zuerst, löst P21 früh; Mobile danach.
2. **Android zuerst** — nur sinnvoll, wenn Mobile das Leitprodukt ist.
3. Reihenfolge frei nach deiner End-Target-Liste.

**Empfehlung:** Option 1, sofern die End-Target-Liste Desktop enthält. Jede Schale ist ein eigener, einzeln abgeschlossener Slice mit eigenem Build + Smoke-Test.

**Entscheidung:** ____ (Datum; Schalen-Reihenfolge)
