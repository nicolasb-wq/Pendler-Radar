# Pendler-Radar – technischer Umsetzungsplan

**Status:** überarbeiteter Entwurf, noch nicht implementiert  
**Version:** 0.2  
**Stand:** 17. September 2026

Dieser Plan überarbeitet die erste Rohfassung des Projekts. Er trennt bewusst zwischen:

- dem fachlichen Produktziel,
- einem realistisch umsetzbaren MVP,
- technischen Entscheidungen,
- späteren Ausbaustufen,
- offenen Fragen, die vor dem produktiven Betrieb geklärt werden müssen.

Die wichtigste Änderung gegenüber der Rohfassung lautet:

> **Wir bauen zunächst keinen Verbund aus Microservices, sondern einen modularen Monolithen mit einem getrennten Hintergrundprozess.**
>
> Damit bleiben Entwicklung, Test und Betrieb beherrschbar. Die fachlichen Grenzen bleiben trotzdem sauber, sodass einzelne Komponenten später getrennt werden können, wenn es dafür einen nachgewiesenen Grund gibt.

---

## 1. Ziel des Projekts

Pendler-Radar soll aktuelle, gemeinschaftlich gemeldete Hinweise im Schienenverkehr anzeigen. Im ersten Anwendungsfall geht es um Beobachtungen zu Fahrkartenkontrollen:

1. Eine Person wählt Verkehrsnetz, Linie, Ort, Richtung und Beobachtungszeit.
2. Die Anwendung prüft die Eingabe und begrenzt Missbrauch.
3. Die Meldung erscheint mit sichtbarem Alter in einer Liste und auf einer Karte.
4. Nach kurzer Zeit verliert sie automatisch ihre Aktualität.
5. Falsche, doppelte oder problematische Meldungen können markiert und moderiert werden.

Die Anwendung ist keine offizielle Auskunft eines Verkehrsunternehmens. Sie ersetzt kein gültiges Ticket und garantiert nicht, dass eine Meldung richtig oder noch aktuell ist.

### Produktprinzipien

- **Orte im Schienennetz statt Nutzer-GPS:** Eine Meldung bezieht sich auf einen Bahnhof, einen Haltepunkt oder einen Streckenabschnitt aus den Netzdaten.
- **Zeitlich begrenzte Information:** Eine alte Meldung darf nicht wie eine aktuelle Warnung wirken.
- **Datensparsamkeit:** Keine dauerhaften Geräteprofile und keine unnötigen Standortdaten.
- **Sachlichkeit:** Keine Namen, Fotos, Audioaufnahmen oder Identifizierung von Personen.
- **Offene Unsicherheit:** Eine Community-Meldung ist keine amtliche Bestätigung.
- **Nachvollziehbarkeit:** Datenflüsse, Aufbewahrung und Betriebsgrenzen werden dokumentiert.

### Verbindlicher Geltungsbereich: nur Züge

Pendler-Radar umfasst ausschließlich den **Schienenverkehr mit Zügen**. Je nach Startnetz können S-Bahn, Regionalbahn, Regional-Express und – nach ausdrücklicher Entscheidung – Fernverkehr enthalten sein.

Busse, Straßenbahnen und U-Bahnen sind nicht Teil des MVP. Sie werden nicht in die erste Netzkonfiguration, die Fahrplandatenquelle oder die fachlichen Zugarten aufgenommen. Eine spätere Erweiterung wäre eine neue Produktentscheidung.

---

## 2. Korrekturen an der Rohfassung

| Rohfassung | Überarbeitete Entscheidung | Begründung |
| --- | --- | --- |
| Mehrere Microservices von Beginn an | Modularer Monolith plus separater Worker | Weniger Betriebsaufwand, einfachere Tests und für den MVP ausreichend |
| Caddy als DDoS- und Rate-Limit-Schutz | Caddy als TLS-Reverse-Proxy; Rate-Limits in API/Redis oder vorgeschaltetem Edge-Dienst | Ein Reverse-Proxy allein ist kein DDoS-Schutz. Caddys Rate-Limit-Funktionen sind nicht ohne Weiteres Bestandteil des Kernprodukts. |
| Exakte GPS-Position clientseitig prüfen | Keine Nutzer-GPS-Position im MVP | GPS ist für eine Meldung über Linie, Haltestelle und Richtung nicht erforderlich, ungenau und datenschutzrechtlich sensibel. |
| 100-Meter-Geohash speichern | Netzdaten-Referenz speichern: `network_id`, `line_id`, `stop_id` oder `segment_id` | Ein Geohash ist trotz Vergröberung eine Ortsangabe und löst das Grundproblem nicht. Der Ort im Schienennetz ist präziser für die Fachlogik und datensparsamer. |
| Kryptografischer Geräte-Hash als Authentifizierung | Kein Geräte-Hash als Identität; kurzlebige Anti-Missbrauch-Mechanismen | Ein Hash beweist keine Identität, kann kopiert werden und wirkt wie Fingerprinting. Er ist kein Authentifizierungsverfahren. |
| Trust-Score pro Gerät | Kein Nutzer-Trust-Score im MVP | Scores sind leicht manipulierbar, benachteiligen neue Nutzer:innen und speichern ein dauerhaftes Verhaltenssignal. |
| Neue Meldung erst nach Bestätigung durch „vertrauenswürdige“ Person | Neue Meldung sofort mit sichtbarer Unsicherheit oder bei Auffälligkeit in Quarantäne | Sonst entsteht ein Kaltstartproblem und die Anwendung macht neue, möglicherweise wichtige Meldungen unsichtbar. |
| WebSockets sofort | Zuerst Polling, danach optional SSE | Für eine überwiegend lesende Karte ist eine unidirektionale oder periodische Aktualisierung einfacher und robuster. |
| Redis als Pflichtbestandteil ab Tag 1 | Redis für nachgewiesenen Cache-, Lock- und Rate-Limit-Bedarf | Eine zusätzliche Infrastrukturkomponente soll einen konkreten Nutzen haben und nicht nur wegen des Architekturdiagramms existieren. |
| Datenbank-Hard-Delete nach 60–120 Minuten | Öffentliche Ablaufzeit getrennt von interner kurzer Aufbewahrung | Moderation, Fehleranalyse, Duplikaterkennung und DSGVO-Löschung brauchen ein bewusstes Datenmodell. |
| HAFAS-Proxy als frei nutzbarer Durchleiter | Begrenzter, eigener Adapter mit Cache, Timeout, Attribution und Nutzungsprüfung | Drittanbieter-APIs haben Bedingungen und dürfen nicht unkontrolliert als Proxy missbraucht werden. |

