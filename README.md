# Pendler-Radar

Pendler-Radar ist ein geplantes, gemeinschaftlich getragenes Projekt für aktuelle Hinweise im öffentlichen Nahverkehr. Die Idee ist von [FreiFahren](https://freifahren.org/) aus Berlin inspiriert – Pendler:innen sollen Beobachtungen schnell teilen und sich unterwegs besser orientieren können.

> **Wichtig:** Pendler-Radar ist ein unabhängiges Projekt und nicht mit FreiFahren, Verkehrsunternehmen oder Verkehrsverbünden verbunden. Hinweise in der Anwendung ersetzen kein gültiges Ticket und keine offiziellen Informationen des Verkehrsunternehmens.

## Vision

Öffentlicher Nahverkehr funktioniert besser, wenn Fahrgäste Informationen miteinander teilen können. Pendler-Radar soll eine einfache, schnelle und datensparsame Möglichkeit schaffen, aktuelle Beobachtungen zu melden und zeitlich begrenzt sichtbar zu machen.

Dabei gilt: Mobilität und Sicherheit stehen im Mittelpunkt. Die Anwendung soll nicht dazu auffordern, ohne Fahrschein zu fahren, und keine Personen identifizieren oder gefährden.

## Geplanter Funktionsumfang

### Für Fahrgäste

- aktuelle Meldungen als Liste und auf einer Karte anzeigen
- Meldungen nach Stadt, Linie, Verkehrsmittel und Zeitraum filtern
- Beobachtungen mit wenigen Angaben melden:
  - Stadt und Verkehrsnetz
  - Linie oder Verkehrsmittel
  - Haltestelle beziehungsweise Streckenabschnitt
  - Fahrtrichtung
  - Zeitpunkt der Beobachtung
- Alter und Aktualität einer Meldung auf einen Blick erkennen
- veraltete Meldungen automatisch ausblenden
- problematische oder falsche Inhalte melden

### Für eine verlässliche Community

- klare Regeln für sachliche Meldungen
- Moderations- und Meldefunktionen gegen Spam, Missbrauch und Diskriminierung
- Rate-Limits und Schutz vor automatisiertem Missbrauch
- keine Veröffentlichung von Namen, Fotos oder anderen personenbezogenen Daten
- möglichst wenige Daten sammeln und nur so lange wie nötig speichern
- barrierearme, mobile-first Oberfläche

## MVP-Ziele

Die erste Version soll sich auf einen kleinen, überprüfbaren Funktionsumfang konzentrieren:

1. Meldungen für eine ausgewählte Stadt erfassen
2. aktuelle Meldungen als chronologische Liste darstellen
3. nach Linie, Verkehrsmittel und Zeitraum filtern
4. Meldungen nach einer kurzen Zeit automatisch ablaufen lassen
5. Missbrauch melden und Inhalte moderieren können
6. Datenschutz- und Community-Regeln sichtbar und verständlich erklären

## Produktprinzipien

- **Aktualität:** Jede Meldung hat einen klaren Beobachtungszeitpunkt und verliert automatisch an Relevanz.
- **Sachlichkeit:** Veröffentlicht werden Beobachtungen, keine Verdächtigungen oder personenbezogenen Informationen.
- **Sicherheit:** Keine Hinweise auf einzelne Personen, keine Aufnahmen von Kontrollpersonal und keine Aufforderung zu gefährlichem Verhalten.
- **Datenschutz:** Keine unnötigen Accounts, Standortdaten oder Bewegungsprofile. Speicherung und Löschung werden transparent dokumentiert.
- **Transparenz:** Offizielle Meldungen des Verkehrsunternehmens bleiben die maßgebliche Quelle für Fahrplan-, Betriebs- und Sicherheitsinformationen.
- **Offenheit:** Die Anwendung soll nachvollziehbar, zugänglich und langfristig gemeinschaftlich weiterentwickelbar sein.

## Abgrenzung

Pendler-Radar ist **nicht**:

- ein Ersatz für ein gültiges ÖPNV-Ticket,
- eine offizielle Anwendung eines Verkehrsunternehmens,
- ein Personenverzeichnis oder ein Werkzeug zur Identifizierung von Kontrolleur:innen,
- eine Plattform für Fotos, Namen, private Daten oder gezielte Belästigung,
- eine Garantie dafür, dass eine Meldung noch aktuell oder vollständig ist.

## Geplante Entwicklung

### Phase 1 – Grundlage

- Anforderungen und Community-Regeln abstimmen
- Datenschutz- und Bedrohungsmodell definieren
- Datenmodell und API festlegen
- erste mobile Oberfläche entwickeln

### Phase 2 – MVP

- Meldungen erstellen und anzeigen
- Filter und automatische Ablaufzeiten
- Moderation, Rate-Limits und Missbrauchsschutz
- Tests mit einer einzelnen Stadt beziehungsweise einem Verkehrsnetz

### Phase 3 – Ausbau

- bessere Barrierefreiheit und Mehrsprachigkeit
- verifizierte Community-Moderation
- Auswertung der Datenqualität ohne Personenprofile zu erstellen
- Unterstützung weiterer Städte nach gemeinsamer Abstimmung

## Mitmachen

Das Projekt befindet sich in einer frühen Planungsphase. Ideen, UX-Vorschläge, Hinweise zu Datenschutz und Sicherheit sowie Beiträge zur Entwicklung sind willkommen.

Bitte eröffne für größere Vorschläge zunächst ein [Issue](https://github.com/nicolasb-wq/Pendler-Radar/issues). Beiträge sollten:

1. das Ziel der Änderung kurz erklären,
2. Datenschutz, Sicherheit und Barrierefreiheit berücksichtigen,
3. keine personenbezogenen Daten in Issues, Pull Requests oder Testdaten enthalten,
4. die bestehenden Community-Regeln einhalten.

## Lokale Entwicklung

Die technische Umsetzung ist noch nicht gestartet. Sobald das Projekt eine ausführbare Anwendung enthält, werden hier Installations- und Entwicklungsanweisungen ergänzt.

## Inspiration und Links

- [FreiFahren](https://freifahren.org/) – Inspiration für die Grundidee
- [FreiFahren FAQ](https://freifahren.org/faq/) – Informationen zum Vorbild
- [Issues](https://github.com/nicolasb-wq/Pendler-Radar/issues) – Fragen, Ideen und Diskussionen

## Lizenz

Die Lizenz wird festgelegt, sobald die erste technische Umsetzung vorliegt.
