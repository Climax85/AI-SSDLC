# Anwendungsdokumentation — `<Anwendungsname>`

> **Dokumentennummer:** `<DOC-ID>`
> **Version:** `<1.0>`
> **Gültig ab:** `<YYYY-MM-DD>`
> **Verantwortlich:** `<Product Owner / Anwendungsverantwortlicher>`
> **Technischer Ansprechpartner:** `<Teamleiter / Senior-Entwickler>`
> **Genehmigt durch:** `<ISB / Fachbereichsleiter>`
> **Nächster Review:** `<YYYY-MM-DD>`
> **Repository / Pfad:** `<URL oder Pfad im Repo>`

---

## 1. Metadaten *(Pflicht)*

| Feld | Inhalt |
|---|---|
| Anwendungsname | `<Name>` |
| Kurzbeschreibung | `<Was macht die Anwendung?>` |
| Verantwortlicher Fachbereich | `<Bereich>` |
| Technischer Verantwortlicher | `<Name>` |
| Einsatzart | `Webanwendung / Desktop-Anwendung / Kommandozeilen-Anwendung / Datenbank-Frontend` |
| Technologiestack | `ASP.NET 8 / Qt 6 C++ / MS Access + VBA` |
| Betriebsmodus | `On-Premise / Cloud / Hybrid` |
| Kritikalität | `hoch / mittel / niedrig` |
| Schutzbedarf | `vertraulich / ausfallsicher / integer` |
| Letzter Sicherheitsreview | `<YYYY-MM-DD>` |

**BSI/ISO-Bezug:** APP.7.A1 (Planung), A.5.37 (Documented operating procedures)

---

## 2. Zweck, Betriebseigner, Daten & Schutzbedarf *(Pflicht)*

### 2.1 Geschäftlicher Zweck
- Welches Problem löst die Anwendung?
- Wer sind die primären Nutzer:innen / Stakeholder?
- Geschäftskritikalität und Auswirkungen bei Ausfall oder Kompromittierung.

### 2.2 Betriebseigner und Verantwortlichkeiten
| Rolle | Name | Verantwortung |
|---|---|---|
| Betriebseigner | | Fachliche Verantwortung, Anforderungen, Budget |
| Technischer Verantwortlicher | | Architektur, Betrieb, Sicherheitsupdates |
| Secure-Coding-Beauftragter | | Sicherheitsreviews, Schwachstellenmanagement |
| Betrieb / DevOps | | Deployment, Monitoring, Backup |

### 2.3 Verarbeitete Daten und Schutzbedarf
| Datenkategorie | Beispiele | Schutzbedarf | Speicherort |
|---|---|---|---|
| Personenbezogene Daten | | hoch | |
| Geschäftsgeheimnisse | | hoch | |
| Betriebsdaten | | mittel | |
| Logs / Metadaten | | niedrig | |

**BSI/ISO-Bezug:** APP.7.A6 (Anforderungen & Sicherheitsprofil), A.8.26 (Application Security Requirements), A.8.12 (Data Leakage Prevention)

---

## 3. Sicherheitsprofil / Risiko- und Bedrohungsmodell *(Pflicht)*

### 3.1 Sicherheitsanforderungen

Dokumentiere die übergeordneten Sicherheitsanforderungen der Anwendung. Orientiere dich an den vier klassischen Sicherheitszielen und beantworte für deine Anwendung die folgenden Leitfragen:

#### Vertraulichkeit
- Welche Daten dürfen von welchen Rollen eingesehen werden? (z. B. jeder Mitarbeiter, nur Abteilung X, nur Admin)
- Gibt es besonders schützenswerte Daten (personenbezogene Daten, Geschäftsgeheimnisse, Kundendaten, Finanzdaten)?
- Wo werden diese Daten gespeichert, verarbeitet und übertragen?
- Welche Verschlüsselung wird im Transport (TLS) und ggf. in der Speicherung (AES, Datenbankverschlüsselung) verwendet?
- Muss die Anwendung gegen interne Bedrohungen geschützt werden (z. B. Mitarbeiter ohne Berechtigung)?

#### Integrität
- Wo ist Manipulationsschutz nötig? (Daten, Konfiguration, Logs, Code, Build-Artefakte)
- Wer darf Daten ändern oder löschen?
- Gibt es Mechanismen zur Erkennung von Veränderungen (Hashing, Signatur, Prüfsummen, Versionskontrolle)?
- Wie wird sichergestellt, dass nur genehmigte Änderungen in Produktion gelangen?