---

## 3. Umfang und Prioritäten

### 3.1 Muss in den MVP

- eine ausgewählte Stadt oder ein ausgewähltes Verkehrsnetz
- statische Netzdaten für Linien, Haltestellen und Richtungen
- API zum Lesen aktueller Meldungen
- API zum Erstellen einer Meldung
- Liste und Karte im mobilen Frontend
- sichtbare Beobachtungszeit und sichtbares Alter
- automatische Ablaufzeit
- serverseitige Validierung aller Eingaben
- Basis-Rate-Limit und Schutz vor offensichtlichem Spam
- Markieren und einfaches Moderieren von Meldungen
- PostgreSQL mit Migrationen
- automatisierte Tests für Kernlogik und API
- lokale reproduzierbare Entwicklungsumgebung
- grundlegende Datenschutz- und Sicherheitsdokumentation

### 3.2 Sinnvoll, wenn Zeit bleibt

- Redis-Cache für externe Abfahrtsdaten
- Request-Coalescing beziehungsweise Singleflight für Cache-Misses
- Stale-While-Revalidate für nicht kritische Fahrplandaten
- Server-Sent Events für schnellere Aktualisierung
- zusätzliche Qualitätsindikatoren auf Basis unabhängiger, aktueller Meldungen
- vorbereitete Unterstützung eines zweiten Verkehrsnetzes
- vollständiges Betriebs- und Restore-Runbook

### 3.3 Nicht im ersten MVP

- Microservice-Verbund mit mehreren unabhängig deploybaren Fachservices
- GPS-Prüfung der meldenden Person
- Geräte-Fingerprinting
- dauerhafter Trust-Score
- automatischer Telegram-Import
- Push-Benachrichtigungen
- native Android- und iOS-Apps
- komplexe Nutzerkonten und soziale Profile
- automatische Routenempfehlungen zur Umgehung von Kontrollen
- eigene Fahrplandatenbank für ganz Deutschland
- Echtzeit-WebSockets, falls Polling oder SSE ausreichen

---

## 4. Offene Produktfragen vor der Implementierung

Diese Fragen sind wichtiger als die Wahl zwischen Vue, React oder Svelte:

1. **Welche Stadt startet?** Berlin, Dresden oder ein anderes Netz?
2. **Welche Zugarten gehören zum Startumfang?** S-Bahn, Regionalbahn, Regional-Express und/oder Fernverkehr? Busse, Straßenbahnen und U-Bahnen gehören ausdrücklich nicht zum Umfang.
3. **Was genau ist eine Meldung?** Nur Fahrkartenkontrollen in Zügen oder auch Störungen, Ausfälle und Baustellen?
4. **Wie lange gilt eine Meldung als aktuell?** Ein einheitlicher Wert oder abhängig von der Zugart?
5. **Braucht das MVP externe Abfahrtsdaten überhaupt?** Für Meldungen können statische Netzdaten genügen.
6. **Wie werden neue Meldungen behandelt?** Sofort sichtbar, gedämpft dargestellt oder bei auffälliger Rate quarantänisiert?
7. **Wie kann die Community moderieren, ohne Nutzerprofile aufzubauen?**
8. **Welche Datenquelle und Lizenz verwenden wir für Linien und Haltestellen?**
9. **Welche Nutzung externer Fahrplandienste ist zulässig?**
10. **Wer trägt den Betrieb und bearbeitet Datenschutz- oder Löschanfragen?**

### Entscheidungskriterium

Eine technische Funktion kommt nur in den MVP, wenn sie mindestens einen dieser Punkte erfüllt:

- sie ist für den Kernnutzen erforderlich,
- sie reduziert ein konkretes Sicherheits- oder Missbrauchsrisiko,
- sie ist für die IHK-Projektabgrenzung fachlich relevant,
- sie lässt sich innerhalb des verfügbaren Zeitbudgets testen und betreiben.

---

## 5. Zielarchitektur

### 5.1 Laufzeitarchitektur

```text
                 +---------------------------+
                 | Browser / PWA             |
                 | Liste, Karte, Formular    |
                 +-------------+-------------+
                               |
                          HTTPS / JSON
                               |
                 +-------------v-------------+
                 | Caddy                     |
                 | TLS, statische Dateien,   |
                 | Sicherheits-Header        |
                 +-------------+-------------+
                               |
                 +-------------v-------------+
                 | Pendler-Radar API         |
                 | modularer FastAPI-Dienst  |
                 | Validierung, Reports,     |
                 | Moderation, Cache-Zugriff |
                 +------+--------------------+
                        |
            +-----------+-----------+
            |                       |
     +------v------+         +------v------+
     | PostgreSQL   |         | Redis        |
     | Reports,     |         | Cache,       |
     | Netzdaten,   |         | Rate-Limits, |
     | Moderation   |         | Locks        |
     +------+-------+         +------+-------+
            |                        |
            +------------+-----------+
                         |
                 +-------v--------+
                 | Worker          |
                 | Ablauf,         |
                 | Bereinigung,    |
                 | externe Refreshes|
                 +-----------------+
```

Der Worker verwendet möglichst dieselben Domänenmodule wie die API, wird aber als eigener Prozess gestartet. Das ist eine sinnvolle Trennung für Hintergrundaufgaben, ohne aus jedem Modul einen eigenen Netzwerkdienst zu machen.

### 5.2 Fachliche Module im Backend

Der Backend-Dienst sollte intern mindestens diese Module besitzen:

