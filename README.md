# Pendler-Radar

Eine selbst betreibbare Web-App für gemeinschaftliche Echtzeit-Hinweise im öffentlichen Nahverkehr.

Pendler-Radar ist von [FreiFahren](https://freifahren.org/) aus Berlin inspiriert: Fahrgäste können Beobachtungen zu Fahrkartenkontrollen melden und aktuelle Meldungen anderer Fahrgäste sehen. Die Anwendung soll schnell, mobil und datensparsam sein – ohne Personen zu identifizieren und ohne ein Bewegungsprofil ihrer Nutzer:innen aufzubauen.

> **Konzeptphase:** Dieses Repository enthält aktuell die Projektdokumentation. Die ausführbare Anwendung, die API und die Infrastruktur werden als Nächstes aufgebaut.
>
> Pendler-Radar ist ein unabhängiges Projekt und nicht mit FreiFahren, der BVG, der S-Bahn Berlin oder einem anderen Verkehrsunternehmen verbunden. Meldungen sind möglicherweise veraltet oder falsch. Sie ersetzen weder ein gültiges Ticket noch offizielle Informationen des Verkehrsunternehmens.

## Was entstehen soll

| Bereich | Ziel |
| --- | --- |
| Live-Karte | Aktuelle Meldungen auf einer Karte und in einer Liste anzeigen |
| Meldungen | Linie, Verkehrsmittel, Haltestelle oder Abschnitt, Richtung und Beobachtungszeit erfassen |
| Aktualität | Alter und Ablauf jeder Meldung sichtbar machen; veraltete Meldungen automatisch ausblenden |
| Städte | Verkehrsnetze über Konfiguration unterstützen, nicht im Programmcode fest verdrahten |
| Community | Falsche, doppelte oder problematische Meldungen melden und moderieren |
| Datenschutz | So wenig Daten wie möglich sammeln und keine Personen oder Bewegungsprofile verfolgen |
| Betrieb | Selbst hostbar, nachvollziehbar zu betreiben und ohne proprietäre Cloud-Abhängigkeit im Kern |

## Was die erste Version können muss

### Für Fahrgäste

- die Anwendung ohne langes Onboarding auf dem Smartphone öffnen
- aktuelle Meldungen als Liste und auf einer Karte sehen
- nach Stadt, Verkehrsnetz, Verkehrsmittel, Linie und Zeitraum filtern
- eine Meldung mit wenigen Pflichtangaben erstellen
- eine Meldung als veraltet, falsch oder unangemessen melden
- erkennen, wann eine Meldung beobachtet und wann sie veröffentlicht wurde
- auch bei langsamer Verbindung eine lesbare, funktionierende Oberfläche erhalten

### Für die Datenqualität

- Haltestellen, Linien und Richtungen aus einem kontrollierten Verkehrsnetz auswählen
- freie Texte nur dort zulassen, wo sie wirklich nötig sind
- Meldungen mit Zeitstempel, Ablaufzeit und Status speichern
- doppelte Meldungen erkennen und nicht ungebremst vervielfachen
- Spam, automatisierte Einsendungen und Missbrauch begrenzen
- Meldungen bei Unsicherheit zunächst zurückhalten können
- Moderationsentscheidungen nachvollziehbar protokollieren, ohne ein öffentliches Nutzerprofil anzulegen

### Später möglich

- Progressive Web App mit optionaler Installation auf dem Startbildschirm
- Unterstützung mehrerer Städte und Verkehrsverbünde
- Import aus einem Community-Kanal wie Telegram, mit derselben Validierung wie bei der Web-App
- optionale Push-Hinweise für frei wählbare Linien oder Gebiete
- anonymisierte Auswertung der Datenqualität
- zusätzliche Sprachversionen und verbesserte Barrierefreiheit

## Wie eine Meldung durch das System läuft

1. **Melden:** Eine Person wählt Verkehrsnetz, Linie, Ort und Richtung und gibt den Beobachtungszeitpunkt an.
2. **Prüfen:** Die API validiert die Eingaben, begrenzt die Häufigkeit und verwirft unmögliche oder unvollständige Zeitangaben.
3. **Speichern:** Die Meldung wird mit einer zufälligen Kennung, dem Beobachtungszeitpunkt und einer Ablaufzeit gespeichert.
4. **Veröffentlichen:** Eine gültige Meldung erscheint in der Liste und auf der Karte. Ihre Aktualität wird immer mit angezeigt.
5. **Ablaufen:** Nach der festgelegten Frist wird sie nicht mehr als aktuelle Meldung ausgeliefert. Alte Daten werden nach der Aufbewahrungsfrist gelöscht oder nur noch anonymisiert für die Qualitätssicherung verwendet.
6. **Moderieren:** Gemeldete Inhalte können vorübergehend quarantänisiert, geprüft und entfernt werden.

Wichtig ist die Trennung zwischen **Beobachtungszeitpunkt** und **Zeitpunkt der Einsendung**. Eine Meldung, die erst später abgeschickt wird, darf nicht so aussehen, als sei sie gerade erst beobachtet worden.

## Aktualität und Vertrauensmodell

Eine Live-Karte ist nur so gut wie ihre zeitliche Einordnung. Daher soll jede Meldung:

- den Zeitpunkt der Beobachtung und den Zeitpunkt der Veröffentlichung getrennt führen,
- einen sichtbaren Alterswert wie „vor 8 Minuten“ anzeigen,
- eine konfigurierbare Ablaufzeit besitzen,
- keine Zeitpunkte in der Zukunft akzeptieren,
- bei widersprüchlichen Meldungen nicht automatisch eine einzelne Wahrheit behaupten,
- nach Ablauf nicht mehr als aktuelle Warnung erscheinen.

Eine Meldung ist eine gemeinschaftliche Beobachtung, keine behördlich bestätigte Information. Die Oberfläche muss diese Unsicherheit deutlich machen und darf keine falsche Präzision vortäuschen.

## Geplantes Datenmodell

Die folgenden Felder beschreiben das Zielmodell; sie sind noch keine implementierte API:

| Feld | Zweck |
| --- | --- |
| `id` | Zufällige, nicht ableitbare Kennung der Meldung |
| `network_id` | Verkehrsnetz oder Stadt, zum Beispiel ein konfiguriertes Berliner Netz |
| `mode` | Verkehrsmittel, zum Beispiel U-Bahn, S-Bahn, Tram oder Bus |
| `line_id` | Kontrollierte Linienreferenz |
| `stop_id` / `segment_id` | Haltestelle oder Streckenabschnitt, niemals ein exakter Personenstandort |
| `direction` | Fahrtrichtung aus den Netzdaten |
| `observed_at` | Zeitpunkt, an dem die Beobachtung gemacht wurde |
| `created_at` | Zeitpunkt, an dem die Meldung beim Server eingegangen ist |
| `expires_at` | Zeitpunkt, ab dem sie nicht mehr als aktuell gilt |
| `status` | Zum Beispiel `published`, `expired`, `quarantined` oder `removed` |
| `source` | Technischer Eingang wie Web-App oder späterer Importkanal |

Nicht zum öffentlichen Meldungsmodell gehören Namen, Fotos, Audioaufnahmen, Telefonnummern, exakte GPS-Koordinaten oder die Identität des meldenden Menschen.

## Sicherheits- und Datenschutzgrenzen

### Was geschützt werden soll

- keine Kontenpflicht für das Lesen der öffentlichen Meldungen im MVP
- keine Veröffentlichung von IP-Adressen, Gerätekennungen oder Kontaktinformationen
- keine dauerhafte Speicherung von exakten Standortdaten
- keine Personenfotos, Namen oder Beschreibungen einzelner Kontrolleur:innen
- Rate-Limits gegen Spam und automatisiertes Einsenden
- serverseitige Validierung; der Client darf nicht als vertrauenswürdig gelten
- getrennte Rechte für öffentliche Nutzung, Moderation und Administration
- Protokolle ohne unnötige Meldungsinhalte und ohne dauerhafte Bewegungsprofile
- Löschfristen für technische Metadaten, Meldungen und Moderationsdaten

Technische Metadaten können für Missbrauchsschutz, Fehleranalyse und gesetzliche Pflichten erforderlich sein. Vor dem produktiven Betrieb werden Zweck, Rechtsgrundlage, Speicherdauer und Zugriff darauf in einem Datenschutzkonzept dokumentiert.

### Was ausdrücklich nicht geschützt werden kann

- die Richtigkeit einer nicht verifizierten Community-Meldung
- die Erreichbarkeit der Anwendung oder eines externen Kartenanbieters
- die Tatsache, dass andere Menschen eine öffentliche Meldung lesen können
- die Entscheidung von Verkehrsunternehmen oder Behörden
- eine rechtliche Bewertung des Fahrens ohne gültigen Fahrschein

Eine unabhängige Sicherheits- und Datenschutzprüfung hat noch nicht stattgefunden. Das Projekt darf erst nach einer solchen Prüfung und einer rechtlichen Bewertung öffentlich betrieben werden.

## Verantwortungsvolle Nutzung

Pendler-Radar soll Informationen über Beobachtungen zugänglich machen, nicht Menschen überwachen oder gefährden. Deshalb gelten für Beiträge mindestens diese Regeln:

- keine Namen, Fotos, Stimmen oder persönlichen Merkmale von Kontrollpersonal
- keine Aufforderung zu gefährlichem Verhalten, Gedränge oder Flucht
- keine gezielte Verfolgung einzelner Personen oder Fahrzeuge
- keine diskriminierenden, beleidigenden oder aufrufenden Inhalte
- keine fingierten Meldungen und keine automatisierte Manipulation der Karte
- bei Unsicherheit lieber keine Meldung veröffentlichen

Die Anwendung ist kein Ersatz für ein Ticket. Wer den ÖPNV nutzt, bleibt selbst für die erforderliche Fahrberechtigung verantwortlich.

## Zielarchitektur

Die Technologie ist noch nicht festgelegt. Die Architektur soll aus klar getrennten Komponenten bestehen:

| Komponente | Aufgabe |
| --- | --- |
| Web/PWA | Karte, Liste, Filter, Meldeformular und Moderationsoberfläche |
| API | Validierung, Meldungen, Netzdaten, Rate-Limits und öffentliche Abfragen |
| Datenbank | Meldungen, Verkehrsnetze, Ablaufstatus und Moderationsvorgänge |
| Ablaufdienst | Ablaufzeiten verarbeiten und alte Daten sicher entfernen |
| Netzdaten | Linien, Haltestellen, Abschnitte und Richtungen pro Verkehrsnetz |
| Importadapter | Optionaler, kontrollierter Eingang für Community-Kanäle |
| Beobachtung | Fehler, Latenzen, Datenbankzustand und fehlgeschlagene Jobs überwachen |

Die Stadt- und linienbezogenen Daten gehören in versionierte Konfiguration beziehungsweise importierte Datensätze. Fachlogik darf nicht auf Berlin oder einzelne Linien fest verdrahtet werden.

### Vorgesehener Repository-Aufbau

Die folgende Struktur beschreibt das Ziel, nicht den aktuellen Dateistand:

```text
apps/
  web/                    Web-App und PWA
services/
  api/                    HTTP-API und Authentifizierung für Moderation
  worker/                 Ablauf, Bereinigung und optionale Importe
packages/
  domain/                Meldungsregeln und gemeinsame Typen
  transit-data/          Linien, Haltestellen und Netzversionen
data/
  networks/              Versionierte Verkehrsnetze
  fixtures/              Kleine, synthetische Testdaten
  docs/
    DATENSCHUTZ-TECHNIK.md
    SICHERHEIT.md
    NUTZERANLEITUNG.md
    RUNBOOK.md
deploy/                   Container, Reverse-Proxy und Betrieb
 tests/                  Integrations- und End-to-End-Tests
 README.md
```

## Vorgeschlagene API

Auch diese Endpunkte sind zunächst ein Entwurf. Die öffentliche API soll klein bleiben und keine privaten Daten ausliefern.

| Methode | Endpunkt | Zweck |
| --- | --- | --- |
| `GET` | `/api/v1/networks` | Verfügbare Städte und Verkehrsnetze |
| `GET` | `/api/v1/networks/{id}/lines` | Linien, Haltestellen und Richtungen |
| `GET` | `/api/v1/reports` | Aktuelle, gefilterte Meldungen |
| `POST` | `/api/v1/reports` | Eine Meldung erstellen |
| `POST` | `/api/v1/reports/{id}/flag` | Eine Meldung zur Prüfung markieren |
| `GET` | `/api/v1/health` | Technischer Gesundheitsstatus für Monitoring |

Für `POST /reports` werden unter anderem Pflichtfelder, Zeitgrenzen, gültige Netzreferenzen und Rate-Limits geprüft. Ein erfolgreicher HTTP-Status bedeutet nur, dass die Meldung angenommen wurde – nicht, dass sie amtlich bestätigt ist.

## Betrieb

Das Ziel ist ein selbst betreibbarer Dienst. Für einen produktiven Betrieb sind mindestens erforderlich:

- HTTPS über einen Reverse-Proxy; die Anwendung darf nicht unverschlüsselt öffentlich erreichbar sein
- getrennte Umgebungen für Entwicklung, Tests und Produktion
- Geheimnisse ausschließlich über Umgebungsvariablen oder einen Secret Store
- Datenbankmigrationen mit sichtbarem Fehler bei einem inkompatiblen Schema
- automatische Sicherungen und regelmäßig getestete Wiederherstellung
- Überwachung von API, Datenbank, Ablaufdienst und Importen
- dokumentierte Aufbewahrungs- und Löschfristen
- ein Notfallweg zum Abschalten von Meldungsannahme oder öffentlicher Karte
- Runbook mit erwarteten Ergebnissen für Installation, Backup, Restore und Störung

Es wird keine Infrastruktur als vorhanden behauptet: Im aktuellen Repository gibt es weder Container noch eine laufende API oder Datenbank.

## Entwicklung und Tests

Die ausführbare Entwicklungsumgebung folgt mit der ersten Implementierung. Sie wird dann mindestens diese Prüfungen enthalten:

- **Domänentests:** Zeitvalidierung, Ablauf, Statusübergänge, Duplikaterkennung und Netzreferenzen
- **API-Tests:** Eingabevalidierung, Rate-Limits, Fehlerantworten und Berechtigungen
- **Datenbanktests:** Migrationen, Indizes, Löschfristen und konkurrierende Einsendungen
- **End-to-End-Tests:** Meldung erstellen, anzeigen, markieren und automatisch ausblenden
- **Sicherheitstests:** Keine privaten Felder in öffentlichen Antworten, kein Zugriff auf Moderationsfunktionen ohne Berechtigung
- **Barrierefreiheit:** Tastaturbedienung, Fokusführung, Kontrast, verständliche Fehlermeldungen und Screenreader-Beschriftungen
- **Betriebstests:** Ablaufdienst, Backup und Wiederherstellung

Tests sollen mit synthetischen Verkehrs- und Meldungsdaten laufen. Echte personenbezogene oder sensible Standortdaten gehören weder in Fixtures noch in Logs.

## Roadmap

### Phase 0 – Grundlage

- [x] Projektname und Zielrichtung festlegen
- [x] Anforderungen, Grenzen und Datenschutzprinzipien dokumentieren
- [ ] rechtliche und datenschutzrechtliche Prüfung vorbereiten
- [ ] Zielstadt und erstes Verkehrsnetz auswählen
- [ ] Technologie und Lizenz festlegen

### Phase 1 – Technischer Kern

- [ ] Repository-Struktur anlegen
- [ ] Verkehrsnetz als versionierten Datensatz einlesen
- [ ] Datenmodell und Migrationen implementieren
- [ ] API für Lesen und Erstellen von Meldungen bauen
- [ ] Ablaufzeiten und Bereinigung implementieren
- [ ] automatisierte Tests einrichten

### Phase 2 – Nutzbarer MVP

- [ ] mobile Liste und Karte
- [ ] Meldeformular mit kontrollierten Linien und Haltestellen
- [ ] Filter und sichtbares Alter jeder Meldung
- [ ] Melden-, Quarantäne- und Moderationsablauf
- [ ] Rate-Limits und Missbrauchsschutz
- [ ] erste selbst betriebene Testinstanz

### Phase 3 – Stabilisierung

- [ ] Barrierefreiheit prüfen
- [ ] Datenschutz- und Sicherheitsdokumentation vervollständigen
- [ ] Backup, Restore und Monitoring testen
- [ ] Last- und Ausfalltests durchführen
- [ ] weitere Städte erst nach Prüfung der Datenqualität hinzufügen

## Mitmachen

Das Projekt befindet sich am Anfang. Besonders hilfreich sind Beiträge zu:

- UX für eine schnelle Nutzung unterwegs
- Datenmodellen für verschiedene Verkehrsnetze
- Datenschutz, Threat Modeling und Missbrauchsschutz
- Barrierefreiheit
- Teststrategie und selbst betriebenem Deployment

Für größere Änderungen bitte zuerst ein [Issue](https://github.com/nicolasb-wq/Pendler-Radar/issues) anlegen. Pull Requests sollten:

1. das Problem und die beabsichtigte Änderung erklären,
2. Datenschutz, Sicherheit und Barrierefreiheit berücksichtigen,
3. keine echten personenbezogenen oder sensiblen Meldungsdaten enthalten,
4. Tests oder eine nachvollziehbare Begründung für fehlende Tests mitbringen,
5. keine Stadt- oder Linienlogik unnötig im Kern festschreiben.

## Inspiration und Abgrenzung

- [FreiFahren](https://freifahren.org/) – Inspiration für die Grundidee
- [FreiFahren auf GitHub](https://github.com/FreiFahren/FreiFahren) – öffentliches Vorbild
- [FreiFahren FAQ](https://freifahren.org/faq/) – Informationen zum Vorbild

Pendler-Radar übernimmt weder Code noch Marke oder Inhalte von FreiFahren. Die Projekte sind unabhängig voneinander.

## Lizenz

Die Lizenz wird vor Beginn der ersten technischen Umsetzung festgelegt. Bis dahin ist dieses Repository eine Projektskizze und keine veröffentlichte Software.