#### Verfügbarkeit
- Wie kritisch ist die Anwendung für den Geschäftsbetrieb?
- Welche maximalen Ausfallzeiten sind akzeptabel? (z. B. RTO / RPO, Betriebszeiten, Wartungsfenster)
- Gibt es Redundanzen, Backups, Failover- oder Wiederanlaufkonzepte?
- Welche Abhängigkeiten (Datenbank, Schnittstellen, Netzwerk) müssen funktionieren?

#### Audit-Nachweis
- Welche sicherheitsrelevanten Vorgänge müssen nachvollziehbar sein?
- Welche Ereignisse werden geloggt (Login, Logout, Berechtigungsänderungen, Datenexport, Fehler)?
- Wie lange müssen Logs aufbewahrt werden?
- Gibt es regulatorische oder kundenspezifische Nachweispflichten?

#### Übersicht: Sicherheitsanforderungen
| ID | Kategorie | Anforderung | Begründung / Bezug | Status |
|---|---|---|---|---|
| SA-01 | Vertraulichkeit | Nur authentisierte Benutzer dürfen auf Daten zugreifen | ORP.4, A.8.2 | umgesetzt |
| SA-02 | Integrität | Änderungen an kritischen Daten sind nachvollziehbar | A.8.16 | umgesetzt |
| SA-03 | Verfügbarkeit | Max. 4h Ausfallzeit an Werktagen tolerierbar | Geschäftsprozess | definiert |
| SA-04 | Audit-Nachweis | Login/Logout und Berechtigungsänderungen werden geloggt | A.8.15 | umgesetzt |
| SA-05 | | | | |

**Hinweise zur Einstufung:**
- Vertraulichkeit: *hoch*, wenn personenbezogene oder besonders schützenswerte Daten verarbeitet werden.
- Integrität: *hoch*, wenn unbemerkbare Manipulation schwerwiegende Folgen hätte.
- Verfügbarkeit: *hoch*, wenn Ausfall Geschäftsprozesse oder Kunden direkt beeinträchtigt.

### 3.2 Bedrohungsmodell *(Threat Modeling)*
| ID | Bedrohung | Asset | Auswirkung | Risiko | Gegenmaßnahme | Status |
|---|---|---|---|---|---|---|
| TM-01 | SQL-Injection | Datenbank | Datenabfluss, Manipulation | hoch | Parametrisierte Abfragen | umgesetzt |
| TM-02 | Unberechtigter Zugriff | Geschäftsdaten | Datenabfluss | hoch | RBAC, Authentisierung | umgesetzt |
| TM-03 | ... | ... | ... | ... | ... | ... |

### 3.3 Akzeptierte Risiken
- Risiken, die bewusst akzeptiert werden (mit Begründung und Verantwortlichem).

**BSI/ISO-Bezug:** CON.8.A21 (Bedrohungsmodellierung), CON.8.A22 (Sicherer Software-Entwurf), APP.7.A6, A.8.27 (Secure Architecture)

---

## 4. Architektur & Architektur-Entscheidungen *(Pflicht)*

### 4.1 Übersichtsdiagramm
- Einbinden eines Architektur-Diagramms (C4-Container-Diagramm, Layer-Diagramm oder vergleichbar).
- Beschreibung der wichtigsten Komponenten und ihrer Verantwortlichkeiten.

### 4.2 Technologie-Stack
| Schicht | Technologie | Version | Bemerkung |
|---|---|---|---|
| Frontend | | | |
| Backend | | | |
| Datenbank | | | |
| Komponenten / Bibliotheken | | | |

### 4.3 Architektur-Entscheidungen (ADRs)
| ID | Entscheidung | Kontext | Konsequenz | Datum |
|---|---|---|---|---|
| ADR-01 | | | | |
| ADR-02 | | | | |

**BSI/ISO-Bezug:** CON.10.A11 (Softwarearchitektur), CON.8.A12 (Ausführliche Dokumentation), A.8.27 (Secure Architecture)

---

## 5. Datenflüsse & Schnittstellen *(Pflicht)*

### 5.1 Externe Schnittstellen
| ID | Name | Protokoll | Authentisierung | Daten | Datenfluss | Verantwortlich | Dokumentation |
|---|---|---|---|---|---|---|---|
| IF-01 | | HTTPS / REST / SOAP / gRPC | OAuth / API-Key / Basic | | eingehend / ausgehend | | |
| IF-02 | | | | | | | | |

### 5.2 Interne Schnittstellen
| ID | Komponente A | Komponente B | Protokoll | Daten | Bemerkung |
|---|---|---|---|---|---|
| IF-10 | | | | | |