- `reports`: Erstellen, Lesen, Ablauf und Statusübergänge
- `transit`: Netze, Linien, Haltestellen, Richtungen und Datenversionen
- `moderation`: Markierungen, Quarantäne und Bearbeitungsaktionen
- `abuse_prevention`: Rate-Limits, Deduplizierung und Auffälligkeiten
- `departures`: optionaler Adapter für externe Abfahrtsdaten
- `realtime`: optionales Publizieren von Aktualisierungen
- `health`: technische und fachliche Gesundheitsprüfungen
- `privacy`: Löschjobs, Aufbewahrung und minimierte Logs

Diese Module dürfen eigene Schnittstellen besitzen, sollen aber zunächst in einem Prozess deploybar bleiben.

### 5.3 Warum kein Microservice-Setup am Anfang?

Ein Microservice-Setup würde zusätzlich erfordern:

- Service-to-Service-Authentifizierung,
- Versionierung mehrerer APIs,
- verteiltes Tracing,
- separate Deployments und Health Checks,
- Netzwerkfehler und Wiederholungslogik,
- Datenkonsistenz über Servicegrenzen,
- mehr Backups, Logs und Monitoring.

Das sind echte Lernziele, aber sie lösen nicht das Kernproblem der App. Für einen IHK-Projektumfang ist ein modularer Monolith mit einem klar abgegrenzten Cache-/Worker-Anteil leichter zu erklären und belastbarer zu prüfen. Eine spätere Aufteilung bleibt möglich, wenn Last, Teamgröße oder unabhängige Skalierung das rechtfertigen.

---

## 6. Technologievorschlag

Die Auswahl ist noch offen. Für eine konkrete Umsetzung wird folgender Stack empfohlen:

| Schicht | Vorschlag | Warum |
| --- | --- | --- |
| Frontend | Vue 3 + TypeScript + Vite | schneller Einstieg, gute PWA-Basis und klare Komponentenstruktur |
| Karte | Leaflet | offen, leichtgewichtig und mit verschiedenen Tile-Anbietern nutzbar |
| Backend | Python 3.12 + FastAPI | gute Validierung mit Pydantic, asynchrone HTTP-Anbindung und verständliche API |
| Datenbank | PostgreSQL | relationale Daten, Migrationen und zuverlässige Transaktionen |
| Geodaten | zunächst keine PostGIS-Pflicht | Netzdaten-Referenzen reichen im MVP; PostGIS nur bei echten räumlichen Anforderungen |
| Cache | Redis ab dem Cache-/Rate-Limit-Sprint | TTL, atomare Operationen und Locking |
| Reverse Proxy | Caddy | einfache TLS-Terminierung und statische Auslieferung |
| lokale Umgebung | Docker Compose | reproduzierbarer Start von API, Datenbank und optional Redis |
| Tests | Pytest, API-Integrationstests, Browser-E2E | Fachlogik, echte Datenbankpfade und Oberfläche abdecken |

Die Empfehlung ist kein bereits vorhandener Stack. Vor Beginn des ersten Sprints wird sie bestätigt oder mit einer kurzen Architekturentscheidung ersetzt.

### PostGIS: erforderlich oder nicht?

PostGIS ist nur notwendig, wenn die Anwendung tatsächlich räumliche Operationen benötigt, zum Beispiel:

- Suche nach Meldungen in einer frei wählbaren Kartenbox,
- räumliche Clusterbildung,
- Linien- oder Streckenabschnittsberechnung,
- Geofencing für öffentliche Netzdaten.

Für Meldungen, die bereits auf eine Haltestelle oder einen Streckenabschnitt verweisen, genügen zunächst:

- `network_id`,
- `line_id`,
- `stop_id` oder `segment_id`,
- Koordinaten aus den statischen Netzdaten für die Kartendarstellung.

Eine räumliche Datenbank einzusetzen, nur um ein 100-Meter-Geohash aus einer Nutzerposition zu berechnen, wäre unnötig und würde die Datenschutzfrage nicht lösen.

---

## 7. Datenquellen und Fahrplan-Integration

### 7.1 Statische Netzdaten

Die Meldefunktion braucht eine stabile Zuordnung von:

- Verkehrsnetz,
- Zugart, zum Beispiel S-Bahn, Regionalbahn, Regional-Express oder Fernverkehr,
- Linie,
- Haltestelle,
- Streckenabschnitt,
- Richtung.

Diese Daten sollten als versionierter Import vorliegen, zum Beispiel aus einem lizenzierten GTFS-Datensatz oder einer offiziell bereitgestellten Quelle. Jede Version braucht:

- Quelle,
- Lizenz/Attribution,
- Importzeitpunkt,
- Versionskennung,
- Anzahl der importierten Datensätze,
- Ergebnis des Validierungslaufs.

Die Anwendung darf nicht von freien Texteingaben abhängig sein, wenn eine kontrollierte Linien- oder Haltestellenreferenz möglich ist.

### 7.2 Externe Abfahrtsdaten

Ein Abfahrts-Proxy ist nützlich, aber für den ersten Meldungs-MVP nicht zwingend. Wenn er umgesetzt wird, darf er kein beliebiger Weiterleitungs-Proxy werden.

Der Adapter braucht:

- eine feste Liste unterstützter Endpunkte,
- Eingabevalidierung der Stations-ID,
- feste Timeouts,
- begrenzte Antwortgröße,
- Upstream-Fehlerbehandlung,
- dokumentierte Attribution,
- Prüfung der Nutzungsbedingungen,
- kein Weiterreichen von Nutzer-IP oder unnötigen Headern,
- Metriken für Cache-Hit, Cache-Miss und Upstream-Fehler.

### 7.3 Redis-Cache und Request-Coalescing

Für Abfahrtsdaten ist ein kurzer Cache sinnvoll. Der Ablauf kann so aussehen:

