# Gate E — Gedächtnis-Apparat (E11–E12)

**Default beider Entscheidungen: nein.** Diese Gates werden nur geöffnet, wenn ein konkreter, wiederholter
Schmerz auftritt (L1/L3) **und** du den Bau explizit beauftragst. Faktenbasis: [`../befund.md`](../befund.md).

> Warum so streng: Wissensgraph (4113 Knoten), Graph-RAG, Plan-Autopilot und Lock-Registry waren im Altrepo
> technisch funktionsfähig — und zusammen die dokumentierte Hauptlast, die das Projekt erstickt hat. Genau diese
> Bausteine standen in den verworfenen Quellplänen wieder als reguläre Phasen. Sie sind hier bewusst zu
> opt-in-Entscheidungen mit Default „nein" zurückgestuft.

---

## E11 — Wissensgraph?

**Frage:** Soll ein Wissensgraph (Module/Abhängigkeiten/Tests) gebaut werden?

**Trigger, der dieses Gate überhaupt öffnet:** Du erwischst dich **wiederholt** dabei, Modul-Abhängigkeiten
oder Impact von Änderungen nicht mehr per direkter Suche (grep/glob) zu überblicken — und die Landkarte
reicht nachweislich nicht.

**Fakten:**
- Agentische Suche (grep/glob + Reasoning) schlägt einen Graph für Code-Navigation in den meisten Fällen; ein Graph driftet bei jedem Edit und braucht ein Generierungs-/Abgleich-Gate (= neue Wahrheitsquelle, verstößt gegen L2).
- Falls je gebaut: nie Source of Truth, nie Abschlussbeweis, nie Freigabeersatz — nur Radar.

**Empfehlung:** Default **nein**. Bei Trigger: zuerst die Landkarte verbessern, erst dann Graph erwägen.

**Entscheidung:** ____ (Default: nein — nur ändern bei dokumentiertem Trigger + Auftrag)

---

## E12 — RAG?

**Frage:** Soll ein RAG-/Retrieval-System gebaut werden?

**Trigger, der dieses Gate überhaupt öffnet:** Es ist eine **große Prosa-Wissensbasis** entstanden (viel
Doku/Design-Text) und die direkte Suche darin reicht nachweislich nicht mehr.

**Fakten:**
- **Kein Code-RAG.** Embeddings über Code veralten bei jedem Edit; agentische Suche ist überlegen. RAG ist allenfalls für große Prosa-Wissensbasen sinnvoll.
- Falls je gebaut: liest nur, ändert nie Code; keine Quelle → keine Antwort; bei Widerspruch gewinnt die Datei; Memory-Schreibvorgänge nur als Vorschlag mit menschlichem Review.

**Empfehlung:** Default **nein**, insbesondere kein RAG über Code.

**Entscheidung:** ____ (Default: nein — nur ändern bei Prosa-Trigger + Auftrag)