### 5.3 Datenflussdiagramm
- Grafische Darstellung, woher Daten kommen, wo sie verarbeitet werden, wo sie gespeichert werden und wohin sie fließen.

**BSI/ISO-Bezug:** APP.7.A3 (Sicherheitsfunktionen zur Systemintegration), APP.3.1.A11 (Schnittstellen), A.5.14 (Information Transfer)

---

## 6. Authentisierung & Autorisierung *(Pflicht)*

### 6.1 Authentisierung
- Verfahren (Form-Based, SSO, Windows-Auth, OAuth2/OpenID Connect, MFA, Zertifikat, API-Key).
- Session-Management (Session-Dauer, Timeout, Renewal, Token-Speicherung).
- Passwort-Policy-Verweis (wenn relevant).

### 6.2 Autorisierung / Berechtigungskonzept
- Rollenmodell (RBAC / ABAC).
- Tabelle der Rollen und Berechtigungen.
- Besondere Berechtigungen (Admin, Service-Accounts).
- Funktionstrennung (SoD).

| Rolle | Berechtigungen | Geltungsbereich |
|---|---|---|
| | | |

### 6.3 Service-Accounts und technische Benutzer
| Account | Verwendung | Berechtigungsumfang | Verantwortlicher |
|---|---|---|---|
| | | | |

**BSI/ISO-Bezug:** CON.10.A1 (Authentisierung), APP.3.1.A1, ORP.4 (IAM), A.8.2 / A.8.5 (Access Control)

---

## 7. Berechtigungsmanagement *(Pflicht)*

### 7.1 Prozess für Zuweisung, Änderung und Entzug
- Wer beantragt Berechtigungen?
- Wer genehmigt?
- Nachweis (Ticket, Formular, Workflow).
- Regelmäßiges Review (z. B. halbjährlich) bestehender Berechtigungen.

### 7.2 Notfallzugriffe
- Break-Glass-Verfahren.
- Protokollierung und Nachweis.

### 7.3 Inaktive Kennungen
- Umgang mit ausscheidenden Mitarbeitern, Dormant-Accounts, Sperrung.

**BSI/ISO-Bezug:** ORP.4 (Identitäts- und Berechtigungsmanagement), A.5.18 (Access Rights)

---

## 8. Kryptografie & Schlüsselmanagement *(Optional — nur wenn Kryptografie genutzt wird)*

### 8.1 Eingesetzte kryptografische Verfahren
| Zweck | Algorithmus | Schlüssellänge | Bibliothek |
|---|---|---|---|
| Transportverschlüsselung | TLS 1.2/1.3 | | |
| Datenspeicherung | AES | 256 Bit | |
| Hashing | SHA-256 / bcrypt / Argon2 | | |
| Signatur | RSA / ECDSA | | |

### 8.2 Schlüsselmanagement
- Schlüsselgenerierung, -speicherung, -rotation, -archivierung, -löschung.
- Verantwortlichkeiten.
- Keine hartcodierten Schlüssel im Quellcode.

**BSI/ISO-Bezug:** CON.1 (Kryptokonzept), A.8.24 (Use of Cryptography)

---

## 9. Logging & Monitoring *(Pflicht)*

### 9.1 Log-Quellen und Ereignisse
- Welche Ereignisse werden geloggt (Login, Logout, Berechtigungsänderungen, Fehler, sensible Datenzugriffe)?
- Log-Level und Format.

### 9.2 Log-Schutz und Aufbewahrung
- Speicherort und -dauer.
- Schutz vor Manipulation und unbefugtem Zugriff.
- Keine personenbezogenen Daten oder Secrets in Logs.

### 9.3 Monitoring und Alarmierung
- Überwachte Metriken und Schwellenwerte.
- Alarmierungskette.
- Incident-Eskalation.

**BSI/ISO-Bezug:** CON.10.A13 (Fehlerbehandlung), APP.3.1.A22 (Revision), A.8.15 (Logging), A.8.16 (Monitoring)

---

## 10. Konfiguration & Hardening *(Pflicht)*

### 10.1 Betriebsumgebung
- Produktivumgebung, Testumgebung, Entwicklungsumgebung (Trennung).
- Hosting-Plattform und Netzwerksegment.
- Zugangswege (VPN, Jump-Host, RDP, Web).

### 10.2 Sicherheitsrelevante Konfigurationen
| Bereich | Konfiguration | Begründung |
|---|---|---|
| HTTP-Header | CSP, HSTS, X-Content-Type-Options, X-Frame-Options | Nur bei Web-Apps |
| Cookies | Secure, HttpOnly, SameSite | Nur bei Web-Apps |
| Datenbank | Zugriffsbeschränkungen, Verschlüsselung | |
| Dateisystem | Berechtigungen | |
| Netzwerk | Firewall-Regeln, Port-Beschränkungen | |

