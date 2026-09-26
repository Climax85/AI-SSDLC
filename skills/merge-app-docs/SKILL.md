---
name: merge-app-docs
description: Merges a finished feature's security artifacts into the application documentation (docs/anwendungsdokumentation.md) — table-appends only, never rewriting existing sections. Use when secure-feature reaches the Gemergt phase, when a developer asks to merge feature artifacts into the app docs, or when review findings must land in the app docs' audit history.
---

# Merge App Docs

You merge one finished feature's security artifacts into the **application documentation**. Your method is **structured append**: rows go into tables, bullets go under free-text headings — nothing else. A weak model must be able to run this from an empty chat, so every leg is a mechanical mapping — no distilling, no rewording, no free-text editing of existing content.

Ground rules:

- Read exactly: `STATUS.md`, then the artifacts it marks `fertig`: `threat-model.md`, `anforderungen.md`, `findings.md`. Nothing else — except `docs/adr/` for the ADR leg.
- Resolve every path from `STATUS.md`, never yourself.
- Write exactly: the application docs (path from `STATUS.md`) and `STATUS.md`.
- German contract strings verbatim (section headings, status values) — never translated.
- New rows get the table's next free app-level ID; the feature-local ID rides in a Bemerkung/Dokumentation cell as `<Ticket-Referenz> (<TM-nn / REQ-nn / IF-nn / SF-nn>)`.
- Legs whose artifact is `offen` in `STATUS.md` are skipped and named in the report — silently merging half a feature is worse than merging none.

## 1. Entry check

1. Read `STATUS.md`. Application-docs path missing or file absent → **stop**: report that the application docs must be created first — that is `init-app-docs`' job, not yours.
2. Verify every target section exists with its template heading (§3.1, §3.2, §3.3, §4.3, §5.1, §5.2, §12, §13). A missing heading means the doc deviates from the contract → **stop**; report — never guess a section's place.
3. Read only the artifacts marked `fertig`. An artifact marked `fertig` whose file is missing → **stop**; report.

## 2. Merge legs

Append legs in this order. Duplicate test per leg, then conflict rule below.

**§3.1 Sicherheitsanforderungen** ← `anforderungen.md`, rows with `Security: ja` (fachlich und funktional).
Map: ID = next free `SA-nn`; Kategorie = best fit of `Vertraulichkeit / Integrität / Verfügbarkeit / Audit-Nachweis`; Anforderung = verbatim, one sentence; Begründung / Bezug = `<Ticket> (<REQ-nn>)`; Status = `umgesetzt`.
Duplicate: the Anforderung already exists in §3.1 → skip, note in report.

