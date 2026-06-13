# Gate F — Erbe & Release (E13–E15)

Fällig: E13/E14 ab Migrationsstart relevant, E15 vor Phase 9. Faktenbasis: [`../befund.md`](../befund.md).

---

## E13 — Bot-Training

**Frage:** Was passiert mit dem Trainings-Stack?

**Fakten:**
- Vorhanden: `python/` (PPO `train.py`/`eval.py`), `trainer/`, `WebSocketTrainerBridge.js`, Checkpoints in `data/training/` (~4 MB je), `data/bot_validation_report.json`, Doku `docs/bot-training/Bot_Trainingsplan.md` (PPO-Stand 2026-05-08; produktive PPO-Umschaltung bis BT95 gesperrt).
- Training braucht einen deterministischen Spielkern — den gibt es im neuen Repo frühestens nach Phase 3.
- Die Spiel-tauglichen Bot-Gegner (`BotTuningConfig.js`) sind davon getrennt und gehören zum Content (E8), nicht zum Trainings-Stack.

**Optionen:**
1. **Einfrieren + sichern:** `python/`, Checkpoints und Trainingsplan-Doku als Kopie bewahren, nicht migrieren; Reaktivierung als eigene Phase 8 nach stabilem Kern.
2. **Mitnehmen:** Phase 8 fest einplanen.
3. **Aufgeben:** Stack fallen lassen, nur die gespielten Bot-Configs behalten.

**Empfehlung:** Option 1 (einfrieren + sichern). Die spielbaren Bot-Configs kommen unabhängig davon über E8 mit.

**Entscheidung:** ____ (Datum)

---

## E14 — Altrepo-Schicksal und offenes Erbe

**Frage:** Ab wann wird das Altrepo read-only, was passiert mit dem Archiv, und wie wird das offene Sach-Erbe disponiert?

**Fakten:**
- Ballast: `archive/` 227 MB + `docs/archive/` 161 MB ≈ **390 MB**. Plus tmp-Logs, `dist/`, `test-results/`, `videos/`.
- Offenes Sach-Erbe aus dem Altrepo:
  - **P21 / V146** — Electron-Security-Doppel-Major, Wiedervorlage **2026-07-11**. Wird vom frischen Electron-Setup in Phase 6 erledigt — **verfällt aber stillschweigend**, falls das neue Repo Phase 6 nicht vorher erreicht. Bewusst terminieren.
  - **V106** (kuratierte GLB-Map-Varianz) und **V113** (Hangar-Shell + Rules Panel) — offene Produktblöcke, Kandidaten für Post-1.0.
  - Findings **P14/P45–P48** (aus `docs/prozess/Open_Findings.md` des Altrepos: P14 = Listener-Duplikation hinter P45, P47 = Boundary-Drift, P48 = Recorder-Drift) — P45/P48 sind als Neuschreibungen schon in Phase 4/5 verplant; P14/P47 mitdenken.
- Git-Historie des Altrepos (1666 Commits) wird nicht migriert; das Altrepo bleibt als Nachschlagewerk inkl. Historie erhalten.

**Optionen / Teilentscheidungen:**
1. Altrepo **ab Migrationsstart read-only** (reine Verhaltensreferenz) vs. parallel weiterpflegen.
2. Archiv (~390 MB): **löschen** / **extern auslagern** / **behalten**.
3. P21: **bewusster Termin** (z. B. „spätestens 2026-07-11 muss Electron-Schale stehen oder Altrepo bekommt den Security-Bump") vs. fallen lassen.
4. V106/V113: als Post-1.0-Feature-Kandidaten notieren vs. verwerfen.

**Empfehlung:** Read-only ab Migrationsstart; Archiv extern auslagern oder löschen (nichts davon ist Spielcode); P21 terminieren statt verfallen lassen; V106/V113 als Post-1.0-Kandidaten parken.

**Entscheidung:** ____ (Datum; vier Teilentscheidungen)

---

## E15 — Release-Definition (1.0)

**Frage:** Was genau ist Version 1.0?

**Fakten:**
- Targets, Verteilweg und Muss-Features hängen an E1 (Start-/End-Targets), E8 (Content) und E9 (MP).
- Verteilwege je Target: Browser = Deploy (Static Hosting), Desktop = Installer (electron-builder), Android = APK.
- Qualitätsmaßstab im neuen Repo ist dein Playtest (Teil 3: du reviewst Verhalten), gestützt auf grüne migrierte Tests + CI.

**Zu entscheiden:**
- Welche Targets müssen für 1.0 fertig sein, welche dürfen 1.x sein?
- Welcher Content (E8-Auswahl) ist Muss?
- Welcher Verteilweg gilt als „released"?
- Welches Qualitätskriterium schließt 1.0 ab (z. B. bestandener Playtest aller Muss-Inhalte + CI grün)?

**Empfehlung (Vorschlag):** 1.0 = Start-Target (E1) stabil + von dir bestandener Playtest aller E8-Muss-Inhalte + CI grün; weitere Targets als 1.x. Final vor Phase 9 schärfen.

**Entscheidung:** ____ (Datum; Targets + Content + Verteilweg + Abschlusskriterium)
