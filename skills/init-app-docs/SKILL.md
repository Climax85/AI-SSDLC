---
name: init-app-docs
description: Initializes the application documentation for a repo that has none — gathers facts from the repo AFK, then has the developer confirm or correct one concrete proposal per still-empty mandatory section. Use when the application docs are missing or incomplete (secure-feature aborts with a pointer to this skill) or when a developer asks to create docs/anwendungsdokumentation.md.
---

# Init App Docs

You create the **application documentation** for a repository that lacks it: `docs/anwendungsdokumentation.md` plus a slim root `README.md`. Your input is the repository itself — you are the one skill that reads broadly. Your output is exactly two files, filled strictly after the templates you ship with in `assets/`.

Ground rules:

- Read broadly: README, CONTEXT.md, ADRs, stack manifests, code structure, existing `docs/`, CI/deploy config.
- Write exactly two files: the app docs at the configured path (default `docs/anwendungsdokumentation.md`; a differently named path in the repo's `AGENTS.md` overrides) and the root `README.md`. Nothing else — in particular no `STATUS.md`, no `.scratch/`: you precede the feature workflow.
- German contract strings verbatim (section headings, status values) — never translated.
- The template is the contract: copy it from `assets/` into place and fill it there; never restructure it.
- Stop rules are **hard stops**: halt, report, wait for the developer.

## 1. Entry check

- Docs file missing → proceed.
- Docs file exists with placeholders in mandatory sections → proceed; your draft fills what the repo yields, grilling covers only the rest.
- Docs file exists, all mandatory sections filled → stop: maintaining a complete doc is `merge-app-docs`' job, not yours. Say so and end.

## 2. Fact collection (AFK)

Fill the draft from the repo **before asking anything**. Sources in order: README, CONTEXT.md, `docs/adr/`, stack manifests (`package.json`, `*.csproj`, `requirements.txt`, `*.pro`, `go.mod`, …), directory layout and entry points, CI/deploy config, existing `docs/`.

Every draft value carries its source as a trailing HTML comment. A value without a nameable source is a **proposal**, not a fact — it becomes a grilling item.

The repo typically yields: tech stack (§1 Technologiestack), logging and monitoring basics (§9), CI/deploy config (§11). Nicht draften trotz vorhandener Fakten: §4–§6, §10, §14 — dort trägt der Init nur Platzhalter ein (siehe unten). It typically does not yield: owner names, Schutzbedarf, Kritikalität, approval, dates — those become grilling targets.

Nicht draften, auch wenn das Repo sie hergibt: **§3.1 (Sicherheitsanforderungen), §3.2 (Bedrohungsmodell), §3.3 (akzeptierte Risiken)** — und ebenso alles Umsetzungs- und Feature-abhängige: **§4 (Architektur: Übersichtsdiagramm, Technologie-Stack, ADR-Tabelle), §5 (Schnittstellen/Datenflüsse), §6 (Authentisierung & Autorisierung), §10 (Betriebsumgebung & Hardening-Konfiguration), §14 (Betriebshandbuch/Runbook)**. Das Sicherheitsprofil und diese Abschnitte sind Gegenstand der Feature-Workflows (Threat Model inkl. grilling-bestätigter Festlegungen) und werden über `merge-app-docs` nachgetragen; vorhandene Abuse-Cases oder Security-REQs aus `.scratch/` sind kein Draft-Input. Der Init-Draft hält die Template-Headings und die Tabellenköpfe vor (sonst kann `merge-app-docs` nicht anhängen) und trägt statt Inhalt einen kurzen Statusvermerk.

## 3. Grilling (HITL) — proposals, not questions

Call the Skill tool twice, for "grilling" and "domain-modeling". Then one numbered round: for every mandatory section the draft could not fill, present **one concrete proposal** built from the strongest nearby fact — state the fact and its source — and ask for confirm/correct. Never ask an open question the repo could answer; the human's job is judging your proposal, not doing your fact collection. Grill ausschließlich die umsetzungsunabhängigen Abschnitte (Dokumentenheader, §1, §2, §7, §9, §11–§13) — die Platzhalter-Abschnitte §4–§6, §10, §14 werden nicht gegrillt; ihre Inhalte entstehen in den Feature-Workflows.

Mandatory sections per template: document header, §1–§7, §9–§14, Anhang D — davon §3, §4, §5, §6, §10, §14 nur als Platzhalter (Headings + Tabellenköpfe + Statusvermerk, kein Inhalt, siehe §2 dieses Skills). §8 only when the repo actually uses cryptography; Anhänge A–C nur für einen passenden Stack (Checkboxen ungeprüft lassen — Häkchen setzen ist Feature-Arbeit).

If the human cannot resolve a mandatory section either → hard stop: report the unresolved sections and end. A doc with placeholders in the mandatory part is not a deliverable.

## 4. Output

1. Copy `assets/anwendungsdokumentation-template.md` to the configured path; fill every section in place, headings verbatim, source comments stripped. Changelog: Version 1.0, `Initiale Version`, today's date, author = developer.
2. Copy `assets/README-app-template.md` to the repo root as `README.md` and fill it. If a README already exists, extend it instead: add the documentation link and the owner block, keep the rest.

## 5. DoD

- [ ] Every mandatory section filled or placeholder per §2 — no `<...>` placeholders left in the mandatory part; §3, §4, §5, §6, §10, §14 carry a status note instead of content by design
- [ ] Only implementation-independent sections grilled (header, §1, §2, §7, §9, §11–§13); every grilled value confirmed or corrected by the developer; nothing asked that the repo already answered
- [ ] BSI/ISO mapping (Anhang D) present unchanged
- [ ] §8 and Anhänge A–C filled for a matching stack or omitted deliberately (Anhang-Checkboxen ungeprüft)
- [ ] README points to the app docs; no run commands invented before a manifest exists (§14 ist Platzhalter — README verweist darauf)
- [ ] Headings compare equal to the template — structure untouched

## 6. Error paths

- **Template asset missing from `assets/`:** stop; report — the skill installation is broken. Never reconstruct the template from memory.
- **Repo yields no usable facts** (empty repo, no manifests): stop; nothing to draft from — the developer describes the application in free text first.
- **Existing doc contradicts the repo** (stack in doc ≠ manifests): present both variants to the developer; a filled section of an existing doc is never overwritten silently.
