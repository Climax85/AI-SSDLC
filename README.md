# Secure-SDLC Skills

Installationsgrundlage für den Secure-SDLC-Workflow (pi/Claude-Code-Skills): vom Feature-Intake über Spezifizieren, Threat-Model, Testfälle und Ticket-Umsetzung bis zu Security-Review und Merge in die Anwendungsdokumentation.

Orchestrator: **`secure-feature`** — sechs Phasen, Zustand in `STATUS.md`, Artefakte als Templates.

## Inhalt

### Workflow-Kette

| Skill | Rolle |
|---|---|
| `secure-feature` | Orchestrator (Phasen, STATUS.md, Templates); legt das Feature-Epic im konfigurierten Tracker an |
| `grilling` | Spezifizieren — Frontier-Fragerunden mit Empfehlung |
| `domain-modeling` | CONTEXT.md-Glossar + ADRs (`docs/adr/`) |
| `threat-model` | STRIDE-Bedrohungsmodell pro Feature |
| `to-spec` | Delta-Spezifikation → `docs/agents/artefakt-erweiterung-to-spec-to-tickets.md` |
| `to-tickets` | Quiz → publizierte Tickets im konfigurierten Tracker als Kind-Tickets des Feature-Epics (Blocker-Reihenfolge) |
| `implement-ticket` | Ein Tracker-Ticket pro Session auf eigenem Feature-Branch: claimen, test-first, ein Commit, Review, PR, resolven |
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
| `wayfinder` | Profil `groß` in `secure-feature` Spezifizieren: Planungs-Engine (Map/Fog-of-War auf dem Tracker) mit Pflicht-Security-Frontier; Fallback ohne ihn: breadth-first-Grilling |
| `research` | Wayfinder-Ticket-Typ `research` (AFK): Recherche an Primärquellen via Hintergrund-Agent, Ergebnis als Markdown-Datei mit Quellenangaben |
| `prototype` | Wayfinder-Ticket-Typ `prototype` (HITL): Wegwerf-Prototyp zur Beantwortung von Designfragen — Logic-Demo (eine HTML-Datei) oder UI-Varianten (`?variant=`) |
| `setup-matt-pocock-skills` | Einmal-Setup pro Repo: Issue-Tracker, Triage-Labels, Domain-Docs |
| `setup-secure-sdlc` | Einmal-Setup pro Repo (Orchestrator): installiert `docs/agents`-Konventionen aus Skill-Templates, mergt AGENTS.md, delegiert an `setup-matt-pocock-skills`, prüft App-Docs/CONVENTIONS |

### Konventions-Dokumente (`docs/agents/`)

| Datei | Verwendung |
|---|---|
| `issue-tracker.md` | Tracker-Adapter (lokales Markdown / GitHub / GitLab) |
| `triage-labels.md` | Label-Strings der fünf Triage-Rollen |
| `domain.md` | Single- vs. Multi-Context-Regeln (CONTEXT.md, ADRs) |
| `artefakt-erweiterung-to-spec-to-tickets.md` | Delta-Spezifikation für den Artefakt-Pass |
| `secure-sdlc-konventionen.md` | Übergreifende SDLC-Konventionen |

## Skill-Installation (ein Befehl)

Das Repo ist ein pi-Package (`package.json` mit `pi.skills` und `pi.subagents.agents`) und folgt der Agent-Skills-Konvention (`skills/<name>/SKILL.md`) — alle 20 Skills sind name-/frontmatter-konform und werden von pi, oh-my-pi und opencode nativ entdeckt. Der Subagent `agents/review-ticket.md` macht die Review-Delegation von `implement-ticket` in pi (mit pi-subagents) zum First-Class-Aufruf; `setup-secure-sdlc` installiert ihn projekt-lokal nach `.pi/agents/`.

**pi coding agent** — klont das Repo nach `~/.pi/agent/git/` und verlinkt es in den Einstellungen:

```bash
pi install git:github.com/Climax85/AI-SSDLC          # global (user-weit)
pi install -l git:github.com/Climax85/AI-SSDLC       # projektweit (.pi/settings.json, wird vom Team geteilt)
```

**oh-my-pi (omp)** — gleiches Repo, eigene Plugin-Verwaltung:

```bash
omp install github:Climax85/AI-SSDLC
# oder getagged:  omp install 'github:Climax85/AI-SSDLC#v1.0.0'
```

**opencode** — kein eigener Paketmanager; der Agent-Skills-CLI (Vercel) installiert in die jeweiligen Skill-Verzeichnisse und verwaltet Updates (Lockfile, `npx skills update`):