### 10.3 Hardening-Checkliste
- Verweis auf team- bzw. stack-spezifische Hardening-Vorgaben (siehe Anhang).

**BSI/ISO-Bezug:** APP.3.1.A12 (Sichere Konfiguration), APP.3.1.A21 (Sichere HTTP-Konfiguration), CON.8.A5 (Sicheres Systemdesign)

---

## 11. Patch- und Schwachstellenmanagement *(Pflicht)*

### 11.1 Verantwortlichkeiten
- Wer überwacht Schwachstellenmeldungen?
- Wer entscheidet über Einspielung?
- Wer dokumentiert Ablehnungen?

### 11.2 Patch-Prozess
- Quellen (BSI, Hersteller, SCA-Scanner).
- Zeitliche Ziele (z. B. kritisch innerhalb 7 Tage, hoch innerhalb 30 Tage).
- Test- und Rollback-Verfahren.

### 11.3 Schwachstellen- und Patch-Entscheidungslog
| Datum | Schwachstelle | Komponente | Risiko | Entscheidung | Begründung | Verantwortlicher |
|---|---|---|---|---|---|---|
| | | | | | | |

**BSI/ISO-Bezug:** OPS.1.1.3.A15 (Aktualisierung), A.8.8 (Management of technical vulnerabilities)

---

## 12. Änderungshistorie / Changelog *(Pflicht)*

| Version | Datum | Änderung | Autor | Genehmigt durch | Sicherheitsrelevant |
|---|---|---|---|---|---|
| 1.0 | | Initiale Version | | | |

**BSI/ISO-Bezug:** OPS.1.1.3.A11 (Kontinuierliche Dokumentation), A.8.32 (Change Management)

---

## 13. Audit- und Testhistorie *(Pflicht)*

### 13.1 Durchgeführte Sicherheitsprüfungen
| Datum | Art der Prüfung | Durchgeführt von | Ergebnis | Bemerkung |
|---|---|---|---|---|
| | Code-Review | Intern | Keine Findings | |
| | SAST / SCA | CI/CD | | |
| | Penetrationstest | Extern | | |
| | Schwachstellen-Scan | | | |

### 13.2 Abweichungs- und Maßnahmenverfolgung
| ID | Finding | Risiko | Maßnahme | Frist | Status |
|---|---|---|---|---|---|
| | | | | | |

**BSI/ISO-Bezug:** APP.3.1.A22 (Penetrationstest und Revision), A.8.29 (Security Testing), A.8.34 (Protection during audit testing)

---

## 14. Betriebshandbuch / Runbook *(Pflicht)*

### 14.1 Starten, Stoppen, Neustarten
- Schritt-für-Schritt-Anleitung.
- Verantwortlicher und Eskalationskette.

### 14.2 Backup & Wiederanlauf
- Backup-Rhythmus, Aufbewahrung, Restore-Test.
- Wiederanlaufverfahren (Recovery Time Objective, Recovery Point Objective).

### 14.3 Fehlersuche und Eskalation
- Häufige Fehlerbilder und Behebung.
- Kontakte für P1/P2/P3-Störungen.

### 14.4 Business-Continuity-Hinweise
- Kritische Abhängigkeiten.
- Ersatzverfahren bei Ausfall.

**BSI/ISO-Bezug:** CON.8.A12 (Betriebsdokumentation), A.5.37 (Documented operating procedures), A.5.29 / A.5.30 (Continuity)

---

## Anhang A — Stack-spezifische Ergänzungen für ASP.NET 8 *(Optional, bei Web-Apps)*

### A.1 Web-Security-Hardening
- [ ] CSP-Header konfiguriert
- [ ] HSTS aktiviert
- [ ] Anti-CSRF-Token vorhanden
- [ ] Sichere Cookie-Attribute (Secure, HttpOnly, SameSite)
- [ ] Input-Validierung serverseitig
- [ ] Output-Encoding gegen XSS
- [ ] Sichere Session-Konfiguration

### A.2 Deployment & Hosting
- IIS / Kestrel / Container-Konfiguration
- Reverse-Proxy-Einstellungen
- Health-Checks und Graceful Shutdown

### A.3 ASP.NET-spezifische Abhängigkeiten
- NuGet-Pakete und deren Versionsstände
- Entity Framework Core-Konfiguration (Migrations, Connection-Pooling)

