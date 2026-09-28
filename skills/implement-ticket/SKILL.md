---
name: implement-ticket
description: Implement exactly one published ticket from a secure-feature workflow in a fresh session on its own feature branch — claim it, branch, implement test-first, run the full suite, commit, delegate the per-ticket review (review-ticket loop), open a pull request, resolve it. The ticket lives in the configured tracker (local tracker: `.scratch/<feature>/issues/`). Use when a ticket number should be worked off, when the frontier ticket should be auto-picked, or when per-ticket sessions run after `to-tickets`.
---

# Implement Ticket

You implement exactly one ticket in a fresh session. The ticket is information-rich by design. You claim, implement, commit, get the ticket reviewed, resolve — nothing else.

The tracker's feature operations (`docs/agents/feature-operations.md`) define the ticket's medium and how to read, claim, status, comment, and resolve it. Local-markdown tracker → one file under `.scratch/<feature>/issues/`; GitHub/GitLab → an issue. STATUS.md gives you the paths (local tracker) and the epic reference.

`resolved` means: implemented **and** reviewed clean. The review-ticket loop is part of this skill, not optional.

Ground rules:

- Read only: `STATUS.md` (paths + phase), the ticket (via the adapter), and only what the ticket itself references (spec, test-cases). Never "the repo" broadly.
- Write only: code/tests and the ticket (claim, status, comment — via the adapter).
- Stop rules are **hard stops**: halt and report, do not improvise.

## 1. Input

- Ticket number (`06`) or nothing → you pick the frontier ticket (adapter frontier query: open, unblocked, unclaimed; local tracker: lowest number).
- Feature reference: pass it as an argument (like the secure-feature call); local tracker: the feature directory or the ticket file path directly. Without a resolvable ticket, stop and ask.

## 2. Claim (before any code work)

1. Read `STATUS.md`: phase must be `Umsetzung` — otherwise stop.
2. Read the ticket via the adapter. Hard stops:
   - ticket `resolved` (closed) → already done, stop.
   - ticket claimed (an assignee set / `Status: claimed`) → another session owns it, stop.
   - a blocking ticket not yet `resolved` → stop and name the blockers.
3. Claim it (adapter: assign yourself / set `Status: claimed`) — the session's first write.
4. Create the ticket branch: `git switch -c feature/<feature-slug>/<NN>-<slug>` from the integration (default) branch. A branch for this ticket already exists (resumed session) → switch to it instead of creating. All commits of this ticket happen only on this branch.

## 3. Implement

- Follow the `tdd` skill: red → green, one slice at a time; tests only at the seams the ticket/spec names (Unit-Seam / E2E-Seam).
- Run type checks and single test files during the loop; the full test suite once before committing.
- Acceptance criteria from the ticket are your contract — check them off as you verify them.
- Enthält das Ticket einen `## Review-Runde n`-Kommentar mit offenen must-fix-Items (Rückgabe aus einer vorherigen Review-Session), arbeite diese zuerst ab — sie sind der Auftrag dieser Session.

- **Self-Check vor dem Commit** (nur wenn STATUS.md `Self-Check: an` ist; fehlende Zeile = `an`): Akzeptanzkriterien nochmals gegen den eigenen Diff lesen und den Diff gegen `CONVENTIONS.md` am Repo-Root prüfen (fehlt die Datei, Achse auslassen — `review-ticket` legt sie ggf. an). Ergebnis in einem Satz im Ticket-Kommentar. Nie blockierend. Bei `aus` oder knappem Kontextfenster: auslassen und im Kommentar vermerken.

## 4. Commit (harte Pflicht)

- Genau **ein Commit pro Runde** — die Initialrunde plus je einen Fixup-Commit pro Review-Runde.
- If the working directory is not a git repo → stop with the precise instruction to initialize git and create a baseline commit; do not commit yourself.
- Message: `<NN>: <ticket title>`; Fixups: `<NN>: Review-Fixups Runde <n>`; Body lists the acceptance criteria with `x`/`-`.

## 5. Review-Runde (harte Pflicht)

Nach dem Commit, vor dem Resolve. Der Scope ist der Ticket-Commit dieser Runde.

1. **Delegiere** den Review an den Subagent: `{ agent: "review-ticket", task: "Feature: <feature-dir>, Ticket: <NN>, Commit: <hash>" }`. Kein `subagent`-Tool verfügbar → arbeite `.pi/skills/review-ticket/SKILL.md` selbst exakt ab (Inline-Fallback, kostet Kontext) — der Loop gilt unverändert.
2. **Verdikt `clean`** → weiter mit §6 (Resolve).
3. **Verdikt `must-fix`** → behebe jedes Item (TDD: zeige mit einem Test, was das Item bricht, falls möglich), Fixup-Commit, dann **Re-Check**: erneuter Subagent-Aufruf mit derselben Runde (prüft nur die offenen Items).
4. Re-Check weiterhin `must-fix` → Ticket auf `ready-for-agent` mit Kommentar, Session stoppen und dem Entwickler melden. Keine dritte Runde, keine Eigenmächtigkeit.
5. **Verdikt `stop`** → Session stoppen, Ursache dem Entwickler melden; Ticket bleibt `claimed`.

Notes (`RV-nn`) stehen bereits in `findings.md` — im Ticket-Kommentar vermerken, kein weiterer Aufwand.

## 6. Resolve

- Voraussetzung: Review-Runde dieser Runde ist `clean`.
- Comment on the ticket (adapter): what was built, test counts, commit hash(s), Review-Ergebnis.
- Create the pull request into the integration branch — repo with remote: `gh pr create` / `glab mr create` (title `<NN>: <ticket title>`, body: what was built, acceptance criteria with `x`/`-`, test counts, commit hash); no remote (local tracker): skip, note `no PR (local)` in the ticket comment. **Merging is not this skill's job** — the merge happens in the re-entered planning session (secure-feature Umsetzung sub-step 4).
- Comment the PR/MR link on the ticket; then resolve it (adapter: close the issue / set `Status: resolved`).
- Report the new frontier (adapter frontier query) so the next session can start.

## 7. Error paths

- Ticket contract-violating (no "What to build", no "Blocked by") → stop, report the ticket.
- Full suite red after your work → either fix within the session, or set the ticket back to `ready-for-agent` with a comment describing the failure. **Never resolve a ticket on red.**
- Acceptance criterion not achievable as written → set `ready-for-agent` with a comment naming the criterion and the blocker; do not silently redefine scope.
- Subagent unerreichbar/stürzt ab → Inline-Fallback wie in §5 Schritt 1; schlägt auch der fehl, Ticket auf `ready-for-agent` mit Vermerk und dem Entwickler melden.