1. **Fresh hit:** Ein gültiger Cachewert wird sofort zurückgegeben.
2. **Stale hit:** Ein leicht veralteter Wert wird als solcher zurückgegeben; genau ein Worker aktualisiert ihn im Hintergrund.
3. **Miss:** Der erste Prozess erhält einen kurzen, token-basierten Redis-Lock.
4. **Lock owner:** Er ruft den Upstream mit Timeout auf, validiert die Antwort und schreibt den Cache.
5. **Concurrent miss:** Weitere Anfragen warten nur kurz oder erhalten einen vorhandenen Stale-Wert; sie starten keinen eigenen Upstream-Request.
6. **Fehler:** Nach Timeout oder ungültiger Antwort bleibt ein alter Wert erhalten oder es wird ein kontrollierter Fehler zurückgegeben.
7. **Lock release:** Der Lock wird nur vom Besitzer mit seinem Token gelöscht; ein abgelaufener Lock darf nicht von einem fremden Prozess entfernt werden.

Mögliche Schlüsselbestandteile:

```text
provider:departures:{network}:{station}:{products}:{time_window}
```

Die konkrete TTL wird anhand der Datenart und der Upstream-Bedingungen festgelegt. „60 Sekunden für alles“ ist keine ausreichende Cache-Strategie.

---

## 8. Melde- und Datenmodell

### 8.1 Öffentliche Meldung

Eine Meldung braucht im MVP:

- `id`: zufällige, nicht fortlaufende Kennung
- `network_id`
- `mode`
- `line_id`
- `stop_id` oder `segment_id`
- `direction`
- `observed_at`
- `created_at`
- `expires_at`
- `status`
- optionaler, kurzer, bereinigter Hinweistext
- `source`, zum Beispiel `web`

Nicht benötigt werden:

- exakte GPS-Koordinaten der meldenden Person,
- Geräte-ID,
- Name oder E-Mail,
- Foto/Video/Audio,
- öffentlicher Trust-Score,
- vollständige IP-Adresse in der Meldung.

### 8.2 Zeitregeln

- Speicherung intern in UTC.
- Anzeige in der für das Verkehrsnetz festgelegten Zeitzone.
- Zukunftszeitpunkte werden abgelehnt, abgesehen von einer dokumentierten Uhrtoleranz.
- Sehr alte Meldungen werden abgelehnt oder direkt als nicht aktuell behandelt.
- `observed_at` darf nicht heimlich durch `created_at` ersetzt werden.
- `expires_at` wird serverseitig berechnet und nicht vom Client vertraut.

### 8.3 Statusmaschine

```text
received -> published -> expired
     |          |
     v          v
quarantined -> removed
     |
     +-------> published
```

Vorgeschlagene Regeln:

- `received`: angenommen, aber noch nicht öffentlich sichtbar
- `published`: öffentlich sichtbar und innerhalb der Ablaufzeit
- `quarantined`: vorübergehend verborgen oder nur für Moderation sichtbar
- `expired`: automatisch nicht mehr aktuell
- `removed`: absichtlich entfernt und nicht öffentlich auslieferbar

Ein Statuswechsel wird nur über eine Domänenfunktion durchgeführt. Controller und Worker dürfen nicht direkt beliebige Statuswerte in die Datenbank schreiben.

### 8.4 Duplikaterkennung

Ein mögliches Duplikat wird anhand einer Kombination aus:

- Netz,
- Zugart, zum Beispiel S-Bahn, Regionalbahn, Regional-Express oder Fernverkehr,
- Linie,
- Ort beziehungsweise Abschnitt,
- Richtung,
- engem Zeitfenster,
- bereinigtem Hinweistext.

erkannt.

Deduplizierung darf nicht blind zwei verschiedene Beobachtungen zusammenlegen. Im Zweifel bleiben Meldungen getrennt und werden nur in der Oberfläche gruppiert. Die genaue Zeitspanne wird mit Testdaten und realen Pilotdaten bestimmt.

---

## 9. Warum kein Geräte-Hash?

Die Rohfassung schlägt vor, beim ersten Start nach Zustimmung einen kryptografischen 256-Bit-Hash zu erzeugen und diesen zur Authentifizierung von Meldungen zu verwenden. Das hat drei Probleme:

1. **Ein Hash ist kein Beweis.** Wer den Wert kopiert, kann ihn wiederverwenden.
2. **Ein dauerhafter Wert ist ein Identifikator.** Auch ohne Namen kann er Verhalten über Zeit verknüpfen.
3. **Der Browser ist kontrollierbar.** Ein Nutzer kann lokale Speicherwerte löschen oder beliebig viele Clients erzeugen.

Für den MVP wird deshalb empfohlen:

- Lesen ohne Gerätedatensatz,
- Melden ohne dauerhaften Geräte-Hash,
- Rate-Limit auf möglichst kurzer technischer Grundlage,
- technische Metadaten nur mit kurzer Aufbewahrung und dokumentiertem Zweck,
- zusätzliche Challenge nur bei Auffälligkeit,
- keine Behauptung, anonyme Einsendungen seien eine sichere Identität.

Ob ein kurzlebiger, serverseitig ausgegebener Anti-Abuse-Token sinnvoll ist, wird getrennt geprüft. Er wäre ein Missbrauchsschutz, keine Authentifizierung und kein Trust-Score.

### Consent und TDDDG

Ein Consent-Banner ist kein Ersatz für ein Datenschutzkonzept. Vor dem Banner muss geklärt werden, welche Speicherung tatsächlich erforderlich ist.

- Der Kern der öffentlichen Ansicht sollte ohne nicht erforderliches Tracking funktionieren.
- Optionale lokale Präferenzen, Push oder Geolocation brauchen eine eigene Betrachtung.
- Ein lokaler Schlüssel oder Cookie darf nicht automatisch als „anonym“ bezeichnet werden.
- Rechtsgrundlage, Zweck, Speicherdauer und Widerruf müssen vor dem Produktivbetrieb geprüft werden.
- Diese README ist keine Rechtsberatung.

---

## 10. Trust, Bestätigungen und Qualität

### 10.1 Warum kein persönlicher Trust-Score?

Ein Score pro Gerät klingt einfach, ist aber fachlich und datenschutzrechtlich problematisch:

- neue Nutzer:innen starten mit einem Nachteil,
- Geräte können gewechselt oder zurückgesetzt werden,
- mehrere Personen können ein Gerät verwenden,
- ein Angreifer kann sich mehrere Werte erzeugen,
- Upvotes können koordiniert manipuliert werden,
- das System würde ein langfristiges Verhaltensprofil führen.

