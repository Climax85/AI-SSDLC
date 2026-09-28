---
name: secure-feature
description: "Secure-SDLC orchestrator: drives a feature from intake to the merge into the application docs through all workflow phases. Use when a feature should be implemented securely, the intake is a ticket reference, or a running workflow should be continued from STATUS.md."
---

# Secure Feature — Orchestrator

You drive a workflow of six **phases**: you delegate each phase to the skill built for it and carry the state in `STATUS.md` — the specialist skills produce the artifacts.

Ground rules:

- Paths exclusively from the STATUS.md "Pfade" section.
- You read and write only the current phase's artifacts, strictly in template format.
- References exclusively as artifact IDs (`REQ-nn` schema).
- Stop rules in the templates are **hard stops**: halt the workflow, leave the decision to the human, record it as an open decision in STATUS.md.

## 1. Intake

- **Free text:** the description is the intake; feature name = short slug.
- **Ticket reference:** read the content via the configured tracker adapter (reference implementation: local markdown, see `docs/agents/issue-tracker.md`); ticket number = feature name. Deviant ticket structure → error paths.

## 2. Setup

1. Create `.scratch/<feature>/` and copy the templates from `templates/` into it.
2. **Materialize paths:** enter the resolved absolute paths of all templates and artifacts into the STATUS.md "Pfade" section — you are the only place that resolves paths.
3. Resolve the application-docs path — an `AGENTS.md` config value overrides the default `docs/anwendungsdokumentation.md`; materialize the result in the STATUS.md "Pfade" section. If the file is missing → **stop** with a pointer to `init-app-docs`. A file that exists but still carries template placeholders counts as missing.
4. Pick the profile (rule below); initial STATUS.md: phase `Spezifizieren`, all artifacts `offen`.

## 3. Profile selection rule

- **small:** bugfix, small extension, tight free-text description without new interfaces or data flows.
- **large:** new feature with interfaces/data, new application, unclear scope.
- Borderline cases → large. If the HITL budget (30–60 minutes) exposes a wrongly small profile → re-profile as large, don't trim.
- Execution: `small` runs grilling (§4); `large` runs wayfinder planning plus the same artifact set (§4).

## 4. Phases

Order: `Spezifizieren → Threat-Model → Testfälle → Umsetzung → Security-Review → Gemergt`. Every phase ends with the STATUS.md update as its last DoD step.

### Spezifizieren

Branch by profile; the artifact set is the destination either way.

**Profile `klein`:** Call the Skill tool twice, for "grilling" and "domain-modeling" (CONTEXT.md) and in parallel fill the artifacts `abuse-cases.md`, `anforderungen.md`, `akzeptanzkriterien.md`. Grilling in rounds: number the **frontier**, give a recommended answer with every question, then wait for the human.

**Profile `groß`:** Plan with `wayfinder` first — call it with the feature as the loose idea and the artifact set (incl. `use-cases.md`, ADRs) as the destination. The map is a planning aid, **not** a replacement for the artifacts. The **security frontier is mandatory**: the map must ticket, graduate from fog, or resolve at least — abuse cases, assets/trust boundaries, security requirements (REQ security flag), security-relevant acceptance criteria. You propose, the human confirms (per template rules; nothing is asked empty). Work the map one ticket per session; the phase label stays `Spezifizieren`, record the map reference and open tickets in STATUS.md "Historie". When the way is clear, fill the artifacts from the map, then call "grilling" and "domain-modeling" (CONTEXT.md; ADRs) for what the map left open. Reconcile per `docs/agents/artefakt-erweiterung-to-spec-to-tickets.md` §3 step 0 (first writer) — never duplicate existing rows. `wayfinder` not installed → fall back to breadth-first grilling rounds covering the same mandatory security frontier.

DoD: all artifacts of the phase `fertig`; each artifact's coverage rules from its template satisfied; no open decisions left; (`groß`) map resolved or its remaining tickets linked in STATUS.md.

### Feature-Epic (nach Spezifizieren, vor Threat-Model)

Create the **feature epic** in the configured tracker (see `docs/agents/feature-operations.md`), unless the intake was a ticket reference — then the referenced ticket *is* the epic and you update it in place. Epic body, condensed, German:

- `## Ziel` — the intake in 2–3 sentences
- `## Ergebnisse Spezifizieren` — abuse cases (condensed), REQ/AK core (counts + the security-flagged ones by ID), open decisions
- `## Verweise` — artifact dir (`.scratch/<feature>/`, flüchtig), wayfinder map link (`groß`), app-docs path

Later phases append condensed results (Threat-Model core with high risks, Security-Review findings); the final condensed write-back is `Gemergt`'s `ticket-update.md`. Materialize the epic reference in STATUS.md "Historie".

