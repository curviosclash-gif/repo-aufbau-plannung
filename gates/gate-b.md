# Gate B — vor der Kern-Migration (E7)

Fällig: Phase 2, bevor das erste Modul aus dem Altrepo gezogen wird. Faktenbasis: [`../befund.md`](../befund.md).

---

## E7 — Contracts 1:1 übernehmen oder vereinfachen?

**Frage:** Werden die Contracts aus `src/shared/contracts/` beim Umzug unverändert übernommen oder verschlankt?

**Fakten:**
- Die Contracts sind Kronjuwelen (Befund §2): `GameplayConfigContract.js`, `FightHangarBalanceContract.js`, Match-Lifecycle, GameState. V112 belegt 585 grüne Contract-Tests gegen sie.
- Sie sind der Determinismus-Anker für Phase 2: gleiche Contracts + gleiche Seeds → gleiche Ergebnisse wie im Altrepo.
- Ein Contract ist die Schnittstelle vieler Konsumenten; verschlankt man ihn vor dem Konsumenten, verliert man den Vergleichsmaßstab.

**Optionen:**
1. **1:1 übernehmen**, vereinfachen erst, wenn der jeweilige Konsument migriert ist und der Test grün bleibt.
2. **Beim Umzug verschlanken** — spart später einen Schritt, riskiert aber stille Verhaltensänderung genau an der Stelle, die als Referenz dienen soll.

**Empfehlung:** Option 1. Die Verschlankung wird ein eigener, späterer Slice mit eigenem Test — nicht Teil der Migration.

**Entscheidung:** ____ (Datum)