Die Anwendung sollte die Qualität einer Meldung nicht an eine vermeintlich vertrauenswürdige Person delegieren.

### 10.2 Besserer erster Ansatz

Im MVP wird Qualität über den Kontext bewertet:

- Aktualität der Beobachtung,
- Plausibilität der Netzdatenreferenzen,
- Rate und Muster des Einsenders auf kurzer Zeitbasis,
- Duplikate,
- Meldungen der Community,
- unabhängige weitere Beobachtungen im gleichen Zeitfenster.

Eine Meldung darf bei Auffälligkeit quarantänisiert werden. Neue Meldungen werden nicht grundsätzlich unsichtbar gemacht.

### 10.3 Bestätigungen als spätere Funktion

Falls später eine Bestätigungsschaltfläche kommt, soll sie zunächst nur ein Qualitätssignal für die Oberfläche sein. Sie darf nicht automatisch:

- den Beitrag als amtlich wahr markieren,
- den Autor:innen einen dauerhaften Score geben,
- eine andere Meldung löschen,
- neue Nutzer:innen benachteiligen.

---

## 11. Echtzeit-Aktualisierung

### 11.1 Stufe 1: Polling

Für den MVP wird zunächst ein periodischer Abruf empfohlen:

- aktuelle Meldungen beim Laden,
- Aktualisierung in einem begrenzten Intervall,
- Backoff bei Fehlern oder inaktiven Tabs,
- kein Abruf, wenn die Seite im Hintergrund ist und es keinen Grund gibt.

Das ist einfacher zu testen und ausreichend, wenn Meldungen nur wenige Minuten aktuell sind.

### 11.2 Stufe 2: Server-Sent Events

SSE ist für diese Anwendung meist passender als WebSockets:

- Server sendet Updates in eine Richtung.
- Der Browser kann bei Verbindungsabbruch neu verbinden.
- Neue Meldungen können nach Netzwerk oder Kartenbereich gefiltert werden.
- Der Client kann nach Reconnect den vollständigen aktuellen Stand nachladen.

### 11.3 Stufe 3: WebSockets nur bei Bedarf

WebSockets werden erst eingesetzt, wenn Messungen zeigen, dass Polling oder SSE nicht ausreichen. Redis Pub/Sub kann dann als interner Verteiler dienen. Pub/Sub ist aber nicht dauerhaft:

- verpasste Nachrichten können nicht nachträglich aus Pub/Sub gelesen werden,
- Clients müssen nach Reconnect immer neu synchronisieren,
- die Datenbank bleibt die Quelle der Wahrheit,
- ein Redis-Ereignis darf keine Meldung ersetzen.

---

## 12. Datenschutz ohne Nutzer-GPS

Die Rohfassung möchte die GPS-Position nur im RAM prüfen und nicht speichern. Das reduziert zwar die Speicherung, beseitigt aber nicht alle Risiken:

- GPS ist personenbezogen beziehungsweise personenbeziehbar.
- Die Erfassung braucht eine Berechtigung und kann abgelehnt oder gefälscht werden.
- Innenräume, Tunnel und Geräte liefern ungenaue Werte.
- Die Prüfung kann legitime Meldungen blockieren, wenn das Gerät keinen Empfang hat.
- Die Meldung selbst enthält bereits einen geeigneten Ort im Schienennetz.

Daher wird im MVP keine GPS-Prüfung durchgeführt. Stattdessen:

- Nutzer:innen wählen einen Ort aus kontrollierten Netzdaten.
- Plausibilität wird über Linie, Haltestelle, Richtung und Zeit geprüft.
- Missbrauch wird über Rate-Limits, Quarantäne und Moderation behandelt.
- Geolocation bleibt eine mögliche spätere, separat geprüfte Funktion – nicht die Grundlage der Authentifizierung.

---

## 13. API-Entwurf

### Öffentliche Endpunkte

| Methode | Endpunkt | Zweck |
| --- | --- | --- |
| `GET` | `/api/v1/health` | technischer Gesundheitsstatus |
| `GET` | `/api/v1/networks` | verfügbare Verkehrsnetze |
| `GET` | `/api/v1/networks/{network_id}/lines` | Linien, Orte und Richtungen |
| `GET` | `/api/v1/reports` | aktuelle Meldungen mit Filtern |
| `POST` | `/api/v1/reports` | neue Meldung anlegen |
| `POST` | `/api/v1/reports/{report_id}/flags` | Meldung markieren |
| `GET` | `/api/v1/departures/{station_id}` | optionaler, begrenzter Fahrplan-Proxy |

### Interne beziehungsweise geschützte Endpunkte

| Methode | Endpunkt | Zweck |
| --- | --- | --- |
| `GET` | `/api/v1/moderation/queue` | Moderationswarteschlange |
| `POST` | `/api/v1/moderation/reports/{id}/publish` | Quarantäne aufheben |
| `POST` | `/api/v1/moderation/reports/{id}/remove` | Meldung entfernen |
| `GET` | `/api/v1/admin/metrics` | geschützte Betriebsmetriken |

### Beispiel: Meldung erstellen

```json
{
  "networkId": "example-network",
  "mode": "s_bahn",
  "lineId": "s2",
  "stopId": "example-stop",
  "direction": "outbound",
  "observedAt": "2026-09-17T12:34:00Z",
  "note": "Sachliche optionale Ergänzung"
}
```

Serverseitig werden die IDs gegen die aktive Datenversion aufgelöst. Der Client darf weder `expiresAt` noch `status`, `source` oder interne Moderationsfelder setzen.

### Beispiel: Fehlerantwort

```json
{
  "error": {
    "code": "REPORT_TIME_IN_FUTURE",
    "message": "Der Beobachtungszeitpunkt liegt in der Zukunft.",
    "fields": ["observedAt"]
  }
}
```

Fehlercodes bleiben stabil, Meldungen können übersetzt werden. Stacktraces, SQL-Fehler und interne Pfade werden nie an den Client gegeben.

