# Pendler-Radar

Eine selbst betreibbare Web-App für gemeinschaftliche Echtzeit-Hinweise im öffentlichen Nahverkehr.

Pendler-Radar ist von [FreiFahren](https://freifahren.org/) aus Berlin inspiriert: Fahrgäste können Beobachtungen zu Fahrkartenkontrollen melden und aktuelle Meldungen anderer Fahrgäste sehen. Die Anwendung soll schnell, mobil, transparent und datensparsam sein – ohne Personen zu identifizieren und ohne ein Bewegungsprofil ihrer Nutzer:innen aufzubauen.

> **Konzeptphase:** Dieses Repository enthält aktuell die Projektdokumentation. Es gibt noch keine ausführbare Anwendung, keine API, keine Datenbank und keine produktive Instanz. Die technischen Entscheidungen und Beispiele in dieser README beschreiben das Zielbild und sind an den Stellen als Vorschlag gekennzeichnet.
>
> Pendler-Radar ist ein unabhängiges Projekt und nicht mit FreiFahren, der BVG, der S-Bahn Berlin oder einem anderen Verkehrsunternehmen verbunden. Meldungen sind möglicherweise veraltet oder falsch. Sie ersetzen weder ein gültiges Ticket noch offizielle Informationen des Verkehrsunternehmens.

---

## Inhaltsverzeichnis

- [Projektstatus](#projektstatus)
- [Was entstehen soll](#was-entstehen-soll)
- [Ziele und Nichtziele](#ziele-und-nichtziele)
- [Funktionsumfang](#funktionsumfang)
- [Wie eine Meldung durch das System läuft](#wie-eine-meldung-durch-das-system-läuft)
- [Aktualität und Vertrauensmodell](#aktualität-und-vertrauensmodell)
- [Datenmodell](#datenmodell)
- [Benutzeroberfläche](#benutzeroberfläche)
- [Moderation und Community-Regeln](#moderation-und-community-regeln)
- [Sicherheit und Datenschutz](#sicherheit-und-datenschutz)
- [Technische Zielarchitektur](#technische-zielarchitektur)
- [API-Entwurf](#api-entwurf)
- [Vorgesehener Repository-Aufbau](#vorgesehener-repository-aufbau)
- [Betrieb](#betrieb)
- [Entwicklung und Tests](#entwicklung-und-tests)
- [Abnahmekriterien für den MVP](#abnahmekriterien-für-den-mvp)
- [Roadmap](#roadmap)
- [Dokumentation, die noch entsteht](#dokumentation-die-noch-entsteht)
- [Mitmachen](#mitmachen)
- [Inspiration und Abgrenzung](#inspiration-und-abgrenzung)
- [Lizenz](#lizenz)

---

## Projektstatus

| Bereich | Status |
| --- | --- |
| Projektbeschreibung | dokumentiert |
| Anforderungen und Grenzen | erster Entwurf |
| Umsetzungsplan | überarbeitet, siehe [`docs/UMSETZUNGSPLAN.md`](docs/UMSETZUNGSPLAN.md) |
| Web-App | noch nicht begonnen |
| API | noch nicht begonnen |
| Datenbank | noch nicht begonnen |
| Verkehrsnetz-Daten | noch nicht ausgewählt/importiert |
| Moderationssystem | konzeptionell beschrieben |
| Datenschutzkonzept | muss erstellt und geprüft werden |
| Sicherheitsprüfung | nicht durchgeführt |
| Deployment | nicht vorhanden |
| Produktive Instanz | nicht vorhanden |
| Lizenz | noch nicht festgelegt |

Diese README beschreibt also kein bereits fertiges Produkt. Sie ist gleichzeitig Produktbeschreibung, technische Spezifikation und Arbeitsgrundlage für die Implementierung. Sobald Code entsteht, werden Aussagen über tatsächlich vorhandene Funktionen hier von Vorschlägen und offenen Entscheidungen getrennt.

---

## Was entstehen soll

Pendler-Radar soll eine fokussierte Community-Plattform für aktuelle Beobachtungen im ÖPNV werden:

1. Eine Person sieht eine Kontrolle oder eine andere relevante Beobachtung.
2. Sie meldet diese mit Linie, Ort, Richtung und Beobachtungszeit.
3. Der Server validiert die Eingabe, begrenzt Missbrauch und speichert die Meldung.
4. Andere Fahrgäste sehen die Meldung in einer Liste und auf einer Karte.
5. Die Meldung verliert nach einer kurzen, sichtbaren Frist ihre Aktualität und wird aus der öffentlichen Ansicht entfernt.
6. Die Community kann falsche, doppelte oder problematische Meldungen markieren.

Der Dienst soll nicht mehr Daten sammeln als für diesen Ablauf erforderlich sind. Eine Meldung beschreibt einen Ort im Verkehrsnetz – nicht den exakten Standort einer Person. Öffentliche Meldungen erhalten keine Namen, Fotos, Telefonnummern oder Gerätekennungen.

### Grundsätze

- **Aktuell statt archiviert:** Eine alte Meldung darf nicht wie eine aktuelle Warnung aussehen.
- **ÖPNV-orientiert statt GPS-orientiert:** Linien, Haltestellen und Streckenabschnitte sind die primären Orte.
- **Sachlich statt personenbezogen:** Beobachtungen ja, Identifizierung und Belästigung nein.
- **Datensparsam statt profilbildend:** Keine unnötigen Konten, Bewegungsverläufe oder dauerhaften Identifikatoren.
- **Selbst betreibbar statt undurchsichtig:** Installation, Datenflüsse, Löschung und Backups sollen dokumentiert sein.
- **Ehrlich statt überversprechend:** Nicht implementierte oder ungeprüfte Eigenschaften werden nicht als fertig dargestellt.

---

## Ziele und Nichtziele

### Ziele

Der MVP soll:

- eine ausgewählte Stadt beziehungsweise ein ausgewähltes Verkehrsnetz abbilden,
- aktuelle Meldungen in einer mobilen Web-Oberfläche anzeigen,
- neue Meldungen ohne unnötig kompliziertes Onboarding entgegennehmen,
- Linien, Haltestellen und Richtungen aus kontrollierten Netzdaten anbieten,
- Zeit und Alter jeder Meldung sichtbar machen,
- veraltete Meldungen automatisch ausblenden,
- Missbrauch durch Rate-Limits, Validierung und Moderation erschweren,
- ohne personenbezogene öffentliche Profile funktionieren,
- mit synthetischen Daten automatisiert testbar sein,
- von einer einzelnen Person oder kleinen Gruppe selbst betrieben werden können.

### Nichtziele

Pendler-Radar ist ausdrücklich nicht:

- ein Ersatz für ein gültiges ÖPNV-Ticket,
- eine offizielle Anwendung eines Verkehrsunternehmens,
- eine amtliche oder garantierte Echtzeitquelle,
- ein Personenverzeichnis für Kontrolleur:innen,
- ein Werkzeug zur Identifizierung, Verfolgung oder gezielten Belästigung von Menschen,
- eine Plattform zum Hochladen von Fotos, Videos oder Tonaufnahmen einzelner Personen,
- ein allgemeines soziales Netzwerk,
- ein Routenplaner, der automatisch die Umgehung von Kontrollen empfiehlt,
- ein System zum Erstellen dauerhafter Bewegungsprofile,
- eine Garantie, dass eine veröffentlichte Meldung noch stimmt.

---

## Funktionsumfang

### Für Fahrgäste

- die Anwendung direkt auf dem Smartphone öffnen
- aktuelle Meldungen als chronologische Liste sehen
- aktuelle Meldungen auf einer Karte sehen
- nach Stadt, Verkehrsnetz, Verkehrsmittel, Linie und Zeitraum filtern
- die Entfernung zur Aktualität einer Meldung erkennen, zum Beispiel „vor 8 Minuten“
- eine Meldung mit wenigen Pflichtangaben erstellen
- einen Ort aus dem Verkehrsnetz auswählen statt beliebige private Ortsdaten einzugeben
- eine Meldung als veraltet, falsch, doppelt oder unangemessen markieren
- auch ohne öffentliches Nutzerprofil Meldungen lesen
- verständliche Fehlermeldungen bekommen, wenn eine Meldung nicht angenommen wird

### Für die Datenqualität

- Haltestellen, Linien und Richtungen aus einem kontrollierten Verkehrsnetz auswählen
- freie Texte nur dort zulassen, wo sie wirklich nötig sind
- Meldungen mit Beobachtungszeit, Eingangszeit und Ablaufzeit speichern
- doppelte Meldungen erkennen und nicht ungebremst vervielfachen
- Meldungen bei Unsicherheit zunächst unter Quarantäne stellen können
- Spam, automatisierte Einsendungen und missbräuchliche Clients begrenzen
- Moderationsentscheidungen nachvollziehbar protokollieren
- keine öffentliche Historie einzelner meldender Personen führen

### Später möglich, aber nicht Teil des ersten MVP

- Progressive Web App mit optionaler Installation auf dem Startbildschirm
- Unterstützung mehrerer Städte und Verkehrsverbünde
- Import aus einem Community-Kanal wie Telegram, mit exakt derselben Validierung wie bei der Web-App
- optionale Push-Hinweise für frei gewählte Linien oder Gebiete
- anonymisierte Auswertung der Datenqualität
- zusätzliche Sprachversionen
- Nutzerkonto nur für freiwillige Einstellungen oder Moderation
- native Apps, falls eine PWA die Anforderungen nicht erfüllt

### Bewusst nicht im ersten MVP

- kein eigenes Bezahlsystem
- keine Werbung und kein Verkauf von Standort- oder Nutzungsdaten
- keine frei hochladbaren Medien
- keine öffentliche Rangliste von Meldenden
- keine automatische Zuschreibung einer Meldung zu einer Person
- keine Live-Verfolgung einzelner Züge, Fahrzeuge oder Menschen
- kein automatischer Import ungeprüfter Chat-Nachrichten

---

## Wie eine Meldung durch das System läuft

### 1. Erstellen

Die meldende Person wählt mindestens:

- Stadt oder Verkehrsnetz,
- Verkehrsmittel,
- Linie,
- Haltestelle oder Streckenabschnitt,
- Fahrtrichtung,
- Zeitpunkt der Beobachtung.

Optional kann die Person eine kurze sachliche Ergänzung angeben. Freie Texte werden begrenzt, bereinigt und dürfen keine Namen, Kontaktdaten, Fotos oder anderen personenbezogenen Angaben enthalten.

### 2. Validieren

Die API prüft unter anderem:

- ob das Verkehrsnetz aktiv ist,
- ob Linie und Ort zu diesem Netz gehören,
- ob die Richtung für die Linie gültig ist,
- ob der Beobachtungszeitpunkt nicht in der Zukunft liegt,
- ob der Zeitpunkt nicht außerhalb eines zulässigen Rückblickfensters liegt,
- ob die Meldung die Längen- und Inhaltsgrenzen einhält,
- ob die Einsenderate zulässig ist,
- ob ein offensichtliches Duplikat bereits existiert.

Der Browser ist nicht vertrauenswürdig. Alle Prüfungen müssen serverseitig wiederholt werden.

### 3. Speichern

Eine gültige Meldung erhält:

- eine zufällige, nicht fortlaufende Kennung,
- `observed_at`: den Zeitpunkt der Beobachtung,
- `created_at`: den Eingang beim Server,
- `expires_at`: den Zeitpunkt, an dem sie nicht mehr aktuell ist,
- einen öffentlichen Veröffentlichungsstatus.

Die Anwendung speichert keine GPS-Koordinate der meldenden Person. Ein Verkehrsnetz-Ort wie „U2 – Alexanderplatz“ ist eine Referenz auf Netzdaten und kein persönlicher Standortnachweis.

### 4. Veröffentlichen

Eine akzeptierte Meldung wird in der öffentlichen Liste und auf der Karte angezeigt. Die Oberfläche zeigt immer das Alter der Meldung. Die Sortierung erfolgt grundsätzlich nach dem Beobachtungszeitpunkt, nicht nur nach dem Zeitpunkt des Uploads.

### 5. Ablaufen

Nach `expires_at` wird eine Meldung nicht mehr als aktuelle Meldung ausgeliefert. Ein Hintergrunddienst verarbeitet den Ablauf zuverlässig; die API prüft zusätzlich beim Lesen, ob eine Meldung bereits abgelaufen ist. Dadurch bleibt eine ausgefallene Bereinigung nicht unbemerkt.

### 6. Markieren und moderieren

Eine Person kann eine Meldung markieren. Je nach Regelwerk wird sie:

- nur zur späteren Prüfung vorgemerkt,
- vorübergehend aus der öffentlichen Ansicht genommen,
- endgültig entfernt,
- oder nach Prüfung wieder veröffentlicht.

Eine Meldung darf nicht allein wegen vieler Markierungen automatisch als wahr oder falsch gelten. Markierungen sind ein Signal für Moderation, keine Abstimmung über Fakten.

---

## Aktualität und Vertrauensmodell

Eine Live-Karte ist nur so gut wie ihre zeitliche Einordnung. Daher gelten folgende Regeln:

- **Beobachtungszeitpunkt und Eingangszeitpunkt sind getrennt.** Eine verspätet gesendete Meldung darf nicht frisch wirken.
- **Das Alter ist sichtbar.** „Gerade eben“ ohne konkrete Zeitspanne reicht nicht.
- **Ablauf ist konfigurierbar.** Die Frist hängt vom Verkehrsnetz und der Produktentscheidung ab und wird nicht unbemerkt verlängert.
- **Zukunftszeiten sind ungültig.** Ein kleiner tolerierter Uhrversatz kann technisch berücksichtigt werden, muss aber dokumentiert sein.
- **Alte Meldungen werden nicht als aktuelle Warnungen wiederverwendet.**
- **Widersprüche werden nicht automatisch geglättet.** Die Anwendung behauptet nicht, aus zwei Meldungen eine amtliche Wahrheit ableiten zu können.
- **Quelle und Qualität sind sichtbar.** Ein späterer Telegram-Import wird nicht vertrauenswürdiger als eine Meldung aus der Web-App, nur weil er von einem anderen Kanal kommt.

### Vorgeschlagenes Standardverhalten

Die folgenden Werte sind Produktvorschläge und werden vor der Implementierung festgelegt:

| Zustand | Öffentliche Darstellung |
| --- | --- |
| Beobachtung sehr frisch | normale Meldung mit sichtbarem Alter |
| Beobachtung älter, aber noch innerhalb der Frist | abgeschwächte Darstellung mit Warnung zur Aktualität |
| Ablauf erreicht | nicht mehr in der aktuellen Liste und nicht mehr auf der aktuellen Karte |
| als problematisch markiert | je nach Moderationsregel sichtbar oder vorübergehend verborgen |
| entfernt | öffentlich nicht mehr abrufbar |

---

## Datenmodell

Die folgende Struktur beschreibt das Zielmodell. Sie ist noch keine implementierte Datenbankmigration.

### `networks`

Konfigurierte Verkehrsnetze oder Städte.

- `id`
- `slug`
- `name`
- `timezone`
- `status`
- `data_version`
- `created_at`
- `updated_at`

### `lines`

Linien innerhalb eines Netzes.

- `id`
- `network_id`
- `code`, zum Beispiel `U2` oder `M10`
- `mode`, zum Beispiel `subway`, `tram`, `bus` oder `rail`
- `name`
- `active`

### `stops` und `segments`

Öffentliche Verkehrsorte beziehungsweise Streckenabschnitte.

- `id`
- `network_id`
- `external_id`
- `name`
- `latitude` und `longitude` nur als öffentliche Netzdaten, nicht als Nutzerstandort
- `geometry` oder Linienzuordnung, falls für die Karte erforderlich
- `data_version`

Exakte GPS-Punkte von meldenden Personen gehören nicht in diese Tabellen.

### `reports`

Die eigentlichen Meldungen.

| Feld | Zweck |
| --- | --- |
| `id` | zufällige Kennung |
| `network_id` | Verkehrsnetz |
| `line_id` | Linie |
| `stop_id` oder `segment_id` | Haltestelle beziehungsweise Abschnitt |
| `direction` | kontrollierte Fahrtrichtung |
| `observed_at` | Zeitpunkt der Beobachtung in UTC gespeichert |
| `created_at` | Zeitpunkt des Eingangs in UTC gespeichert |
| `expires_at` | Ablaufzeit in UTC gespeichert |
| `status` | `received`, `published`, `expired`, `quarantined` oder `removed` |
| `source` | zum Beispiel `web` oder später `telegram_import` |
| `note` | optionaler, bereinigter Kurztext |
| `created_by` | nur falls später ein freiwilliges Konto eingeführt wird; nicht öffentlich |

### `report_flags`

Meldungen über problematische Beiträge.

- zufällige Kennung
- Referenz auf die Meldung
- Kategorie, zum Beispiel `stale`, `duplicate`, `abuse` oder `privacy`
- Eingang und Bearbeitungsstatus
- notwendige technische Metadaten mit eigener Löschfrist

### `moderation_actions`

Nachvollziehbare, zugriffsgeschützte Aktionen der Moderation.

- Meldungsreferenz
- Aktion
- Grundkategorie
- Zeitpunkt
- Moderationskonto oder technische Prozesskennung
- keine unnötige Kopie des vollständigen Meldungstextes

### Indizes und Konsistenz

Vorgesehene Prüfungen und Indizes:

- `reports(network_id, expires_at)` für die aktuelle Ansicht
- `reports(line_id, observed_at)` für Linienfilter
- `reports(status, expires_at)` für Bereinigung
- Fremdschlüssel zwischen Netz, Linie, Haltestelle und Meldung
- Datenbank-Constraint für gültige Statusübergänge, soweit möglich
- Schutz gegen identische Meldungskennungen
- Transaktion beim Erstellen und beim Wechsel in die Quarantäne

---

## Benutzeroberfläche

### Öffentliche Startseite

Die Startseite soll ohne Konto verständlich sein und direkt die aktuelle Ansicht zeigen:

- ausgewähltes Verkehrsnetz
- Alter der Daten
- Liste aktueller Meldungen
- Kartenansicht als Ergänzung, nicht als einzige Informationsquelle
- Filter nach Linie, Verkehrsmittel, Richtung und Zeitraum
- deutlicher Hinweis, dass Meldungen unbestätigte Community-Beobachtungen sind
- gut erreichbarer Button zum Erstellen einer Meldung

### Meldeformular

Das Formular soll eine Meldung in wenigen Schritten ermöglichen, aber nicht auf Kosten der Datenqualität:

1. Verkehrsmittel auswählen
2. Linie auswählen
3. Haltestelle oder Abschnitt auswählen
4. Richtung auswählen
5. Beobachtungszeit prüfen
6. optional kurzen sachlichen Hinweis ergänzen
7. Community- und Datenschutzregeln bestätigen
8. Meldung absenden

Die Pflichtfelder müssen auch mit Tastatur und Screenreader bedienbar sein. Fehler werden am Feld und in einer zusammenfassenden Fehlermeldung angezeigt.

### Kartenansicht

Die Karte zeigt ausschließlich Meldungen, die noch innerhalb ihrer Veröffentlichungsfrist liegen. Vorgesehene Regeln:

- keine exakten Nutzerstandorte
- Cluster bei vielen Meldungen im gleichen Gebiet
- sichtbare Legende und verständliche Symbole
- Liste als gleichwertige Alternative für Screenreader und schlechte Verbindungen
- Kartenanbieter austauschbar
- OpenStreetMap- und Kartenanbieter-Lizenzen beachten
- keine stillschweigende Weitergabe persönlicher Standortdaten an einen Kartenanbieter

### Moderationsansicht

Die Moderationsansicht ist nicht öffentlich. Sie soll mindestens können:

- markierte Meldungen nach Kategorie und Alter sortieren
- Meldung, Kontext und technische Prüfinformationen getrennt anzeigen
- veröffentlichen, quarantänisieren oder entfernen
- eine interne Kategorie statt eines freien, personenbezogenen Kommentars speichern
- Entscheidungen nachvollziehbar protokollieren
- Moderationszugriff einzeln widerrufen

---

## Moderation und Community-Regeln

### Verbotene Inhalte

Nicht veröffentlicht werden:

- Namen, Fotos, Videos oder Tonaufnahmen von Kontrollpersonal
- private Kontaktdaten oder andere personenbezogene Daten
- Spekulationen über Identität, Herkunft oder private Lebensumstände
- diskriminierende, beleidigende oder bedrohliche Texte
- Aufrufe zu Gewalt, Flucht oder gefährlichem Verhalten
- bewusst falsche oder automatisiert erzeugte Meldungsserien
- Werbung, Spam und Links ohne Bezug zum Verkehrsnetz

### Moderationsstatus

```text
received -> published -> expired
     |          |
     v          v
quarantined -> removed
     |
     +-------> published
```

Nicht jeder Übergang muss öffentlich sichtbar sein. Für Nutzer:innen ist wichtig, dass eine entfernte Meldung nicht weiter auf der Karte erscheint und dass eine Quarantäne nicht als bestätigte Meldung behandelt wird.

### Missbrauchsschutz

Der erste Entwurf sieht mehrere unabhängige Grenzen vor:

- Limit pro technischer Quelle und Zeitfenster
- Limit pro Netz und Meldungstyp
- Erkennung identischer oder nahezu identischer Einsendungen
- Begrenzung von Textlänge und Sonderzeichen
- optionale zusätzliche Challenge bei auffälligem Verhalten
- Quarantäne bei ungewöhnlich hoher Meldungsrate
- Monitoring von Fehlerraten und ungewöhnlichen Spitzen

Rate-Limits dürfen nicht dazu führen, dass die öffentliche Karte unnötig personenbezogene Identifikatoren langfristig speichert. Die konkrete technische Umsetzung wird im Datenschutz- und Sicherheitskonzept festgehalten.

---

## Sicherheit und Datenschutz

### Vertrauensgrenzen

| Grenze | Annahme |
| --- | --- |
| Browser zu API | Der Browser kann manipuliert werden; jede Eingabe wird serverseitig geprüft. |
| Öffentliche API zu Datenbank | Nur die API und Hintergrunddienste erhalten Datenbankzugriff. |
| Moderation zu öffentlicher Ansicht | Moderationsfunktionen brauchen getrennte Berechtigungen. |
| Importkanal zu API | Externe Nachrichten sind unzuverlässig und müssen denselben Regeln wie Web-Meldungen folgen. |
| API zu Kartenanbieter | Nur notwendige öffentliche Kartendaten verwenden; keine Nutzerposition übertragen. |
| Anwendung zu Logs | Logs dürfen keine vollständigen sensiblen Eingaben enthalten. |

### Bedrohungen und Gegenmaßnahmen

| Bedrohung | Gegenmaßnahme |
| --- | --- |
| Spam und Meldungsflut | Rate-Limits, Größenlimits, Deduplizierung und Quarantäne |
| SQL- oder Script-Injection | parametrisierte Datenbankzugriffe, Output-Encoding, Content-Security-Policy und Tests |
| gefälschte oder unmögliche Zeiten | serverseitige Zeitprüfung und UTC-Speicherung |
| Zugriff auf Moderationsfunktionen | getrennte Rollen, sichere Sitzungen, 2FA für Admins als Zielanforderung |
| Scraping und Überlastung | Cache für öffentliche Netzdaten, API-Limits, Monitoring und Notfallabschaltung |
| Datenbankleck | minimale Datenspeicherung, Verschlüsselung bei Transport und Speicherung, restriktive Datenbankrechte |
| Import-Spoofing | verifizierter Adapter, Signatur oder kontrollierte Bot-Anbindung, nie blindes Vertrauen in Chattext |
| Kartenanbieter erhält private Daten | nur öffentliche Netzdaten verwenden; keine Browser-Geolocation standardmäßig aktivieren |
| Wiederherstellung aus alten Backups | Löschfristen und Backups gemeinsam bewerten; gelöschte sensible Daten nicht unbegrenzt vorhalten |

### Datenschutzprinzipien

- Lesen öffentlicher Meldungen ohne Konto im MVP
- keine Pflicht zur Angabe von Name, E-Mail oder Telefonnummer für das Lesen
- keine Veröffentlichung von IP-Adresse, Geräte-ID oder Kontaktinformationen
- keine Browser-Geolocation als Voraussetzung für Meldungen
- keine exakten Meldestandorte außerhalb kontrollierter Verkehrsnetzdaten
- technische Metadaten nur für einen dokumentierten Zweck
- getrennte Aufbewahrungsfristen für aktuelle Meldungen, Moderation, Logs und Backups
- Auskunfts-, Lösch- und Kontaktweg vor dem öffentlichen Betrieb dokumentieren
- Drittanbieter wie Karten-, Monitoring- oder Push-Dienste in den Datenfluss aufnehmen
- Datenschutz- und Sicherheitsdokumente vor dem Launch gegen die tatsächliche Implementierung prüfen

### Vorgeschlagene Aufbewahrung

Die Werte sind Vorschläge und keine rechtliche Festlegung:

| Datenart | Ziel |
| --- | --- |
| aktuelle Meldung | nur bis zum Ablauf öffentlich ausliefern |
| abgelaufene Rohmeldung | kurze technische Aufbewahrung für Duplikat- und Missbrauchsprüfung, danach löschen |
| Moderationsentscheidung | so lange wie für Nachvollziehbarkeit und Regelumsetzung erforderlich |
| Rate-Limit-Daten | kurz und zweckgebunden |
| Fehlerlogs | minimiert und mit eigener Löschfrist |
| Backups | verschlüsselt, begrenzt und mit dokumentiertem Löschkonzept |

„DSGVO-konform“ wird nicht als Marketingbehauptung verwendet, solange die konkrete Verarbeitung, Rechtsgrundlage, Auftragsverarbeitung und Löschung nicht geprüft wurden.

### Was ausdrücklich nicht geschützt werden kann

- die Richtigkeit einer nicht verifizierten Community-Meldung
- die Erreichbarkeit der Anwendung oder eines externen Kartenanbieters
- die Tatsache, dass andere Menschen eine öffentliche Meldung lesen können
- Entscheidungen von Verkehrsunternehmen oder Behörden
- eine rechtliche Bewertung des Fahrens ohne gültigen Fahrschein
- ein vollständiger Schutz vor koordinierter Falschinformation trotz Moderation

Eine unabhängige Sicherheits- und Datenschutzprüfung hat noch nicht stattgefunden. Dieses Dokument ist kein Audit.

---

## Technische Zielarchitektur

Die Architektur ist noch nicht implementiert. Das Zielbild besteht aus wenigen, klar getrennten Komponenten:

```text
                    +----------------------+
                    | Browser / PWA        |
                    | Liste, Karte, Form   |
                    +----------+-----------+
                               |
                         HTTPS / JSON
                               |
                    +----------v-----------+
                    | Public API           |
                    | Validierung          |
                    | Rate-Limits          |
                    | öffentliche Abfragen |
                    +----+-------------+---+
                         |             |
                    +----v----+   +----v-----+
                    | Datenbank|   | Worker    |
                    | Meldungen|   | Ablauf    |
                    | Netzdaten|   | Bereinigung|
                    +---------+   +----------+
                         |
                    +----v-------------+
                    | versionierte     |
                    | Verkehrsnetzdaten|
                    +------------------+
```

### Komponenten

| Komponente | Aufgabe |
| --- | --- |
| Web/PWA | Karte, Liste, Filter, Meldeformular und verständliche Zustände |
| API | Validierung, Meldungen, Netzdaten, Rate-Limits und öffentliche Abfragen |
| Datenbank | Meldungen, Verkehrsnetze, Status und Moderationsvorgänge |
| Worker | Ablaufzeiten, Bereinigung, Aggregationen und optionale Importe |
| Netzdaten | Linien, Haltestellen, Abschnitte, Richtungen und Versionen |
| Monitoring | Fehler, Latenzen, Datenbankzustand und fehlgeschlagene Jobs |
| Reverse-Proxy | TLS, Kompression, Header und Begrenzung öffentlicher Zugriffe |

### Vorgeschlagener Startstack

Noch nicht festgelegt, aber für den MVP naheliegend:

- TypeScript für gemeinsame Typen und Serverlogik
- React oder eine vergleichbare PWA-Oberfläche
- HTTP-API mit klar versionierten JSON-Endpunkten
- PostgreSQL für relationale Daten und saubere Migrationen
- Docker Compose für lokale Entwicklung und kleine selbst betriebene Installationen
- OpenStreetMap-kompatible Kartendarstellung mit dokumentierter Attribution
- GTFS oder ein vergleichbares, lizenziertes Format für Verkehrsnetzdaten

Das ist eine technische Empfehlung, keine Behauptung über bereits vorhandenen Code. Die endgültige Entscheidung sollte vor dem ersten Implementierungsschritt als ADR dokumentiert werden.

### Stadt- und Netzneutralität

Berlin darf nicht als Sonderfall in der Fachlogik auftauchen. Stadtbezogene Informationen gehören in versionierte Netzdaten:

- Linien und Verkehrsmittel
- Haltestellen und Abschnitte
- Richtungen
- Zeitzone
- Datenquelle und Lizenz
- Importdatum und Datensatzversion
- Regeln für Ablauf und Darstellung, falls ein Netz sie benötigt

Die Fachlogik arbeitet gegen ein einheitliches Modell. Dadurch kann später eine zweite Stadt hinzukommen, ohne die Meldungs- oder Moderationslogik zu duplizieren.

---

## API-Entwurf

Diese Endpunkte sind ein Entwurf und noch nicht erreichbar. Die API wird versioniert, damit sich die öffentliche Oberfläche nicht bei jeder internen Änderung ändert.

| Methode | Endpunkt | Zweck |
| --- | --- | --- |
| `GET` | `/api/v1/health` | technischer Gesundheitsstatus für Monitoring |
| `GET` | `/api/v1/networks` | verfügbare Städte und Verkehrsnetze |
| `GET` | `/api/v1/networks/{id}` | Metadaten und Datenversion eines Netzes |
| `GET` | `/api/v1/networks/{id}/lines` | Linien, Haltestellen und Richtungen |
| `GET` | `/api/v1/reports` | aktuelle, gefilterte Meldungen |
| `POST` | `/api/v1/reports` | eine neue Meldung erstellen |
| `POST` | `/api/v1/reports/{id}/flag` | eine Meldung zur Prüfung markieren |
| `GET` | `/api/v1/meta` | API-Version und öffentliche Betriebsinformationen |

### Beispiel: Meldung anlegen

Vorgesehene Struktur, noch nicht implementiert:

```json
{
  "networkId": "berlin",
  "mode": "subway",
  "lineId": "u2",
  "stopId": "alexanderplatz",
  "direction": "pankow",
  "observedAt": "2026-09-17T12:34:00Z",
  "note": "Sachliche optionale Ergänzung"
}
```

Die API darf niemals blind Felder aus diesem JSON übernehmen. Sie löst IDs gegen die aktuelle Netzversion auf, prüft Status und Zeit und entfernt oder verweigert nicht zulässige Inhalte.

### Beispiel: öffentliche Antwort

```json
{
  "id": "rpt_7f3b1e...",
  "networkId": "berlin",
  "mode": "subway",
  "line": "U2",
  "stop": "Alexanderplatz",
  "direction": "Pankow",
  "observedAt": "2026-09-17T12:34:00Z",
  "createdAt": "2026-09-17T12:35:12Z",
  "expiresAt": "2026-09-17T14:04:00Z",
  "status": "published"
}
```

Öffentliche Antworten enthalten keine IP-Adresse, Gerätekennung, Moderationsnotiz, interne Benutzer-ID oder vollständige technische Request-Daten.

### Fehlerformat

Alle fachlichen Fehler sollen strukturiert und maschinenlesbar sein:

```json
{
  "error": {
    "code": "REPORT_TIME_IN_FUTURE",
    "message": "Der Beobachtungszeitpunkt liegt in der Zukunft.",
    "fields": ["observedAt"]
  }
}
```

Die Benutzeroberfläche darf die `code`-Werte für konkrete Übersetzungen verwenden. Interne Stacktraces und Datenbankfehler gehören nicht in öffentliche Antworten.

### Öffentliche Daten und Cache

Netzdaten und aktuelle Meldungen können für kurze Zeit zwischengespeichert werden. Cache-Header dürfen aber nicht dazu führen, dass abgelaufene Meldungen dauerhaft in einer öffentlichen Ansicht bleiben. Die Ablaufzeit einer Meldung muss bei der Auslieferung und bei der Cache-Konfiguration berücksichtigt werden.

---

## Vorgesehener Repository-Aufbau

Die folgende Struktur beschreibt das Ziel, nicht den aktuellen Dateistand:

```text
apps/
  web/                         Web-App und PWA
services/
  api/                         HTTP-API und Moderationszugriff
  worker/                      Ablauf, Bereinigung und optionale Importe
packages/
  domain/                      Meldungsregeln und gemeinsame Typen
  transit-data/                Linien, Haltestellen und Netzversionen
  api-client/                  typisierter Client für die Web-App
data/
  networks/                    versionierte Verkehrsnetze
  fixtures/                    kleine, synthetische Testdaten
docs/
  SICHERHEIT.md                Bedrohungsmodell und Restrisiken
  DATENSCHUTZ-TECHNIK.md       Speicherorte, Flüsse und Löschung
  NUTZERANLEITUNG.md           Nutzung der Web-App
  MODERATION.md                Regeln und Ablauf für Moderation
  RUNBOOK.md                   Installation, Betrieb, Backup und Restore
deploy/
  compose.yaml                 lokaler und kleiner selbst betriebener Stack
  reverse-proxy/               Beispiele für TLS und Weiterleitung
tests/
  integration/                 API und Datenbank
  e2e/                         Browserabläufe
README.md
```

Die Zielstruktur ist noch nicht angelegt. Aktuell vorhanden sind diese README und der [überarbeitete Umsetzungsplan](docs/UMSETZUNGSPLAN.md).

---

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
- ein Runbook mit erwarteten Ergebnissen für Installation, Backup, Restore und Störung
- dokumentierte Versionen und Lizenzen von Karten- und Verkehrsnetzdaten

### Vorgesehener Betriebsweg

Nach der Implementierung soll eine lokale Installation ungefähr so aussehen:

```bash
cp .env.example .env
$EDITOR .env

docker compose up -d --build
```

Diese Befehle funktionieren im aktuellen Repository noch nicht, weil `compose.yaml`, `.env.example` und die Anwendung noch fehlen. Sie stehen hier als Ziel für die spätere Betriebsdokumentation und nicht als falsche Installationsanleitung.

### Geheimnisse und Konfiguration

Geplante Konfigurationsbereiche sind unter anderem:

- Datenbankverbindung
- öffentliche Basis-URL
- Session- oder Signaturschlüssel für Moderation
- Karten- und Netzdatenquellen
- Monitoring-Endpunkt
- Importkanal, falls später aktiviert
- Rate-Limit- und Ablaufparameter

Keine dieser Werte gehört in Git. Beispielkonfigurationen dürfen keine echten Schlüssel, Tokens, privaten URLs oder Produktionsdaten enthalten.

### Backup und Restore

Ein Backup gilt erst als vorhanden, wenn eine Wiederherstellung getestet wurde. Das Runbook soll dokumentieren:

1. wie ein verschlüsseltes Backup erstellt wird,
2. welche Daten es enthält und welche Löschfristen gelten,
3. wie eine isolierte Testdatenbank gestartet wird,
4. wie Migrationen angewendet werden,
5. wie Integrität und Datenumfang geprüft werden,
6. wie die Anwendung wieder auf die Datenbank zeigt,
7. wie ein gescheiterter Restore erkannt und abgebrochen wird.

### Monitoring

Mindestens überwacht werden sollen:

- API-Erreichbarkeit und Antwortzeiten
- Fehler nach Endpunkt und Statuscode
- Datenbankverbindung und Migrationstatus
- Anzahl und Dauer fehlgeschlagener Ablaufjobs
- Importfehler und veraltete Netzdaten
- Rate-Limit-Spitzen und ungewöhnliche Meldungsraten
- Backup-Alter und letzter erfolgreicher Restore-Test

Logs sollen strukturiert sein und keine vollständigen Meldungstexte oder personenbezogenen Daten unnötig wiederholen.

### Notfallmaßnahmen

Der Betrieb braucht mindestens Schalter für:

- neue Meldungen temporär annehmen oder ablehnen,
- einen einzelnen Importkanal deaktivieren,
- ein Verkehrsnetz vorübergehend ausblenden,
- eine kompromittierte Moderationssitzung widerrufen,
- eine auffällige Meldungsserie quarantänisieren,
- die öffentliche Karte in einen Wartungsmodus versetzen.

---

## Entwicklung und Tests

Die ausführbare Entwicklungsumgebung folgt mit der ersten Implementierung. Bis dahin gibt es keine gültigen Installations- oder Testbefehle für eine Anwendung.

### Geplante Testebenen

- **Domänentests:** Zeitvalidierung, Ablauf, Statusübergänge, Deduplizierung und Netzreferenzen
- **API-Tests:** Eingabevalidierung, Rate-Limits, Fehlerantworten und Berechtigungen
- **Datenbanktests:** Migrationen, Fremdschlüssel, Indizes, Löschfristen und konkurrierende Einsendungen
- **Integrationstests:** kompletter Ablauf mit echter Testdatenbank und synthetischen Netzdaten
- **End-to-End-Tests:** Meldung erstellen, anzeigen, markieren und automatisch ausblenden
- **Sicherheitstests:** keine privaten Felder in öffentlichen Antworten, keine unberechtigten Moderationszugriffe
- **Barrierefreiheit:** Tastaturbedienung, Fokusführung, Kontrast, Screenreader-Beschriftungen und reduzierte Bewegung
- **Betriebstests:** Ablaufdienst, Backup, Wiederherstellung und Notfallabschaltung
- **Lasttests:** viele öffentliche Lesezugriffe ohne unkontrollierte Datenbanklast

### Qualitätsregeln

- Keine echten personenbezogenen oder sensiblen Standortdaten in Fixtures, Logs oder Screenshots.
- Keine Tests, die durch veraltete Systemzeit zufällig grün oder rot werden; Zeit wird kontrollierbar injiziert.
- Kein dauerhaftes Überspringen fehlender Infrastruktur ohne sichtbaren Hinweis.
- Ein Test, der eine Sicherheits- oder Löschregel prüft, darf nicht still ignoriert werden.
- Jede öffentliche API-Antwort wird auf Datenminimierung geprüft.
- Migrationen werden sowohl auf einer leeren als auch auf einer bestehenden Testdatenbank geprüft.

### Geplante lokale Befehle

Die Namen sind vorläufig und werden an den gewählten Stack angepasst:

```bash
# Abhängigkeiten installieren
<package-manager> install

# Formatierung und statische Prüfung
<package-manager> format:check
<package-manager> lint

# Unit- und Integrationstests
<package-manager> test

# Browser- beziehungsweise End-to-End-Tests
<package-manager> test:e2e

# Produktionsartefakt bauen
<package-manager> build
```

Diese Platzhalter werden entfernt, sobald ein konkreter Stack festgelegt und lauffähig eingerichtet ist.

---

## Abnahmekriterien für den MVP

Der MVP gilt erst dann als vorzeigbar, wenn alle folgenden Punkte erfüllt sind:

### Funktion

- [ ] Eine ausgewählte Stadt beziehungsweise ein Netz kann geladen werden.
- [ ] Linien, Haltestellen und Richtungen stammen aus versionierten Netzdaten.
- [ ] Eine gültige Meldung kann erstellt und öffentlich angezeigt werden.
- [ ] Eine ungültige Linie-Ort-Kombination wird serverseitig abgelehnt.
- [ ] Beobachtungs- und Eingangszeitpunkt werden getrennt angezeigt beziehungsweise verarbeitet.
- [ ] Abgelaufene Meldungen erscheinen nicht mehr in der aktuellen Ansicht.
- [ ] Liste und Karte zeigen dieselbe veröffentlichte Datenbasis.
- [ ] Filter funktionieren auch ohne Reload der gesamten Anwendung.
- [ ] Eine Meldung kann markiert und moderiert werden.

### Sicherheit und Datenschutz

- [ ] Öffentliche Antworten enthalten keine internen IDs, IPs oder Moderationsnotizen.
- [ ] Moderationsfunktionen sind getrennt geschützt.
- [ ] Rate-Limits sind aktiv und werden getestet.
- [ ] Freie Texte werden begrenzt und sicher dargestellt.
- [ ] Lösch- und Aufbewahrungsfristen sind implementiert oder technisch eindeutig vorbereitet.
- [ ] Kartenanbieter und externe Dienste sind im Datenfluss dokumentiert.
- [ ] Geheimnisse sind nicht im Repository.
- [ ] Ein Bedrohungsmodell und ein Datenschutz-Technikdokument liegen vor.

### Betrieb

- [ ] Eine neue Instanz kann aus einer dokumentierten Anleitung aufgebaut werden.
- [ ] Ein Backup kann erstellt und in einer isolierten Umgebung wiederhergestellt werden.
- [ ] API, Datenbank und Ablaufdienst haben einen überprüfbaren Gesundheitsstatus.
- [ ] Der Dienst kann bei Missbrauch die Meldungsannahme abschalten.
- [ ] Fehler und Ablaufprobleme werden sichtbar, ohne sensible Daten zu loggen.

### Qualität

- [ ] Kernlogik ist automatisiert getestet.
- [ ] Ein kompletter Bericht-Lebenszyklus ist als Integrations- oder E2E-Test abgedeckt.
- [ ] Die Oberfläche ist mit Tastatur nutzbar.
- [ ] Mobile Ansichten und langsame Verbindungen wurden geprüft.
- [ ] Es existieren keine bewussten, unbekannten kritischen Testfehler.

---

## Roadmap

### Phase 0 – Grundlage

- [x] Projektname und Zielrichtung festlegen
- [x] Anforderungen, Grenzen und Datenschutzprinzipien dokumentieren
- [x] aktueller Projektstatus transparent machen
- [ ] rechtliche und datenschutzrechtliche Prüfung vorbereiten
- [ ] Zielstadt und erstes Verkehrsnetz auswählen
- [ ] Technologie, Lizenz und Datenquellen festlegen
- [ ] erstes Bedrohungsmodell als eigenes Dokument anlegen

### Phase 1 – Technischer Kern

- [ ] Repository-Struktur anlegen
- [ ] Entwicklungsumgebung reproduzierbar machen
- [ ] Verkehrsnetz als versionierten Datensatz einlesen
- [ ] Datenmodell und Migrationen implementieren
- [ ] Domänenregeln für Meldungsstatus und Ablauf implementieren
- [ ] API für Lesen und Erstellen von Meldungen bauen
- [ ] automatische Ablauf- und Bereinigungsjobs implementieren
- [ ] automatisierte Tests einrichten

### Phase 2 – Nutzbarer MVP

- [ ] mobile Liste und Karte
- [ ] Meldeformular mit kontrollierten Linien und Haltestellen
- [ ] sichtbares Alter jeder Meldung
- [ ] Filter für Netz, Verkehrsmittel, Linie, Richtung und Zeitraum
- [ ] Markieren-, Quarantäne- und Moderationsablauf
- [ ] Rate-Limits und Missbrauchsschutz
- [ ] erste selbst betriebene Testinstanz
- [ ] erste vollständige Datenschutz- und Nutzerdokumentation

### Phase 3 – Stabilisierung

- [ ] Barrierefreiheit prüfen und dokumentieren
- [ ] Datenschutz- und Sicherheitsdokumentation gegen Code und Betrieb prüfen
- [ ] Backup, Restore und Monitoring testen
- [ ] Last- und Ausfalltests durchführen
- [ ] Karten- und Netzdatenlizenzen prüfen
- [ ] öffentliche Pilotphase mit klarer Feedbackmöglichkeit

### Phase 4 – Ausbau

- [ ] weitere Verkehrsnetze nach Datenqualitätsprüfung hinzufügen
- [ ] optionalen, kontrollierten Community-Import entwickeln
- [ ] Mehrsprachigkeit und weitere Barrierefreiheitsverbesserungen
- [ ] Push-Hinweise nur mit ausdrücklicher Einwilligung
- [ ] unabhängige Sicherheitsprüfung vor größerem Betrieb

---

## Dokumentation, die noch entsteht

| Datei | Inhalt |
| --- | --- |
| `docs/SICHERHEIT.md` | Bedrohungsmodell, Vertrauensgrenzen, Gegenmaßnahmen und Restrisiken |
| `docs/DATENSCHUTZ-TECHNIK.md` | Datenflüsse, Speicherorte, Aufbewahrung und Löschung |
| `docs/NUTZERANLEITUNG.md` | Meldung erstellen, Karte nutzen, Beitrag markieren |
| `docs/MODERATION.md` | Kategorien, Entscheidungsregeln, Quarantäne und Eskalation |
| `docs/RUNBOOK.md` | Installation, Migrationen, Betrieb, Backup, Restore und Störungen |
| `docs/DATENQUELLEN.md` | Herkunft, Lizenz, Versionierung und Aktualisierung der Netzdaten |
| `CHANGELOG.md` | Nutzerrelevante Änderungen und behobene Sicherheitsfunde |
| `LICENSE` | gewählte Open-Source-Lizenz, sobald sie feststeht |

Keine dieser Dateien wird als vorhanden ausgegeben, bevor sie tatsächlich im Repository liegt.

---

## Mitmachen

Das Projekt befindet sich am Anfang. Besonders hilfreich sind Beiträge zu:

- UX für eine schnelle Nutzung unterwegs
- Datenmodellen für verschiedene Verkehrsnetze
- Datenschutz, Threat Modeling und Missbrauchsschutz
- Barrierefreiheit
- Teststrategie
- Karten- und GTFS-Datenquellen
- selbst betriebenem Deployment und Wiederherstellung

Für größere Änderungen bitte zuerst ein [Issue](https://github.com/nicolasb-wq/Pendler-Radar/issues) anlegen. Pull Requests sollten:

1. das Problem und die beabsichtigte Änderung erklären,
2. zwischen implementiertem Verhalten und Zielbild unterscheiden,
3. Datenschutz, Sicherheit und Barrierefreiheit berücksichtigen,
4. keine echten personenbezogenen oder sensiblen Meldungsdaten enthalten,
5. Tests oder eine nachvollziehbare Begründung für fehlende Tests mitbringen,
6. keine Stadt- oder Linienlogik unnötig im Kern festschreiben,
7. die Dokumentation aktualisieren, wenn sich Verhalten oder Betrieb ändern.

### Commit-Konvention

Für die Entwicklung ist folgende Schreibweise vorgesehen:

- `feat:` neue Funktion
- `fix:` Fehlerbehebung
- `refactor:` interne Umstrukturierung
- `test:` Tests
- `docs:` Dokumentation
- `chore:` Wartung und Tooling
- `security:` sicherheitsrelevante Änderung

---

## Inspiration und Abgrenzung

- [FreiFahren](https://freifahren.org/) – Inspiration für die Grundidee
- [FreiFahren auf GitHub](https://github.com/FreiFahren/FreiFahren) – öffentliches Vorbild
- [FreiFahren FAQ](https://freifahren.org/faq/) – Informationen zum Vorbild

Pendler-Radar übernimmt weder Code noch Marke oder Inhalte von FreiFahren. Die Projekte sind unabhängig voneinander. Die Nutzung des Vorbilds als Inspiration bedeutet nicht, dass FreiFahren dieses Projekt unterstützt oder für seine Richtigkeit verantwortlich ist.

---

## Lizenz

Die Lizenz wird vor Beginn der ersten technischen Umsetzung festgelegt. Bis dahin ist dieses Repository eine Projektskizze und keine veröffentlichte Software.
Projektskizze und keine veröffentlichte Software.
