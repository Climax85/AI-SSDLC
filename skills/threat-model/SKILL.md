---
name: threat-model
description: "Feature-level STRIDE threat model: derives threats from the requirement artifacts, maps them to abuse cases, proposes new interfaces/data flows for the developer to confirm, and hard-stops on unmitigated high risks. Use when a feature reaches the Threat-Model phase (delegated by secure-feature) or when a threat model must be rebuilt from the artifacts of the Spezifizieren phase."
---

# Threat Model

You produce the feature threat model: the STRIDE table plus the proposed interfaces delta and the ops-relevant decisions (target sections in the application docs). You read your input artifacts and write exactly one: `threat-model.md`.

Ground rules:

- Paths exclusively from the STATUS.md "Pfade" section.
- Read only `anforderungen.md`, `abuse-cases.md` and `use-cases.md` (large profile only); write only `threat-model.md` — plus the STATUS.md update as your last DoD step.
- References exclusively as artifact IDs (`REQ-nn`, `AC-nn`, `IF-nn`).
- Stop rules are **hard stops**: halt, record the decision in STATUS.md "Offene Entscheidungen", wait for the developer.

## 1. Inputs

Read from the STATUS.md paths. Every input must be `fertig`; the profile comes from the STATUS.md "Steuerung" table. Each requirement with `Security: ja` and each abuse case is a threat seed.

## 2. STRIDE

Walk all six STRIDE categories. One row per realistic threat, `TM-01` upward in order, derived from the requirements and abuse cases. Every row complete: threat in one sentence, asset, impact in one sentence, risk, mitigation or `keine`, the covering `AC-nn` or `-`, status `offen`.

Profile limits (hard):

- **small:** 3–8 rows. Walk STRIDE completely, enter only realistic threats; risk `niedrig` → one-line row, status `akzeptiert`. Omit the "Annahmen und Abgrenzung" section.
- **large:** every realistic threat its own row; fill "Annahmen und Abgrenzung".

A threat without a covering abuse case gets `-` in the AC-nn column and an entry in STATUS.md "Offene Entscheidungen" — the developer decides whether an abuse case gets added; you don't write `abuse-cases.md`.

## 3. Neue Schnittstellen/Datenflüsse

Propose every interface or data flow the feature introduces, one `IF-nn` row each: direction (`eingehend / ausgehend / intern`), data, authentication method or `nein`, status `vorgeschlagen`. No limit in either profile — this section feeds the application docs. Present the table to the developer as a numbered list; the developer sets each row to `bestätigt` or `korrigiert`. If the feature introduces no new interfaces, write that explicitly — never leave the section empty.

## 4. Betriebsrelevante Festlegungen

Propose every ops-relevant decision confirmed in grilling with a target section in the application docs, one `OP-nn` row each: `Zielabschnitt` (section or subsection heading, e.g. `§6`, `§6.1`, `§14.3`), Festlegung in one sentence, status `vorgeschlagen`. Targets: §4.1, §4.2 (Technologie-Stack), §6, §8 (Kryptografie — verfahren, Bibliothek, Schlüssellänge, Schlüsselmanagement), §10, §14, optionally Anhang A–C. For features that fix the tech stack or use cryptography, proposing §4.2/§8 rows is mandatory — ask for the concrete facts (algorithm, library, key length) instead of leaving the app-doc cells empty. Present the table to the developer as a numbered list; the developer sets each row to `bestätigt` or `korrigiert`. If the feature confirms no ops decisions, write that explicitly — never leave the section empty. This section feeds `merge-app-docs`.

## 5. Stop-Stelle

`Risiko: hoch` with `Gegenmaßnahme: keine` → **stop**. Record the threat ID in STATUS.md "Offene Entscheidungen" and wait: the developer picks mitigation, risk acceptance with reasoning, or scope change.

## 6. DoD

- [ ] STRIDE walked across all six categories; `TM-nn` numbering sequential
- [ ] Every realistic threat from the inputs has a row; profile limits respected
- [ ] Every row has risk, mitigation or `keine`, `AC-nn` or `-`, status
- [ ] Every high-risk row carries a mitigation or stopped at the Stop-Stelle
- [ ] Interfaces section: every new interface proposed and confirmed or corrected by the developer
- [ ] Ops section: every grilling-confirmed ops decision proposed and confirmed or corrected by the developer
- [ ] STATUS.md updated as last step: Threat Model `fertig`, "Letzte Aktualisierung" = `threat-model`

## 6. Error paths

- **Missing or contract-violating input artifact:** stop; report the artifact to the developer — the Spezifizieren phase repeats it. Never improvise missing inputs.
- **More realistic threats than the small-profile limit:** re-profile the feature as large, don't drop threats.
- **Ambiguous STRIDE category:** assign the category matching the threat's primary mechanism and note it in "Annahmen und Abgrenzung" (large) or the row's threat sentence (small).