DoD: epic exists in the tracker and is referenced from STATUS.md; intake-by-reference repos reuse the referenced ticket.

### Threat-Model

Call the `threat-model` skill; it reads its own inputs from STATUS.md.

DoD: `threat-model.md` `fertig`; the "Neue Schnittstellen/Datenflüsse" and "Betriebsrelevante Festlegungen" sections confirmed by the human.

### Testfälle

Create `test-cases.md` with the developer; the coverage rules of its template apply unabridged.

DoD: `test-cases.md` `fertig`; the "Abdeckt AC-nn" mapping column complete. **Last DoD step: stop the session** — the workflow is interrupted after Testfälle and continues in a fresh session (§5).

### Umsetzung

Four sub-steps, **each in its own session**:

1. **Spec session (this skill):** Call the Skill tool with "to-spec" (artifact output into the feature directory), present the proposed test seams to the developer for confirmation, update STATUS.md (phase stays `Umsetzung`, spec referenced in "Historie") — and **stop the session**. Never implement in this session.
2. **Tickets session (this skill):** Call the Skill tool with "to-tickets" (quiz the developer, then publish via the **configured tracker** as children of the feature epic — see `docs/agents/feature-operations.md`; local tracker: into the feature's `issues/`), update STATUS.md (tickets referenced in "Historie") — and **stop the session**. Never implement in this session.
3. **Ticket sessions (one fresh session per ticket):** each runs the `implement-ticket` skill with the ticket number (or lets it pick the frontier). It claims the ticket, implements test-first, runs the full suite, **commits exactly once**, resolves the ticket.
4. **Re-entered planning session (this skill):** query the tracker's feature tickets (adapter frontier query; STATUS.md paths for the local tracker). Unresolved tickets → report the frontier (open, unblocked, unclaimed) and stop. All tickets `resolved` → run the full test suite once; green → DoD satisfied.

DoD: all tickets `resolved`; test cases green.

### Security-Review

`code-review`, then `security-review`. Both produce findings (security axis: `SF-nn`; standards/spec axes: reported in the session).

**Findings loop:** present every finding to the human with a fix-or-accept recommendation. Then:

- **Fix:** publish a follow-up ticket per coherent fix into the feature's `issues/` (same structure as Umsetzung tickets, status `ready-for-agent`) and implement it in ticket sessions (`implement-ticket`, Umsetzung Sub-Step-3 mechanics — fresh session per ticket, one commit each).
- **Accept:** record as `akzeptiert (<Begründung, durch wen>)` in `findings.md` and STATUS.md.
- **Artifact-level amendments** (e.g. an `IF-nn` row, an AK Lesart) are made directly in the artifact, not as tickets.

While the loop runs, the phase label stays `Security-Review`. After the last fix ticket: re-run the affected review walks against the new diff and the full suite once; then close the phase.

DoD: every finding `behoben` or `akzeptiert` with reasoning and risk rating in STATUS.md; for fixes: full suite green and the re-run walks clean.

### Gemergt

Call `merge-app-docs`. Then insert `ticket-update.md` into the ticket, condensed (free-text intake without a tracker ticket: skip, note in STATUS.md). Then commit the merged application docs in one commit (`docs: Anwendungsdokumentation vX.Y — <feature> Artefakte gemergt`) and delete the feature directory.

DoD: application docs merged, committed and ticket updated, phase = `Gemergt`; the feature directory deleted.

## 5. Continuing with empty context

First step of every session: read `STATUS.md`. Then:

1. "Offene Entscheidungen" not empty → clarify these with the human first.
2. `Artefakt-Status` of the current phase: catch up open artifacts in template order; continue an `entwurf` where it stands.
3. All artifacts of the phase `fertig` → check DoD, advance the phase, begin the next phase. Phase `Umsetzung` is the exception — it has no phase artifacts; derive the sub-step from the "Historie": no spec published → **Spec session** (sub-step 1); spec published but no tickets in `issues/` → **Tickets session** (sub-step 2); tickets published → report the frontier or, all resolved, run the suite (sub-step 4). Phase `Security-Review` with findings not yet dispositioned → derive the state from `findings.md` + `Historie`: open fix tickets → report the frontier; all dispositioned → DoD check and advance.

## 6. Error paths

- **Unclear grilling result:** re-ask the question as a numbered round with a recommended answer; if it stays unclear → open decision in STATUS.md, wait for the human.
- **Missing or contract-violating artifact:** repeat the producing phase; record the deviation in STATUS.md.
- **Deviant ticket structure:** treat the ticket body as feature free text, ticket number = feature name, record the deviation in STATUS.md.
- **STATUS.md inconsistent** (phase and artifact status contradict each other): present both values to the human with a question for correction.