---

## 14. Cache-Strategie

### Was gecacht wird

Geeignet für Cache:

- externe Abfahrtsdaten,
- statische Netzdaten,
- kurzzeitig aggregierte öffentliche Meldungsabfragen, sofern Ablaufzeiten beachtet werden.

Nicht unkritisch cachen:

- Moderationsentscheidungen ohne Invalidierung,
- abgelaufene Meldungen,
- personenbezogene oder technische Anti-Abuse-Daten mit unklarer Löschfrist.

### Cache-Anforderungen

- Schlüssel enthalten alle fachlich relevanten Parameter.
- TTL wird pro Datenart festgelegt.
- Upstream-Timeouts sind begrenzt.
- Stale-Daten werden sichtbar oder kontrolliert behandelt.
- Cache-Ausfall darf die Meldungsfunktion nicht vollständig lahmlegen.
- Redis ist nicht die dauerhafte Quelle der Wahrheit.
- Locks haben Ablaufzeit und Besitzer-Token.
- Ein Lock wird nicht blind von einem anderen Prozess gelöscht.

### Rate-Limits

Rate-Limits werden zunächst pro Endpunkt und technischer Quelle entworfen:

- öffentliche GET-Abfragen großzügiger als POST-Meldungen,
- Burst und langfristiges Fenster getrennt,
- Antwort `429 Too Many Requests` mit verständlicher Retry-Information,
- Monitoring ohne dauerhafte Nutzerprofile,
- Möglichkeit zur Anpassung, ohne Codeänderung deployen zu müssen.

Ein Reverse Proxy kann eine erste grobe Grenze setzen. Die fachliche Rate-Limit-Logik bleibt trotzdem im Backend, damit sie unabhängig vom Proxy getestet werden kann.

---

## 15. Sicherheit und Datenschutz

### 15.1 Bedrohungsmodell

| Bedrohung | Gegenmaßnahme |
| --- | --- |
| Bot erzeugt viele Meldungen | Rate-Limits, Request-Größenlimit, Quarantäne, Monitoring |
| Nutzer fälscht Zeit oder Linie | serverseitige Prüfung gegen Netz- und Zeitdaten |
| XSS über Hinweistext | strikte Ausgabe-Kodierung, CSP, kein unbereinigtes HTML |
| SQL-Injection | parametrisierte Zugriffe und Integrationstests |
| Missbrauch geschützter Endpunkte | Rollen, sichere Sitzungen, CSRF-Schutz bei Cookie-Auth |
| Redis-Ausfall | kontrollierter Fallback, keine dauerhafte Datenabhängigkeit |
| Upstream-Ausfall | Timeout, Cache, Stale-Strategie, klare Fehlermeldung |
| DDoS | Rate-Limits, Reverse Proxy und gegebenenfalls vorgeschalteter Provider; kein Versprechen vollständiger Abwehr |
| Personenbezogene Inhalte | Textregeln, Meldefunktion, Moderation und Löschprozess |
| Kompromittiertes Backup | Verschlüsselung, Zugriffstrennung und Restore-Tests |
| gefälschter Import | verifizierter Adapter und gleiche Validierung wie bei Web-Meldungen |

### 15.2 Rollen

- **Öffentlich:** aktuelle Meldungen und freigegebene Netzdaten lesen.
- **Meldend:** Meldung einreichen und öffentliche Markierung senden; kein öffentliches Profil.
- **Moderation:** Markierungen prüfen und Status ändern.
- **Administration:** Betrieb, Konfiguration und Berechtigungen; möglichst getrennt von Moderation.
- **Worker:** nur die für Ablauf, Bereinigung und Cache benötigten Rechte.

### 15.3 Aufbewahrung

Die öffentliche Ablaufzeit ist nicht automatisch die einzige interne Löschfrist. Vorläufiges Modell:

| Datenart | Behandlung |
| --- | --- |
| aktuelle Meldung | bis zum Ablauf öffentlich |
| abgelaufene Rohmeldung | kurze technische Aufbewahrung nach dokumentiertem Zweck, danach löschen |
| Moderationsentscheidung | begrenzte Aufbewahrung für Nachvollziehbarkeit |
| Rate-Limit-Schlüssel | kurzlebig und zweckgebunden |
| Fehlerlog | minimiert, ohne vollständige Eingaben |
| Backup | verschlüsselt und mit eigener Löschfrist |

Konkrete Fristen werden erst nach fachlicher, technischer und rechtlicher Prüfung festgeschrieben.

---

## 16. Moderation

### Meldekategorien

- `stale`: vermutlich veraltet
- `duplicate`: doppelte Meldung
- `privacy`: personenbezogener Inhalt
- `abuse`: Beleidigung, Bedrohung oder gezielte Belästigung
- `wrong_location`: falsche Linie, Haltestelle oder Richtung
- `spam`: wiederholte oder automatisierte Einsendung

### Moderationsablauf

1. Markierung wird angenommen und rate-limitiert.
2. Meldung bleibt entweder sichtbar oder wird bei hohem Risiko automatisch quarantänisiert.
3. Moderation sieht die Meldung und die Kategorie, aber nicht mehr technische Daten als nötig.
4. Eine Entscheidung wird mit Kategorie und Zeitpunkt protokolliert.
5. Die öffentliche Darstellung wird aktualisiert.
6. Löschung und Aufbewahrung folgen der festgelegten Frist.

Markierungen sind kein Beweis. Ein einzelner Upvote macht eine Meldung nicht wahr; viele Markierungen machen sie nicht automatisch falsch.

---

## 17. Umsetzung in Sprints

### Sprint 0 – Entscheidungen und Begrenzung

**Ziel:** Ein umsetzbarer, prüfbarer Projektumfang steht fest.

Aufgaben:

- Zielstadt und Verkehrsnetz auswählen
- fachliche Definition einer Meldung festlegen
- Zugarten für den Start festlegen; Bus, Straßenbahn und U-Bahn bleiben außerhalb des Scopes
- Datenquelle und Lizenz auswählen
- exakte MVP-Abnahmekriterien bestätigen
- Stackentscheidung dokumentieren
- Datenschutz- und Bedrohungsmodell als Arbeitsdokument beginnen
- externe Fahrplan-API nur aufnehmen, wenn sie für den Projektnutzen erforderlich ist

