---
name: setup-secure-sdlc
description: "Einmal-Setup pro Ziel-Repo: installiert die repo-lokalen Secure-SDLC-Konventionen aus den gebündelten Templates (docs/agents/*, AGENTS.md-Abschnitt), delegiert Tracker-/Triage-/Domain-Konfiguration an setup-matt-pocock-skills und prüft die Voraussetzungen für secure-feature (App-Docs, CONVENTIONS). Run once after installing the skills, before the first /secure-feature."
disable-model-invocation: true
---

# Setup Secure-SDLC

Vervollständigt die Installation des Secure-SDLC-Skill-Sets im **Ziel-Repo** (das Arbeitsverzeichnis). Die Skills selbst sind harness-weit installiert (`pi install` / `omp install` / `npx skills add`); dieser Skill legt die **Repo-lokalen Konventions-Dateien** an, ohne die die Workflow-Kette nicht läuft — `secure-feature`, `code-review`, `security-review` und der Artefakt-Pass lösen `docs/agents/*`-Pfade hart auf.

Ground rules:

- **Idempotent:** vorhandene, vom Entwickler gepflegte Dateien werden nie still überschrieben — Inhalt zeigen und fragen.
- **Templates nur aus `templates/`** dieses Skills kopieren; nie aus dem Gedächtnis rekonstruieren.
- **Keine doppelte Logik:** Tracker-Adapter, Triage-Labels und Domain-Doc-Layout gehören `setup-matt-pocock-skills`; dieser Skill delegiert und ergänzt.
- Stop-Regeln sind **harte Stop-Stellen**: halt, report, auf den Entwickler warten.

## 1. Explore (Repo-Zustand)

Vor jedem Schritt den Ausgangszustand lesen, nichts annehmen:

- `docs/agents/`: welche der fünf Konventions-Dateien existieren bereits (`issue-tracker.md`, `triage-labels.md`, `domain.md` — verwaltet von `setup-matt-pocock-skills`; `secure-sdlc-konventionen.md`, `artefakt-erweiterung-to-spec-to-tickets.md` — verwaltet von diesem Skill)?
- `AGENTS.md` / `CLAUDE.md` am Repo-Root: welche existiert, enthält sie bereits einen `## Agent skills`-Block oder einen `## Secure-SDLC`-Abschnitt?
- `docs/anwendungsdokumentation.md`: existiert sie, trägt sie noch Template-Platzhalter?
- `CONVENTIONS.md` am Repo-Root?
- Skills-Verfügbarkeit: sind `setup-matt-pocock-skills`, `triage`, `secure-feature`, `init-app-docs` sowie `wayfinder` (nur Profil `groß` nötig) installiert (Skill-Liste bzw. Geschwister-Verzeichnisse dieses Skills)?

Ergebnis dem Entwickler in fünf Zeilen zusammenfassen: vorhanden / fehlt / wird übersprungen.

## 2. Basiskonfiguration delegieren

Rufe den Skill `setup-matt-pocock-skills` auf (Skill-Tool bzw. `/skill:setup-matt-pocock-skills`; ist er nicht aufrufbar, lies seine `SKILL.md` aus demselben Skill-Verzeichnis und führe sie aus). Er übernimmt:

- `docs/agents/issue-tracker.md` (GitHub / GitLab / lokal / Other)
- `docs/agents/triage-labels.md` (nur wenn `triage` installiert ist)
- `docs/agents/domain.md` (Single- vs. Multi-Context)
- Den `## Agent skills`-Block in `AGENTS.md` bzw. `CLAUDE.md` (Datei-Wahl trifft er selbst)

Vorbedingung: `docs/agents/` existiert — Verzeichnis ggf. vorher anlegen. Ist die Basiskonfiguration bereits erfolgt (drei Dateien vorhanden, Block in AGENTS.md), Schritt überspringen und kurz vermerken.

## 3. Secure-SDLC-Konventionen installieren

Kopiere die beiden Templates aus `templates/docs/agents/` dieses Skills nach `docs/agents/`:

- `secure-sdlc-konventionen.md` — Artefakt-Contracts, Kontextbudget, Profile, Zustandsführung, Anwendungsdoku-Regeln
- `artefakt-erweiterung-to-spec-to-tickets.md` — Delta-Spezifikation des Artefakt-Passes

Eine der beiden Dateien existiert bereits → aktuellen Inhalt zeigen, Unterschiede benennen, ausdrücklich bestätigen lassen; dann ersetzen oder belassen wie er ist.

## 4. AGENTS.md — Secure-SDLC-Abschnitt

