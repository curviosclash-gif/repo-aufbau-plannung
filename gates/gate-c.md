# Gate C — vor Modi & Content (E8)

Fällig: Phase 3, bevor Hunt/Arcade und Content gezogen werden. Faktenbasis: [`../befund.md`](../befund.md).

---

## E8 — Übernahme-Umfang: Modi, Maps, Fahrzeuge, Items, Assets

**Frage:** Welche Modi, Maps, Fahrzeuge, Items und Assets kommen ins neue Repo — vollständig oder kuratiert?

**Fakten:**
- Modi: Classic (Phase 2), Hunt, Arcade inkl. Ghost-Selbstduell. `ArcadeRunRuntime.js` = 1492 Zeilen, beim Umzug in 2–3 Module zu schneiden (einziger geplanter Zuschnitt).
- Content: 15 JS-authored Map-Presets (davon 6 shipping-fähige Parcours, `parcours_pack_v130.js`), `data/maps/`, `data/vehicles/`, Blueprint-/Tier-Regeln (V76, Tests grün).
- Assets: `assets/` = **252 MB** (120 `.glb`, 43 `.obj`, 9 `.blend`, 9 `.fbx`). Blind alles zu kopieren zieht den Ballast wieder ein.
- Blaupause für selektive Übernahme: `scripts/export-game-only-repo.mjs` definiert bereits den minimalen Spielumfang.

**Optionen:**
1. **Vollständig:** alle 3 Modi, 6 Parcours, 15 Presets, gesamtes referenziertes Asset-Set. Maximaler Umfang, maximaler Migrationsaufwand.
2. **Kuratiert:** eine Auswahl, die du als 1.0-Inhalt festlegst (z. B. Classic + Arcade, 3–4 beste Maps); Rest als Post-1.0-Kandidat.
3. **Hybrid:** alle Modi behalten, aber Maps/Fahrzeuge kuratieren.

**Empfehlung:** keine — das ist deine Produktentscheidung. Regelvorschlag unabhängig vom Umfang: **Assets nur selektiv per Skript** übernehmen (nur tatsächlich referenzierte Dateien), nie pauschal das 252-MB-Verzeichnis.

**Entscheidung:** ____ (Datum; Modi + Maps + Fahrzeuge/Items + Asset-Regel)