**Ergebnis:** Architekturentscheidung, Datenquellenentscheidung und MVP-Scope.

### Sprint 1 – Repository und lokale Infrastruktur

**Ziel:** Ein reproduzierbarer, noch kleiner Entwicklungsstack läuft.

Aufgaben:

- Projektstruktur anlegen
- Python-Umgebung und Formatierung einrichten
- FastAPI-Anwendung mit `/api/v1/health` erstellen
- PostgreSQL per Compose starten
- Migrationen mit leerem und bestehendem Schema prüfen
- Caddy zunächst optional für den lokalen Stack konfigurieren
- Umgebungsvariablen dokumentieren
- CI für Formatierung, Lint und Unit-Tests einrichten

**Ergebnis:** Ein neuer Entwickler kann die API und Datenbank lokal starten und einen grünen Health-Test ausführen.

### Sprint 2 – Verkehrsnetzdaten

**Ziel:** Die Anwendung kennt ein kontrolliertes Netz.

Aufgaben:

- Importformat festlegen
- Netze, Linien, Haltestellen und Richtungen modellieren
- Importvalidierung schreiben
- Datensatzversion speichern
- ungültige Referenzen melden
- Quelle und Lizenz dokumentieren
- öffentliche Endpunkte für Netz und Linien bauen

**Ergebnis:** Das Frontend kann valide Auswahlwerte laden; freie Fantasielinien werden abgelehnt.

### Sprint 3 – Meldungen und Ablauf

**Ziel:** Der Kernnutzen funktioniert ohne externe Live-Fahrplandaten.

Aufgaben:

- `reports`-Tabelle und Migration
- Eingabe- und Zeitvalidierung
- Statusmaschine
- `POST /reports`
- `GET /reports`
- serverseitige Ablaufberechnung
- Ablaufjob im Worker
- zusätzliche Ablaufprüfung beim Lesen
- Duplikaterkennung als vorsichtige Heuristik

**Ergebnis:** Eine Meldung kann erstellt, angezeigt, zeitlich eingeordnet und automatisch ausgeblendet werden.

### Sprint 4 – Frontend

**Ziel:** Der Kernablauf funktioniert mobil.

Aufgaben:

- Liste aktueller Meldungen
- Karte mit Netzdaten und Meldungsmarkern
- Meldeformular mit kontrollierten Auswahlfeldern
- Filter
- Alter und Ablauf verständlich anzeigen
- Lade-, Offline- und Fehlerzustände
- Tastaturbedienung und Screenreader-Beschriftungen
- keine Browser-Geolocation im MVP

**Ergebnis:** Der komplette Nutzerablauf kann im Browser durchgespielt werden.

### Sprint 5 – Moderation und Missbrauchsschutz

**Ziel:** Der Dienst bleibt bei falschen oder problematischen Beiträgen handhabbar.

Aufgaben:

- Flag-Endpunkt
- Markierungskategorien
- Moderationswarteschlange
- Quarantäne und Entfernen
- Rate-Limits auf API-Ebene
- Eingabe- und Antwortgrößen begrenzen
- minimierte Logs
- Testfälle für Spam und unberechtigte Moderationszugriffe

**Ergebnis:** Eine problematische Meldung kann gemeldet, geprüft und aus der öffentlichen Ansicht entfernt werden.

### Sprint 6 – Externe Abfahrtsdaten, falls priorisiert

**Ziel:** Ein begrenzter, sauberer Read-Only-Adapter ist verfügbar.

Aufgaben:

- Nutzungsbedingungen und Attribution prüfen
- Adapter statt beliebigem Proxy implementieren
- Timeout und Fehlerverhalten
- Redis-Cache
- Request-Coalescing
- Stale-While-Revalidate, wenn fachlich sinnvoll
- Metriken für Upstream und Cache
- Tests mit Mock-Upstream und Fehlerfällen

**Ergebnis:** Ein externer Ausfall führt nicht zu einem hängenden Backend und nicht zu einer Anfrageflut.

### Sprint 7 – Betrieb und Abschluss

**Ziel:** Der Projektstand ist reproduzierbar und vorzeigbar.

Aufgaben:

- Produktionsähnlicher Compose-Stack
- HTTPS-Reverse-Proxy dokumentieren
- Backup und Restore testen
- Monitoring und Health Checks
- Sicherheits- und Datenschutzdokumente gegen Code prüfen
- Lasttest für Lesen und Meldungen
- bekannte Grenzen und nicht getestete Punkte dokumentieren
- Demoablauf und technische Präsentation vorbereiten

**Ergebnis:** Der MVP kann nachvollziehbar installiert, getestet, betrieben und wiederhergestellt werden.

---

## 18. IHK-Projektabgrenzung

Die Idee eignet sich als IHK-Abschlussprojekt, wenn die Aufgabe nicht als „eine Karte bauen“ formuliert wird, sondern als nachvollziehbare technische Entscheidung mit messbarem Ergebnis. Die genaue Zeit und die Vorgaben der zuständigen IHK müssen vorher geprüft werden.

### Geeigneter Projektschwerpunkt

> Konzeption und Umsetzung eines datensparsamen Meldesystems für aktuelle Beobachtungen im Schienenverkehr mit externer Fahrplanintegration, Cache-Strategie, automatischem Ablauf und Moderationsschnittstelle.

### Was daran prüfbar ist

- Anforderungsanalyse
- Architekturentscheidung gegen oder für Microservices
- Datenmodell und Migrationen
- API-Design
- Rate-Limit- und Cache-Konzept
- externe API-Integration
- Datenschutzentscheidung gegen Nutzer-GPS und Geräte-Fingerprinting
- Fehler- und Ausfallverhalten
- automatisierte Tests
- Deployment und Dokumentation

### Was im IHK-Zeitrahmen gestrichen werden kann