Hänge den Inhalt von `templates/AGENTS-secure-sdlc-section.md` an **dieselbe Datei** an, die `setup-matt-pocock-skills` in Schritt 2 gewählt hat (`CLAUDE.md` oder `AGENTS.md`). Ein bereits vorhandener `## Secure-SDLC`-Abschnitt wird in-place aktualisiert, niemals dupliziert; fremde Abschnitte bleiben unangetastet.

## 5. CONVENTIONS.md-Check

Fehlt `CONVENTIONS.md` am Repo-Root, den Entwickler informieren: `review-ticket` kopiert sonst sein mitgeliefertes **C#-Beispiel** als Fallback-`CONVENTIONS.md` (Namespaces/Beispiele des Ursprungsprojekts). Anbieten:

- **Projektspezifisch anlegen** (empfohlen): ein knappes Gerüst (Sprache/Framework, Test-Konventionen, Commit-Regeln, verbindliche Patterns) aus den Repo-Signalen (Manifeste, bestehende Tests, CI) entwerfen, einmal bestätigen lassen, schreiben.
- **Fallback akzeptieren:** nichts tun; `review-ticket` legt zur Laufzeit das C#-Beispiel an (eine vorhandene `CONVENTIONS.md` wird nie überschrieben).

## 6. App-Docs-Check (harte Voraussetzung)

Prüfe den konfigurierten Pfad (Default `docs/anwendungsdokumentation.md`, ggf. im `## Secure-SDLC`-Abschnitt konfiguriert):

- **Datei fehlt oder trägt Template-Platzhalter** in Pflichtabschnitten → `init-app-docs` aufrufen (`/skill:init-app-docs`) und das Setup damit abschließen. Weigert sich der Entwickler, hier anzufangen: harter Stop mit dem Hinweis, dass `secure-feature` in §2 Setup ebenso stoppt — der Workflow startet erst nach initialisierter App-Doku.
- **Datei vollständig** → nur vermerken und weiter.

## 7. Abschluss-Verifikation

Tabelle liefern (Datei | Status | geprüft gegen):

- [ ] `docs/agents/issue-tracker.md`, `triage-labels.md`, `domain.md` — von `setup-matt-pocock-skills` geschrieben
- [ ] `docs/agents/secure-sdlc-konventionen.md`, `artefakt-erweiterung-to-spec-to-tickets.md` — aus diesem Skill installiert
- [ ] `## Agent skills`-Block und `## Secure-SDLC`-Abschnitt in derselben AGENTS.md/CLAUDE.md, keine Duplikate
- [ ] `docs/anwendungsdokumentation.md` initialisiert
- [ ] CONVENTIONS-Entscheidung getroffen (projektspezifisch oder Fallback)
- [ ] `wayfinder` installiert oder Profil-`groß`-Fallback (breadth-first-Grilling) bewusst akzeptiert

**Zusatzwarnung (kein Stop):** `secure-feature` erwartet zudem die zentralen Artefakt-Templates im `templates/`-Verzeichnis des Ziel-Repos (§2 Setup seiner SKILL.md). Fehlt das Verzeichnis, dort ebenfalls stoppt der Orchestrator — rechtzeitig anlegen (Templates sind Contracts, siehe `secure-sdlc-konventionen.md` §4).

Dem Entwickler abschließend sagen, dass alle Konventions-Dateien später direkt editierbar sind und ein Re-Run dieses Skills nur bei strukturellen Änderungen nötig ist.

## Fehlerpfade

- **Template-Datei fehlt in `templates/`:** Stop — die Skill-Installation ist unvollständig (bei Paket-/Symlink-Installation: Repo aktualisieren; bei Copy-Installation: Skill neu installieren).
- **`setup-matt-pocock-skills` nicht installiert:** Stop mit Installationshinweis — ohne ihn fehlen Tracker-Adapter und AGENTS.md-Block, der Rest wäre wirkungslos.
- **`triage` nicht installiert:** kein Stop — `triage-labels.md` entfällt (so verhält sich auch `setup-matt-pocock-skills`), im Abschluss vermerken.
- **`wayfinder` nicht installiert:** kein Stop — Profil `klein` braucht ihn nicht; Profil `groß` fällt laut `secure-feature` §4 auf breadth-first-Grilling mit derselben Pflicht-Security-Frontier zurück.
- **Weder `AGENTS.md` noch `CLAUDE.md` existieren:** `setup-matt-pocock-skills` fragt selbst, welche Datei angelegt werden soll; Schritt 4 folgt dann seiner Wahl.