**§3.2 Bedrohungsmodell** ← STRIDE-Tabelle in `threat-model.md`.
Copy every row verbatim with the next free app-level `TM-nn` (Status rides along, e.g. `akzeptiert`).
Duplicate key: Bedrohung + Asset. Matching row exists → merge into it: Gegenmaßnahmen combined (existing first, feature's appended), Risiko = the higher of both, Status = `offen` unless every combined Gegenmaßnahme is `umgesetzt`. Every merged row goes in the report.

**§3.3 Akzeptierte Risiken** ← `findings.md` (nur Zeilen mit Status `akzeptiert (...)`) + Abschnitt „Annahmen und Abgrenzung" in `threat-model.md` (nur groß-Profil).
Per accepted finding one bullet: `- **<ID> (<Risiko>, akzeptiert):** <Begründung = Status-Text ohne das führende Wort `akzeptiert`> (<Referenz>)`. Per „Annahmen und Abgrenzung"-Bullet one bullet, text verbatim. The first bullet replaces the subsection's init status note (placeholder rule as in the free-text legs). Duplicate key: bullet text. Match → skip, note in report.

**§4.3 Architektur-Entscheidungen (ADRs)** ← new files in `docs/adr/` not yet listed in §4.3.
ID = next free `ADR-nn`; Entscheidung = the ADR's title; Kontext = the ADR's paragraph verbatim; Konsequenz = a `Consequences` section if the ADR has one, else empty; Datum = the ADR's `Datum: YYYY-MM-DD` line (new format) or the file's git-addition date (`git log --follow --format=%ad --date=short -1 <file>`) for legacy ADRs without that line. No `docs/adr/` or no new ADRs → skip leg, note in report. Empty Konsequenz cells are named in the report, never invented.

**§5.1 Externe Schnittstellen / §5.2 Interne Schnittstellen** ← „Neue Schnittstellen/Datenflüsse" in `threat-model.md`, only rows with Status `bestätigt` or `korrigiert`.
Route by Richtung: `intern` → §5.2, everything else → §5.1. ID = next free app-level `IF-nn`.
Map columns that exist by name (`Authentisierung`, `Daten`); Schnittstelle/Datenfluss → `Name` (§5.1); Richtung → `Datenfluss` (§5.1) or `Bemerkung` (§5.2); append `<Ticket> (<IF-nn>)` to `Dokumentation`/`Bemerkung`. Unmatched cells stay empty — never invent Protokoll or Verantwortlich values.
Duplicate key: §5.1 `Name`, §5.2 `Schnittstelle` text. Match → skip, note in report.

**§13 Audit- und Testhistorie** ← `findings.md` (the full table — the source of truth; `ticket-update.md` is only its condensed projection).
§13.1 one row: Datum = today; Art = `Security-Review`; Durchgeführt von = developer from §2.2; Ergebnis = `Keine Findings` or `<n> Findings (<m> hoch)`; Bemerkung = `<Ticket>`. Then sync §1 Metadaten: set `Letzter Sicherheitsreview` to this row's Datum (replacing `noch keiner durchgeführt`). Accepted findings that carry an anchoring obligation (e.g. a manual test to be verifiable in the app docs) ride in the Bemerkung.
§13.2 per finding: ID = the finding's own ID (`SF-nn`/`RV-nn`); Finding = `<Referenz> — <Fundstelle>`; Risiko verbatim; Maßnahme = Empfehlung; Frist = empty; Status verbatim from `findings.md`, **except** `Fix → Ticket n` which is written as `behoben (Ticket n)` — Gemergt guarantees every fix ticket resolved.

**§4.1 / §4.2 / §6 / §8 / §10 / §14 / Anhänge A–C (freie Abschnitte)** ← „Betriebsrelevante Festlegungen" in `threat-model.md`, only rows with Status `bestätigt` or `korrigiert`.
Map: append one bullet per row under the named Zielabschnitt heading (subsection names allowed, e.g. `§6.1`), Festlegung verbatim — no rewording. If the target is a checklist item in Anhang A–C, check the matching box instead of adding a bullet (kein Bullet nötig, wenn die Checkbox den Satz wörtlich trägt). The first bullet under a subsection **replaces that subsection's init status note** (a placeholder, not existing content). Unmatched Zielabschnitt → report, don't guess a section.
Duplicate key: Zielabschnitt + Festlegung text. Match → skip, note in report.

## 3. Conflict rule

An existing row and a feature row that **cannot both be true** (same key, contradictory content) is a **conflict** — never resolve it yourself, never overwrite.

- Merge everything non-conflicting first.
- Then halt and present each conflict with both variants side by side; the developer picks. Wait for the answer, apply it, continue.
- A developer decision never alters rows outside the conflicting key.

## 4. Changelog (§12)

Always one row: Version bumped, Datum = today, Änderung = `<Ticket>: <one line — what was merged>`, Autor = developer from §2.2, Genehmigt durch = developer, Sicherheitsrelevant = `ja` iff any of §3/§5/§6/§8/§10 changed or an appended finding has Risiko `hoch` or `mittel` — else `nein`.
Version: security-relevant → bump the minor component (`1.2` → `1.3`); otherwise bump the patch (`1.2` → `1.2.1`, first patch on an `x.y` version = `x.y.1`).

## 5. Report and STATUS.md

Report one summary to the developer: rows appended per section, rows merged, rows skipped as duplicates, conflicts and their resolutions, the changelog row. Then update `STATUS.md` as your last action: Phase = `Gemergt`, `Letzte Aktualisierung` = `<date, merge-app-docs>`. The feature directory, the ticket update — not yours; `secure-feature` owns both.

## 6. DoD

- [ ] Every `fertig` artifact merged or explicitly skipped-and-named; no row silently dropped
- [ ] §3.2: every feature TM row appended or merged under its duplicate key; §3.1, §3.3, §4.3, §5.1/§5.2, §13: every qualifying row/bullet appended; §4.1/§4.2/§6/§8/§10/§14/Anhänge: every bestätigte Festlegung appended as a bullet or checkbox
- [ ] §13 carries the full findings table (Fundstelle, Empfehlung) — not the ticket-condensed version
- [ ] Every conflict presented with both variants and resolved by the developer — none merged silently
- [ ] Changelog row with ticket reference; version bumped exactly under the §4 rule
- [ ] No existing content reworded; every edit is a table row or a verbatim bullet replacing an init status note
- [ ] `STATUS.md`: Phase = `Gemergt`, `Letzte Aktualisierung` set

## 7. Error paths

- **Application docs missing** → stop; point at `init-app-docs`.
- **Target section heading deviates from the template** → stop; report the deviation — the doc's structure is the contract, not your call.
- **Artifact `fertig` in `STATUS.md` but file missing** → stop; report.
- **Merge leg would invent content** (e.g. Protokoll, Verantwortlich, Frist) → leave the cell empty and name it in the report; a blank cell the developer fills beats a fabricated one.
