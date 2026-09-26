# Secure-SDLC Skills

Installationsgrundlage für den Secure-SDLC-Workflow (pi/Claude-Code-Skills): vom Feature-Intake über Spezifizieren, Threat-Model, Testfälle und Ticket-Umsetzung bis zu Security-Review und Merge in die Anwendungsdokumentation.

Orchestrator: **`secure-feature`** — sechs Phasen, Zustand in `STATUS.md`, Artefakte als Templates.

## Inhalt

### Workflow-Kette

| Skill | Rolle |
|---|---|
| `secure-feature` | Orchestrator (Phasen, STATUS.md, Templates) |
| `grilling` | Spezifizieren — Frontier-Fragerunden mit Empfehlung |
| `domain-modeling` | CONTEXT.md-Glossar + ADRs (`docs/adr/`) |
| `threat-model` | STRIDE-Bedrohungsmodell pro Feature |
| `to-spec` | Delta-Spezifikation → `docs/agents/artefakt-erweiterung-to-spec-to-tickets.md` |
| `to-tickets` | Quiz → publizierte Tickets (Blocker-Reihenfolge) |
| `implement-ticket` | Ein Ticket pro Session: claimen, test-first, ein Commit, resolven |
| `review-ticket` | Per-Ticket-Review (Spec/Security/Conventions) — Loop-Partner |
| `code-review` | Standards- + Spec-Achse (Security-Review-Phase) |
| `security-review` | Security-Achse: Mitigations, Abuse-Case-Abdeckung, Findings |
| `merge-app-docs` | Feature-Artefakte in `docs/anwendungsdokumentation.md` mergen |
| `init-app-docs` | Erstbefüllung der Anwendungsdokumentation (vor dem Workflow) |

### Laufzeit-Abhängigkeiten

| Skill | Warum |
|---|---|
| `tdd` | `implement-ticket` folgt ihm; Artefakt-Templates referenzieren ihn |
| `codebase-design` | Vokabular-Referenz für `tdd` (deep modules, seams) |
| `triage` | Triage-Rollen; `to-tickets`/`to-spec` labeln damit |
| `setup-matt-pocock-skills` | Einmal-Setup pro Repo: Issue-Tracker, Triage-Labels, Domain-Docs |

### Konventions-Dokumente (`docs/agents/`)

| Datei | Verwendung |
|---|---|
| `issue-tracker.md` | Tracker-Adapter (lokales Markdown / GitHub / GitLab) |
| `triage-labels.md` | Label-Strings der fünf Triage-Rollen |
| `domain.md` | Single- vs. Multi-Context-Regeln (CONTEXT.md, ADRs) |
| `artefakt-erweiterung-to-spec-to-tickets.md` | Delta-Spezifikation für den Artefakt-Pass |
| `secure-sdlc-konventionen.md` | Übergreifende SDLC-Konventionen |

## Installation in ein Ziel-Repo

1. **Skills kopieren** — den Inhalt von `skills/` nach `.pi/skills/` im Ziel-Repo (Claude Code: `.claude/skills/`).
2. **Konventions-Dokumente kopieren** — `docs/agents/` in das Ziel-Repo (Pfade sind hart kodiert, z. B. greifen `secure-feature`/`code-review` auf `docs/agents/issue-tracker.md` zu).
3. **AGENTS.md mergen** — die drei Abschnitte aus der Vorlage `AGENTS.md` in die AGENTS.md/CLAUDE.md des Ziel-Repos übernehmen und projektspezifisch anpassen.
4. **Setup-Skill laufen lassen** — `/setup-matt-pocock-skills` interaktiv ausführen. Es *überschreibt* die mitgelieferten `docs/agents/*`-Dateien mit der getroffenen Konfiguration (Tracker-Wahl, Labels, Domain-Layout) — die mitgelieferten Dateien sind nur Referenz-Beispiele aus dem Ursprungsprojekt.
5. **App-Docs initialisieren** — falls `docs/anwendungsdokumentation.md` fehlt: `/init-app-docs` (hartes Stop-Kriterium des Orchestrators).
6. **Loslegen** — `/secure-feature <feature>` oder Ticket-Referenz.

## Projekt-spezifisch anzupassen

- **`skills/review-ticket/templates/conventions/csharp.md`** — Beispiel-Conventions aus dem Ursprungsprojekt (C#/.NET 10, `AccessExportTool.*`-Namespaces). Für ein neues Projekt ersetzen; `review-ticket` kopiert diese Datei als Fallback-`CONVENTIONS.md` ans Repo-Root. Eine vorhandene `CONVENTIONS.md` wird nie überschrieben.
- **`docs/agents/*`** — siehe Schritt 4: werden pro Repo generiert.
- **Sprache der Artefakte** — Templates und Skills sind auf Deutsch (Projektkonvention des Ursprungsprojekts).

## Bewusst nicht enthalten

- `writing-for-agents` (Meta-Skill zur Pflege von Skills/AGENTS.md) — nicht nötig zur Laufzeit.
- `archify`, `prototype`, `research`, `wizard`, `ponytail`, `diagnosing-bugs` etc. — unabhängige Werkzeuge, keine Workflow-Abhängigkeit (max. optionale Erwähnungen im Fließtext).