1. native Apps,
2. Telegram-Import,
3. WebSockets,
4. Trust-Score,
5. Mehrstadtbetrieb,
6. komplexe Nutzerkonten,
7. räumliche PostGIS-Analysen, wenn Netzdaten-IDs ausreichen.

Ein kleiner, sauber geprüfter Scope ist besser als ein Microservice-System, das am Ende weder vollständig getestet noch sicher betrieben werden kann.

---

## 19. Geplante Konfiguration

Beispielhafte Bereiche, noch keine fertige `.env`:

```text
APP_ENV=development
APP_BASE_URL=http://localhost
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
REPORT_DEFAULT_TTL_SECONDS=...
REPORT_MAX_AGE_SECONDS=...
UPSTREAM_DEPARTURES_BASE_URL=https://...
UPSTREAM_TIMEOUT_SECONDS=...
RATE_LIMIT_REPORTS_PER_WINDOW=...
LOG_LEVEL=INFO
```

Regeln:

- keine echten Werte im Repository,
- keine Produktionsschlüssel in Beispieldateien,
- sichere Standardwerte in Produktion,
- Ablauf- und Rate-Limit-Werte ohne Codeänderung konfigurierbar,
- Konfigurationsänderungen mit Auswirkungen auf Datenschutz oder Aktualität dokumentieren.

---

## 20. Abnahmekriterien

### Fachlich

- [ ] Eine Meldung kann ohne Geräte-GPS erstellt werden.
- [ ] Linie, Ort und Richtung werden gegen Netzdaten geprüft.
- [ ] Beobachtungszeit und Eingangszeit bleiben getrennt.
- [ ] Das Ablaufdatum wird serverseitig gesetzt.
- [ ] Abgelaufene Meldungen verschwinden aus der aktuellen Ansicht.
- [ ] Eine Meldung kann markiert und moderiert werden.
- [ ] Liste und Karte zeigen dieselbe Datenbasis.

### Technisch

- [ ] API-Fehler haben stabile Fehlercodes.
- [ ] Datenbankmigrationen funktionieren auf leerem Schema.
- [ ] der Worker verarbeitet abgelaufene Meldungen idempotent.
- [ ] Redis-Ausfall führt nicht zu Datenverlust bei Meldungen.
- [ ] externe Upstream-Fehler haben Timeout und verständlichen Fallback.
- [ ] Rate-Limits sind aktiv und automatisiert getestet.
- [ ] kein öffentlicher Endpunkt gibt interne oder private Felder aus.

### Betrieb

- [ ] lokale Installation ist dokumentiert.
- [ ] Produktions-HTTPS ist dokumentiert.
- [ ] Backup und Restore wurden praktisch getestet.
- [ ] Health Checks und Logs sind vorhanden.
- [ ] die Annahme neuer Meldungen kann im Notfall abgeschaltet werden.

### Datenschutz und Sicherheit

- [ ] kein permanenter Geräte-Hash im Kernablauf.
- [ ] keine Nutzer-GPS-Daten im MVP.
- [ ] Aufbewahrungs- und Löschfristen sind festgelegt.
- [ ] Karten- und Fahrplandatenanbieter sind dokumentiert.
- [ ] Threat Model und technische Datenschutzdokumentation sind vorhanden.
- [ ] eine unabhängige Prüfung beziehungsweise ein Review ist vor einem größeren öffentlichen Betrieb eingeplant.

---

## 21. Offene Risiken

| Risiko | Auswirkung | Nächste Maßnahme |
| --- | --- | --- |
| Zu wenige aktuelle Meldungen | Karte ist leer oder nicht nützlich | Pilot und Messung der Meldungsdichte vor großem Ausbau |
| Viele falsche Meldungen | Vertrauensverlust | Moderation, Ablauf, Rate-Limits und Qualitätsmetriken |
| Externe API ändert Bedingungen | Abfahrtsfunktion fällt aus | Adapter, Cache, Attribution und Fallback |
| Karten-/Netzdatenlizenz unklar | rechtliches und technisches Risiko | Quelle und Lizenz vor Import dokumentieren |
| Unklare Rechtslage der Kontrollmeldungen | Veröffentlichungsrisiko | rechtliche Prüfung vor öffentlichem Betrieb |
| Geräte-Hash wird als „anonym“ missverstanden | Profilbildung und falsche Sicherheit | kein permanenter Hash im MVP |
| PostGIS zu früh eingeführt | unnötige Komplexität | zunächst Netzdaten-IDs verwenden |
| WebSockets ohne Bedarf | schwerer Betrieb | Polling/SSE messen, erst dann aufrüsten |
| zu großer IHK-Umfang | unfertiger Abschluss | Must/Should/Cut-Scope verbindlich machen |

---

## 22. Definition of Done

Der technische MVP ist erst fertig, wenn:

1. die Anwendung lokal reproduzierbar startet,
2. ein versioniertes Verkehrsnetz importiert ist,
3. eine Meldung serverseitig validiert und gespeichert wird,
4. die Meldung in Liste und Karte erscheint,
5. Alter und Ablauf korrekt dargestellt werden,
6. abgelaufene Meldungen nicht mehr öffentlich erscheinen,
7. Markieren und Moderieren funktioniert,
8. Rate-Limits und Eingabegrenzen aktiv sind,
9. Tests den vollständigen Kernablauf abdecken,
10. externe Fehler kontrolliert behandelt werden,
11. Backup und Restore nach Anleitung funktionieren,
12. Datenschutz- und Sicherheitsdokumentation zum tatsächlichen Code passt,
13. bekannte Nichtziele und Restrisiken in der README stehen.

---

## 23. Nächster konkreter Schritt

Nicht mit WebSockets, PostGIS oder Geräte-Hashes beginnen.

Der nächste Schritt ist ein kurzer **Sprint 0** mit drei Ergebnissen:

1. Startstadt und Verkehrsnetz festlegen.
2. Entscheiden, ob externe Abfahrtsdaten für den MVP wirklich benötigt werden.
3. Eine verbindliche Entscheidung für den MVP-Scope und den Startstack dokumentieren.

Erst danach lohnt sich das Anlegen von Docker-Compose, FastAPI und Datenbankmigrationen.