```bash
npx skills add Climax85/AI-SSDLC --skill '*' -a pi -a opencode -y        # Projekt-Scope
npx skills add Climax85/AI-SSDLC --skill '*' -a pi -a opencode -y -g     # Global
```

> Tipp: omp und opencode lesen zusätzlich das universelle Verzeichnis `.agents/skills/` (projekt) bzw. `~/.agents/skills/` — pi ebenfalls. Ein Symlink dieses Repos dorthin funktioniert für alle drei Harnesses gleichzeitig und aktualisiert sich per `git pull` (unter Windows: `git config core.symlinks true` oder `npx skills add --copy`).

Updates: `pi update --extensions` bzw. `pi update --all` (pi), Ref neu setzen via `omp install 'github:…#<neuer-ref>'` (omp), `npx skills update` (opencode/pi via CLI).

## Einrichtung des Ziel-Repos (ein Befehl)

Die Skills allein reichen nicht — die Workflow-Kette erwartet pro Ziel-Repo Konventions-Dokumente und App-Docs (hart kodierte Pfade in `skills/secure-feature`, `skills/code-review` u. a.). Das übernimmt der Setup-Skill:

1. **`/skill:setup-secure-sdlc`** im Ziel-Repo ausführen. Er erkundet den Repo-Zustand, ruft `setup-matt-pocock-skills` auf (Tracker-Wahl, Triage-Labels, Domain-Layout inkl. `## Agent skills`-Block in AGENTS.md/CLAUDE.md), installiert danach die Secure-SDLC-Konventions-Dateien aus seinen gebündelten Templates nach `docs/agents/`, ergänzt einen `## Secure-SDLC`-Abschnitt in derselben AGENTS.md/CLAUDE.md, legt den Subagent `review-ticket` nach `.pi/agents/` an (delegiert wird damit statt im teuren Inline-Fallback), klärt die CONVENTIONS-Entdeckung (`review-ticket`-Fallback ist ein C#-Beispiel) und prüft die App-Docs (`init-app-docs` wird delegiert bzw. als harte Voraussetzung gemeldet).
2. **Loslegen** — `/skill:secure-feature <feature>` oder Ticket-Referenz.

Idempotent: bereits gepflegte Dateien werden nicht still überschrieben. Der Skill warnt außerdem, falls das zentrale Artefakt-Template-Verzeichnis `templates/` im Ziel-Repo fehlt (Stop-Stelle von `secure-feature` §2 Setup).

## Projekt-spezifisch anzupassen

- **`skills/review-ticket/templates/conventions/csharp.md`** — Beispiel-Conventions aus dem Ursprungsprojekt (C#/.NET 10, `AccessExportTool.*`-Namespaces). Für ein neues Projekt ersetzen; `review-ticket` kopiert diese Datei als Fallback-`CONVENTIONS.md` ans Repo-Root. Eine vorhandene `CONVENTIONS.md` wird nie überschrieben.
- **`docs/agents/*`** — werden pro Repo von `setup-matt-pocock-skills` (drei Dateien) bzw. `setup-secure-sdlc` (zwei Dateien) generiert.
- **Sprachen** — Markdown-Dokumentation (Artefakte, Templates, App-Docs, Tickets) auf Deutsch; Code auf Englisch (Identifier, Variablen, Kommentare, Testnamen, Commit-Messages, Branch-Namen). Verbindliche Regel: `docs/agents/secure-sdlc-konventionen.md` §4.

## Bewusst nicht enthalten

- `writing-for-agents` (Meta-Skill zur Pflege von Skills/AGENTS.md) — nicht nötig zur Laufzeit.
- `archify`, `wizard`, `ponytail`, `diagnosing-bugs` etc. — unabhängige Werkzeuge, keine Workflow-Abhängigkeit (max. optionale Erwähnungen im Fließtext).

## Herkunft und Lizenz

Die Pocock-Basis-Skills (`grilling`, `to-spec`, `to-tickets`, `triage`, `tdd`, `codebase-design`, `domain-modeling`, `code-review`, `setup-matt-pocock-skills`, `wayfinder`, `research`, `prototype`) stammen aus [mattpocock/skills](https://github.com/mattpocock/skills) (MIT) — teils unverändert übernommen (u. a. `wayfinder`, `research`, `prototype`), teils dokumentiert erweitert (siehe `docs/agents/secure-sdlc-konventionen.md` §1). Damit deckt das Bundle alle vier Wayfinder-Ticket-Typen ab (`research`, `prototype`, `grilling`, `task`).
