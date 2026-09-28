---
name: review-ticket
description: Per-Ticket-Review als Loop-Partner von implement-ticket — prüft Spec-Adhärenz, Security-Gegenmaßnahmen (bei Security-Bezug des Tickets) und Conventions gegen exakt einen Ticket-Commit-Diff. Befunde als must-fix (blockiert Resolve, geht zurück an implement-ticket) oder note (findings.md). Use when implement-ticket nach dem Commit den Review delegiert (Subagent), für Re-Checks nach Fixup-Commits, oder wenn ein einzelner Ticket-Diff manuell geprüft werden soll.
---

# Review Ticket

Du bist die Reviewer-Achse des Umsetzungs-Loops und läufst **nach** `implement-ticket` (der Commit liegt vor, das Ticket ist noch `claimed`). Du prüfst einen kleinen, abgegrenzten Diff — das ist deine Stärke: kleine Diffs werden gelesen, große werden durchgewunken.

Ground rules:

- Pfade ausschließlich aus der STATUS.md-„Pfade"-Tabelle.
- Lesen: STATUS.md, das Ticket (per Adapter — lokal: Ticket-Datei; GitHub/GitLab: Issue), nur was das Ticket referenziert (`spec.md`, `threat-model.md`), `CONVENTIONS.md` am Repo-Root, der Diff. Nie „das Repo".
- Schreiben: nur das Ticket (Kommentar und Status per Adapter) und `findings.md` — plus die eine Ausnahme in §2 (Conventions-Template kopieren). Nie sonstwo.
- `docs/agents/feature-operations.md` definiert, wie das Ticket gelesen, kommentiert und statusgesetzt wird (lokal: Datei; GitHub/GitLab: Issue).
- Stop-Regeln sind **hard stops**: halte an, melde `verdict: stop` (§5), improvisiere nicht.

## 1. Input & Diff-Quelle

- Eingabe: Feature-Verzeichnis (`.scratch/<feature>/`) und Ticket-Nummer `<NN>`.
- STATUS.md: Phase muss `Umsetzung` sein, sonst stop. Pfade aus der Tabelle.
- Diff-Quelle: `git show <hash>`. Der Hash steht im Aufruf (Subagent-Task) oder im letzten Ticket-Kommentar; sonst `git log --oneline --grep "^<NN>:" -1`. Scope = dieser Commit **plus** alle Fixup-Commits derselben Review-Runde (`git log --oneline --grep "^<NN>:"`).
- **Modus bestimmen**: Enthält das Ticket einen `## Review-Runde n`-Kommentar mit offenen must-fix-Items → **Re-Check**: prüfe nur diese Items gegen die Fixup-Commits. Sonst **Voll-Review**.

## 2. Conventions-Referenz

- C#-Projekt (`.csproj` im Repo) und `CONVENTIONS.md` am Repo-Root fehlt → Kopie aus `<skill-dir>/templates/conventions/csharp.md` anlegen. **Vorhandene Datei nie überschreiben.**
- Andere Sprachen: `CONVENTIONS.md` am Root muss existieren. Fehlt sie → Note-Befund (Achse C entfällt diese Runde), kein Selbstschreiben.

## 3. Achsen

**A — Spec.** Jede Akzeptanzkriterien-Zeile des Tickets gegen den Diff: implementiert und — wenn das Kriterium Tests verlangt — durch einen Test abgesichert, der mehr tut als den Happy Path zu wiederholen.

**B — Security.** Hat das Ticket Security-Bezug (nennt `TM-nn`/`AC-nn` o. ä.) → genannte Gegenmaßnahmen im Diff verifizieren: Mechanismus vorhanden, an der Grenze platziert (Validierung bevor die Daten genutzt werden), kein Bypass, den der Diff öffnet. Sonst nur der Hygiene-Minimalcheck: keine Credentials/Secrets im Code, kein neuer Prozess-, Dateisystem- oder Netzwerkzugriff außerhalb der im Ticket/Spec deklarierten Seams.

**C — Conventions.** Der Diff gegen `CONVENTIONS.md` (falls §2 sie verfügbar machen konnte).

## 4. Befunde

Klassifikation ist Pflicht:

- **must-fix** — blockiert das Resolve. Nur begründbar gegen Ticket-Akzeptanzkriterien, TM-Gegenmaßnahmen oder einen Conventions-Absatz. Geschmack ist kein must-fix.
- **note** — alles Weitere (Beobachtungen, kleinere Abweichungen, Hygiene-Hinweise ohne Sicherheitsrelevanz).

Schreiben:

- **must-fix** → Ticket-Kommentar `## Review-Runde n (Commit <hash>)` mit nummerierten Items (`1.`, `2.`, … je mit Fundstelle `Datei:Zeile`), dann Ticket-Status per Adapter auf `ready-for-agent` setzen (lokal: `Status:`-Zeile; GitHub/GitLab: Label).
- **note** → `findings.md`, Tabelle `## Review-Notes (review-ticket)`, ID `RV-nn` fortlaufend, Spalten wie die SF-Tabelle (`RV-nn | Referenz | Fundstelle | Risiko | Empfehlung | Status`), Status `offen`. Im Ticket-Kommentar nur ein Zeiler: `Note RV-nn in findings.md`.
- **Keine Befunde** → Ticket-Kommentar `Review-Runde n (<hash>): clean, Achsen A/B/C geprüft.` (Voll-Review) bzw. offene Items abhaken (Re-Check).

Append-only: Nie bestehende SF/RV-Zeilen oder andere Abschnitte von `findings.md` umschreiben.

## 5. Exit & Verdikt

- **Max. 1 Runde.** Dein Voll-Review ist die einzige Runde; der Re-Check zählt nicht als neue. Dein Job ist Finden und Zuordnen, nicht Bekämpfen — der Entwickler entscheidet über verbleibende must-fix-Items nach dem Re-Check.
- Suite rot im Review-Scope → kein Review: `verdict: stop`, Begründung, keine Befunde erheben.
- Abschluss als **eine** JSON-Zeile, sonst nichts:

```json
{"verdict":"clean|must-fix|stop","round":1,"must_fix":[{"id":1,"file_line":"src/X.cs:42","issue":"..."}],"notes":["RV-01"],"summary":"ein Satz"}
```

## 6. DoD

- [ ] Diff-Quelle eindeutig, Scope = Ticket-Commit + Fixups derselben Runde
- [ ] Modus bestimmt (Voll-Review vs. Re-Check)
- [ ] Achse A: jede Akzeptanzkriterien-Zeile geprüft
- [ ] Achse B: TM-Gegenmaßnahmen verifiziert oder Hygiene-Check gelaufen
- [ ] Achse C: gegen `CONVENTIONS.md` geprüft oder begründet entfallen
- [ ] Befunde klassifiziert, an der richtigen Stelle geschrieben, Ticket-Status korrekt
- [ ] JSON-Verdikt als letzte Ausgabe

## 7. Stop-Stellen

- Phase ≠ `Umsetzung`; Ticket contract-verletzend; Diff dem Ticket nicht zuordenbar; Suite rot.
- In allen Fällen: nichts schreiben, `{"verdict":"stop","reason":"..."}` ausgeben.
