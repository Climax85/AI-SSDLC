---
name: security-review
description: Security-axis review that runs after code-review — walks every threat-model mitigation, abuse-case test coverage, and new interface against the diff, then writes findings. Use in the Security-Review phase (delegated by secure-feature) or when an implementation must be checked against the feature's threat model.
---

# Security Review

You are the **Security axis** of the review, run after Pocock's `code-review` (Standards and Spec axes). You walk the security artifacts against the diff and write exactly one artifact: `findings.md`.

Ground rules:

- Paths exclusively from the STATUS.md "Pfade" section.
- Read only `threat-model.md`, `abuse-cases.md`, `akzeptanzkriterien.md`, `test-cases.md`; write only `findings.md` plus the Findings table in `ticket-update.md` — plus the TM-Status column in `threat-model.md` (§3, TM status closure) and the STATUS.md update as your last DoD step.
- References exclusively as artifact IDs (`TM-nn`, `AC-nn`, `IF-nn`, `AK-nn`).
- Stop rules are **hard stops**: halt, record in STATUS.md "Offene Entscheidungen", wait for the developer.

## 1. Fixed point

The same fixed point `code-review` used (commit, branch, tag, or merge-base); if none was supplied, ask. Capture `git diff <fixed-point>...HEAD` and `git log <fixed-point>..HEAD --oneline`. Confirm the ref resolves and the diff is non-empty — fail here, not inside the walks.

## 2. Inputs

Every input must be `fertig` per STATUS.md "Artefakt-Status"; otherwise stop and report the artifact — the producing phase repeats it. The review derives wholly from artifacts + diff; no chat context needed.

**Axis boundary:** Standards and spec conformance belong to `code-review` and are out of scope here. Every finding carries a `TM-nn`/`AC-nn`/`IF-nn`/`AK-nn` reference; security-relevant behavior in the diff with no covering artifact becomes a completeness finding against the threat model (§3), not licence for a free-form review.

## 3. Walks

Three walks plus a sweep, in order. Finish every walk before writing findings — the walks are the legwork; the findings table is just their record.

### Mitigation walk

One pass per `TM-nn` row: locate the Gegenmaßnahme in the diff or the code it touches and verify it enforces — mechanism present, placed at the boundary (validation before the data is used), no bypass path opened by the diff. Skip rows the developer accepted (`Status: akzeptiert`): acceptance is their recorded decision; you verify mitigations, not acceptances. Missing, misplaced, or bypassable → finding, severity from the row's Risiko.

### Coverage walk

One pass per `AC-nn`: the mapped `TC-nn` exists, asserts the protective behavior — attacker-shaped input, refused outcome, not just the happy path — and passes. Every security criterion (`Art: security`) gets the same pass against the diff; AK-to-TC mapping runs via the shared `REQ-nn` reference (an AK is covered iff a TC mapped to its `REQ-nn` asserts the protective behavior — `test-cases.md` carries no AK column by contract). Missing, toothless, or red coverage → finding.

### Interface walk

One pass per `IF-nn` row: the declared Authentisierung exists at that boundary and the Daten are validated as described. Declared-but-absent control → finding. An interface the diff introduces without an `IF-nn` row → finding.

### New surface

Security-relevant behavior in the diff that no `TM-nn`/`AC-nn` covers → completeness finding referencing the nearest `TM-nn` or the uncovered `IF-nn`. The developer amends the threat model or accepts the gap — you don't write `threat-model.md` beyond the TM status closure below.

### TM status closure

When you verify a `TM-nn` row's mitigation in the mitigation walk and no `offen` finding references that row, set the row's Status to `umgesetzt` — the only edit you make to `threat-model.md`. Rows whose residual gap was accepted (`akzeptiert (...)`) keep that status. On a findings-loop re-run, re-check the rows the fix tickets touched and flip them the same way. Status flips are recorded in the STATUS.md update.

## 4. Findings

One `SF-nn` per finding, in walk order: `SF-nn | Referenz | Fundstelle (file:line) | Risiko | Empfehlung (one line) | Status: offen`.

Write the findings twice — full once, condensed once:

1. `findings.md` — the full table; source of truth for `merge-app-docs` (→ Anwendungsdoku §13).
2. The Findings table in `ticket-update.md` — same rows, Empfehlung trimmed to one line; the ticket rule applies (Kerntabellen, keine Volltexte).

Then report the findings in the session, ordered by Risiko — the tables outlive the chat.

## 5. Stop-Stelle

Every finding with `Risiko: hoch` is a **hard stop**: list the `SF-nn` in STATUS.md "Offene Entscheidungen" and wait — the developer fixes or accepts with reasoning (Status `akzeptiert (<Begründung, durch wen>)`), the same shape as the threat model's stop rule. `mittel`/`niedrig` findings are reported, not blocking.

## 6. DoD

- [ ] Fixed point resolves; diff non-empty
- [ ] Mitigation walk: every `TM-nn` row verified or finding written; verified rows flipped to `umgesetzt` (accepted rows untouched)
- [ ] Coverage walk: every `AC-nn` and every security `AK-nn` verified or finding written
- [ ] Interface walk: every `IF-nn` verified; every new interface in the diff has an `IF-nn` row
- [ ] New surface swept; every finding carries an artifact reference
- [ ] `findings.md` written (explicit "keine Findings" if empty); Findings table in `ticket-update.md` updated
- [ ] Every `hoch` finding in STATUS.md "Offene Entscheidungen" or the Stop-Stelle held
- [ ] STATUS.md updated as last step: Findings `fertig`, "Letzte Aktualisierung" = `security-review`

## 7. Error paths

- **Missing or contract-violating input artifact:** stop; report the artifact — the producing phase repeats it. Never improvise missing inputs.
- **Tests not green:** stop; coverage can't be verified against red tests — Umsetzung repeats.
- **Mitigation lives entirely outside the diff** (pre-existing code): verify it in place and name the existing file as Fundstelle; only absent-or-bypassable-anywhere is a finding.
