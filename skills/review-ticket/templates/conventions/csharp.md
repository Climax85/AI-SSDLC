# CONVENTIONS.md — C#

Verbindliche Coding-Conventions für Produkt- und Testcode dieses Repos (Review-Referenz für `review-ticket`, Achse C). Abgeleitet aus dem gelebten Stil (CLI-Gerüst, Namens-Sanitizer); Ergänzungen nur per Entwickler-Entscheidung.

## Projekt-Grundeinstellungen

- `TargetFramework net10.0`, `Nullable enable`, `ImplicitUsings enable` — in beiden Projekten (`src/`, `tests/`).
- Keine externen Laufzeit-Abhängigkeiten im Produktcode; Test-Stack: xUnit + coverlet.

## Struktur

- Namespace = Ordner: `AccessExportTool.<Bereich>` (z. B. `AccessExportTool.Cli`, `AccessExportTool.Names`); Tests spiegeln das unter `AccessExportTool.Tests.<Bereich>`.
- Pure Logik (kein I/O, kein COM, kein Prozess-Zugriff) lebt als `static class` in eigenem Ordner — sie ist der Unit-Seam und darf ihre Reinheit nicht verlieren.
- Unveränderliche Daten und Resultat-Typen sind `record` (abstract record mit geschlossenen Sub-records für Resultat-Fälle, wie `SanitizedNameResult`).

## Sprache

- Identifier und Testnamen: Englisch (`snake_case` bei Testmethoden, wie `path_separators_lead_to_rejection`).
- XML-Doc-Kommentare und Begründungskommentare: Englisch (Code-Sprachregel, siehe `secure-sdlc-konventionen.md` §4 — nur Markdown-Dokumentation ist deutsch).
- `var` nur, wo der Typ evident ist; ansonsten explizit.

## Sicherheitsrelevante String-Regeln

- Stringvergleiche und `StartsWith`/`EndsWith`/`Contains` immer mit explizitem `StringComparison` (default: `Ordinal`; case-insensitiv: `OrdinalIgnoreCase`). Nie kulturabhängige Vergleiche an Sicherheitsentscheidungen.
- Pfade und Dateinamen niemals durch Konkatenation bauen, wo ein Traversal-Eingriff möglich ist — Zielnamen kommen ausschließlich aus dem Sanitizer.

## Tests

- Verhalten über öffentliche Schnittstellen prüfen, nie über Interna.
- Erwartungswerte sind unabhängige Wahrheitsquellen (Literale aus Spec/Ticket), keine aus dem Code abgeleiteten Ausdrücke — keine Tautologien.
- Negativfälle und Edge-Cases als `[Theory]` mit benannten `InlineData`; jeder Theorie-Datensatz dokumentiert in einem Kommentar, welche Regel er triggert.
- Arrange/Act/Assert in dieser Reihenfolge, ohne Kommentar-Markierungen — die Struktur trägt sich selbst.