**BSI/ISO-Bezug:** CON.10.A1–A16, APP.3.1.A21, A.8.28

---

## Anhang B — Stack-spezifische Ergänzungen für Qt 6 C++ *(Optional, bei Desktop-Apps)*

### B.1 C++-Secure-Coding-Hinweise
- [ ] Keine Buffer-Overflows (sichere APIs, Bounds-Checking)
- [ ] Memory Safety (Smart Pointer, RAII)
- [ ] Sichere Verwendung von Qt-Klassen
- [ ] Keine unsicheren String-Operationen
- [ ] Sichere Datei- und Netzwerkoperationen

### B.2 Desktop-spezifische Sicherheit
- Lokale Konfiguration und Hardening
- Umgang mit lokalen Credentials / Tokens
- Update-Mechanismus der Desktop-Anwendung

**BSI/ISO-Bezug:** CON.8.A5, A.8.28

---

## Anhang C — Stack-spezifische Ergänzungen für MS Access / VBA *(Optional, bei Datenbank-Frontends)*

### C.1 VBA-Secure-Coding-Hinweise
- [ ] Keine hartcodierten Credentials in Makros oder Verbindungszeichenfolgen
- [ ] Parametrisierte Abfragen gegen SQL-Injection
- [ ] Eingabevalidierung in Formularen
- [ ] Keine unsicheren Dateioperationen (z. B. unsicherer `Kill`, `Shell`)
- [ ] Fehlerbehandlung ohne interne Details

### C.2 Access-Datenbank-Hardening
- Berechtigungskonzept auf Datenbankebene
- Verschlüsselung der Access-Datenbank (ACCDB mit Passwort)
- Verwendung von verknüpften Tabellen statt lokaler Datenhaltung
- Dokumentation aller Makros, Abhängigkeiten und externen Datenquellen

**BSI/ISO-Bezug:** APP.4.3 (Relationale Datenbanken), CON.8.A5, A.8.28

---

## Anhang D — Mapping: Kapitel → BSI / ISO-27001:2022 *(Pflicht)*

| Kapitel | BSI-Anforderungen | ISO-27001:2022-Kontrollen |
|---|---|---|
| 1. Metadaten | APP.7.A1 | A.5.37 |
| 2. Zweck, Daten & Schutzbedarf | APP.7.A6 | A.8.26, A.8.12 |
| 3. Sicherheitsprofil / Bedrohungsmodell | CON.8.A21, CON.8.A22, APP.7.A6 | A.8.27 |
| 4. Architektur & ADRs | CON.10.A11, CON.8.A12, CON.8.A22 | A.8.27 |
| 5. Datenflüsse & Schnittstellen | APP.7.A3, APP.3.1.A11 | A.5.14 |
| 6. Authentisierung & Autorisierung | CON.10.A1, APP.3.1.A1, ORP.4 | A.8.2, A.8.5 |
| 7. Berechtigungsmanagement | ORP.4 | A.5.18 |
| 8. Kryptografie & Schlüsselmanagement | CON.1 | A.8.24 |
| 9. Logging & Monitoring | CON.10.A13, APP.3.1.A22 | A.8.15, A.8.16 |
| 10. Konfiguration & Hardening | APP.3.1.A12, APP.3.1.A21, CON.8.A5 | A.8.27 |
| 11. Patch- und Schwachstellenmanagement | OPS.1.1.3.A15 | A.8.8 |
| 12. Änderungshistorie | OPS.1.1.3.A11 | A.8.32 |
| 13. Audit- und Testhistorie | APP.3.1.A22 | A.8.29, A.8.34 |
| 14. Betriebshandbuch | CON.8.A12 | A.5.37, A.5.29, A.5.30 |
| Anhang A (ASP.NET) | CON.10.A1–A16 | A.8.28 |
| Anhang B (Qt 6 C++) | CON.8.A5 | A.8.28 |
| Anhang C (MS Access / VBA) | APP.4.3, CON.8.A5 | A.8.28 |

---

## Review- und Pflegehinweise *(Pflicht)*

- Diese Anwendungsdokumentation muss **jährlich** oder bei **sicherheitsrelevanten Änderungen** reviewt und aktualisiert werden.
- Bei sicherheitsrelevanten Änderungen ist die [Pull-Request-Checkliste](../vorlagen/pr-checkliste.md) anzuwenden.
- Änderungen werden im Kapitel 12 (Changelog) nachvollziehbar dokumentiert.
- Freigabe und Review durch Anwendungsverantwortlichen und Secure-Coding-Beauftragten.
