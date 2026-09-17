# Pendler-Radar – technischer Umsetzungsplan

**Status:** überarbeiteter Entwurf, noch nicht implementiert  
**Version:** 0.3
**Stand:** 17. September 2026

## Kurzfassung

1. **Daten-Spike zuerst:** Echte Stationen in Münster gegen `db.transport.rest`, DB Timetables API und DELFI/GTFS prüfen. Vollständigkeit, Aktualität, Fehlerverhalten und Lizenzen dokumentieren.
2. **Walking Skeleton:** FastAPI mit `/api/v1/departures/{station_id}` und ein minimales Frontend. Docker Compose, aber zunächst ohne Redis, PostgreSQL und Karte.
3. **Caching:** Redis, Request-Coalescing und adaptive TTL. Locust belegt, dass 100 parallele Anfragen höchstens einen Upstream-Request auslösen.
4. **Harmlose Meldungen:** Stationsbezogen „Aufzug defekt“, „überfüllt“ und „Störung“ per Tipp melden; Redis-TTL; Station aus dem aktuellen Kontext vorauswählen.
5. **Bestätigungen und Anreize:** Einmal pro Sitzung „Noch aktuell? Ja / Nein“. Kein Geld, keine Punkte, kein öffentliches Ranking – nur verständliches Feedback zur Wirkung einer Meldung.
6. **Spam-Schutz:** Zufallstoken statt Geräte-Hash, kurze Lebensdauer, Rate-Limits und Quarantäne. Rechtliche Prüfung der erforderlichen Speicherung einschließlich § 25 Abs. 2 Nr. 2 TDDDG.
7. **Kontrollkategorie später:** `ticket_inspection` ist nicht im MVP. Erst eine eigene rechtliche, strategische und communitybezogene Entscheidung.

Nach dem Walking Skeleton existiert bereits ein vorzeigbares Produkt, unabhängig davon, ob die spätere Meldeplattform oder die Kontrollkategorie umgesetzt wird.

---

Dieser Plan überarbeitet die erste Rohfassung des Projekts. Er trennt bewusst zwischen:

- dem fachlichen Produktziel,
- einem realistisch umsetzbaren MVP,
- technischen Entscheidungen,
- späteren Ausbaustufen,
- offenen Fragen, die vor dem produktiven Betrieb geklärt werden müssen.

## Leitentscheidung: Daten zuerst, Infrastruktur danach

Die größte Unbekannte ist nicht Docker, FastAPI oder Redis, sondern die Datenbasis. Wenn Stations-, Linien- und Abfahrtsdaten nicht vollständig, aktuell, erreichbar oder rechtssicher nutzbar sind, ist jedes Infrastruktur-Setup davor verschwendet.

Darum beginnt das Projekt mit einem kleinen Daten-Spike gegen echte Stationen. Erst wenn eine belastbare Quelle gefunden ist, entsteht ein Walking Skeleton. Redis, PostgreSQL, Karte und Echtzeitkanal kommen erst hinzu, wenn der vorherige Schritt einen konkreten Bedarf belegt.

## Produktentscheidung: harmlose Meldungen zuerst

Der MVP konzentriert sich auf stationsbezogene, unkritische Meldungen:

- `elevator_out_of_order`: Aufzug oder Rolltreppe defekt
- `crowded`: Zug oder Station ungewöhnlich überfüllt
- `disruption`: sichtbare Störung oder Ausfall

Die Kategorie `ticket_inspection` gehört **nicht** in den MVP. Sie wird erst nach einer separaten rechtlichen, strategischen und communitybezogenen Prüfung betrachtet. Technisch wäre sie später lediglich eine zusätzliche Kategorie; fachlich ist sie eine eigene Produktentscheidung.

Die erste Meldung wird mit einem Tipp und möglichst wenig Text erstellt. Die aktuelle Station soll aus dem Stations- oder Abfahrtskontext vorausgewählt werden. Kurzlebige Meldungen werden zunächst mit Redis-TTL umgesetzt. Eine dauerhafte Datenbank kommt erst hinzu, wenn Moderation, Auswertung oder längerfristige Betriebsanforderungen dies rechtfertigen.

Die wichtigste Änderung gegenüber der Rohfassung lautet:

> **Wir bauen zunächst keinen Verbund aus Microservices, sondern einen modularen Monolithen mit einem getrennten Hintergrundprozess.**
>
> Damit bleiben Entwicklung, Test und Betrieb beherrschbar. Die fachlichen Grenzen bleiben trotzdem sauber, sodass einzelne Komponenten später getrennt werden können, wenn es dafür einen nachgewiesenen Grund gibt.

---

## 1. Ziel des Projekts

Pendler-Radar soll aktuelle, gemeinschaftlich gemeldete Hinweise im Schienenverkehr anzeigen. Im MVP geht es zunächst um harmlose, stationsbezogene Meldungen:

1. Eine Person öffnet den Kontext einer Station oder ruft deren Abfahrten ab.
2. Die Station ist im Meldeformular vorausgewählt.
3. Die Person tippt eine Kategorie wie „Aufzug defekt“, „überfüllt“ oder „Störung“ an.
4. Die Anwendung prüft die Eingabe, begrenzt Missbrauch und speichert eine kurzlebige Meldung.
5. Andere Fahrgäste sehen die Meldung mit sichtbarem Alter.
6. Beim späteren Abruf kann die Anwendung höchstens einmal pro Sitzung fragen, ob sie noch aktuell ist.
7. Nach der TTL verliert die Meldung automatisch ihre Aktualität.

Die Kategorie `ticket_inspection` ist kein MVP-Bestandteil. Sie wird erst in einer eigenen Entscheidung nach rechtlicher und strategischer Prüfung betrachtet. Die Anwendung ist keine offizielle Auskunft eines Verkehrsunternehmens. Meldungen können falsch oder veraltet sein.

### Produktprinzipien

- **Orte im Schienennetz statt Nutzer-GPS:** Eine Meldung bezieht sich auf einen Bahnhof, einen Haltepunkt oder einen Streckenabschnitt aus den Netzdaten.
- **Zeitlich begrenzte Information:** Eine alte Meldung darf nicht wie eine aktuelle Warnung wirken.
- **Datensparsamkeit:** Keine dauerhaften Geräteprofile und keine unnötigen Standortdaten.
- **Sachlichkeit:** Keine Namen, Fotos, Audioaufnahmen oder Identifizierung von Personen.
- **Offene Unsicherheit:** Eine Community-Meldung ist keine amtliche Bestätigung.
- **Nachvollziehbarkeit:** Datenflüsse, Aufbewahrung und Betriebsgrenzen werden dokumentiert.
- **Daten zuerst:** Erst Datenquelle und Lizenz prüfen, dann Infrastruktur aufbauen.
- **Harmloser Start:** Kontrollmeldungen werden nicht heimlich in den MVP geschoben.

### Verbindlicher Geltungsbereich: nur Züge

Pendler-Radar umfasst ausschließlich den **Schienenverkehr mit Zügen**. Dazu gehören U-Bahn, S-Bahn, Regionalbahn und Regional-Express. Fernverkehr kann nach ausdrücklicher Entscheidung für das Startnetz ebenfalls enthalten sein.

Busse und Straßenbahnen sind nicht Teil des MVP. Sie werden nicht in die erste Netzkonfiguration, die Fahrplandatenquelle oder die fachlichen Zugarten aufgenommen. Eine spätere Erweiterung wäre eine neue Produktentscheidung.

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

## 3. MVP-Umfang und Prioritäten

### 3.1 Was in den MVP gehört

- Daten-Spike gegen echte Stationen und mehrere mögliche Datenquellen
- minimales Frontend mit Stationsauswahl und Abfahrtsliste
- FastAPI-Endpunkt `/api/v1/departures/{station_id}`
- reproduzierbarer Docker-Compose-Start für das Walking Skeleton
- Redis-Cache mit Request-Coalescing und TTL, sobald der Lasttest den Bedarf zeigt
- harmlose stationsbezogene Kategorien:
  - `elevator_out_of_order`
  - `crowded`
  - `disruption`
- Station aus dem aktuellen Stations- oder Abfahrtskontext vorauswählen
- Meldungen mit kurzer TTL speichern
- einmalige Frage pro Sitzung, ob eine offene Meldung noch aktuell ist
- einfache Bestätigung ohne Punkte oder Geld
- Basis-Rate-Limit und Schutz vor offensichtlichem Spam
- automatisierte Tests für Cache, Meldungen, Ablauf und Upstream-Fehler

### 3.2 Sinnvoll, wenn Zeit bleibt

- PostgreSQL erst für dauerhafte Moderation, Auswertung oder Betriebsdaten
- Server-Sent Events für schnellere Aktualisierung
- Stale-While-Revalidate für nicht kritische Abfahrtsdaten
- Locust-Lasttest und dokumentierter Nachweis, dass 100 parallele Nutzer:innen nur einen Upstream-Request auslösen
- mehrere Datenquellen hinter einem Adapter
- Barrierefreiheit, Kartenansicht und weitere Netze

### 3.3 Ausdrücklich nach dem MVP

- `ticket_inspection` beziehungsweise Meldungen zu Fahrkartenkontrollen
- rechtliche und strategische Bewertung dieser Kategorie
- dauerhafter Trust-Score
- automatische Routenempfehlungen zur Umgehung von Kontrollen
- Telegram-Import
- Push-Benachrichtigungen
- native Apps
- Microservice-Verbund mit mehreren unabhängig deploybaren Fachservices
- Geräte-Fingerprinting und Nutzer-GPS

Die Reihenfolge ist absichtlich daten- und ergebnisorientiert: Nach dem Walking Skeleton existiert bereits ein vorzeigbares Produkt, auch wenn spätere Meldungs- und Moderationsfunktionen verworfen werden sollten.

---

## 4. Offene Produktfragen vor der Implementierung

Diese Fragen sind wichtiger als die Wahl zwischen Vue, React oder Svelte:

1. **Welche Stadt startet?** Berlin, Dresden oder ein anderes Netz?
2. **Welche Zugarten gehören zum Startumfang?** U-Bahn, S-Bahn, Regionalbahn, Regional-Express und/oder Fernverkehr? Busse und Straßenbahnen gehören ausdrücklich nicht zum Umfang.
3. **Welche harmlosen Meldungskategorien starten?** Aufzug defekt, überfüllt, Störung und welche weiteren Kategorien sind sinnvoll?
4. **Wie lange gilt eine Meldung als aktuell?** Ein einheitlicher Wert oder abhängig von der Kategorie beziehungsweise Zugart?
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

### Name und Außenwirkung

„Radar“ kann Assoziationen mit einem Kontrollradar oder einer App zum Umgehen von Kontrollen wecken. Das ist eine bewusste Produktentscheidung und darf nicht nebenbei passieren. Vor einer öffentlichen Veröffentlichung sollten geprüft werden:

- Marken- und Namenskonflikte beim DPMA,
- App-Store-Namen und ähnliche Anwendungen,
- Domain und Social-Media-Namen,
- Verständlichkeit für die harmlose MVP-Ausrichtung.

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

Dieses Diagramm beschreibt den Zielzustand nach den Daten- und Cache-Spikes. Das Walking Skeleton startet bewusst kleiner: FastAPI, minimales Frontend und der geprüfte Upstream. Redis kommt beim Cache-Nachweis hinzu; PostgreSQL erst bei dauerhaftem Speicher- oder Moderationsbedarf.

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

### 7.0 Daten-Spike vor Infrastruktur

Der erste technische Arbeitsschritt ist kein Docker-Setup, sondern ein kleines Prüfprogramm gegen echte Stationen. Münster eignet sich als überschaubares Testnetz; die Stadt wird dadurch noch nicht als spätere Produkt- oder Startstadt festgelegt.

Kandidaten für den Vergleich:

- `db.transport.rest` für Abfahrts- und Stationsabfragen,
- DB Timetables API, sofern Zugang und Nutzungsbedingungen passen,
- DELFI/GTFS-Daten für statische Linien-, Stations- und Fahrplandaten.

Das Prüfprogramm soll für eine feste Liste realer Stationen erfassen:

- ob die Station gefunden wird,
- ob Abfahrten vollständig und aktuell wirken,
- wie schnell und stabil die Antwort kommt,
- wie sich Ausfälle, leere Antworten und Rate-Limits verhalten,
- welche Lizenz, Attribution und Nutzungsgrenze gilt,
- ob die Quelle für einen selbst betriebenen Dienst verwendet werden darf.

Ergebnis des Spikes ist eine kurze Vergleichsdokumentation mit Rohdaten, Messwerten und einer Entscheidung für den Walking Skeleton. Eine Quelle darf nicht nur gewählt werden, weil sie im Browser eine Antwort liefert.

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
- `category`: `elevator_out_of_order`, `crowded` oder `disruption`
- `station_id`: Station oder Bahnhof; im MVP erforderlich
- `mode`: optionale Zugart
- `line_id`: optionale Zuglinie
- `segment_id`: optionaler Streckenabschnitt
- `direction`: optionale, kontrollierte Fahrtrichtung
- `observed_at`
- `created_at`
- `expires_at`
- `status`
- optionaler, kurzer, bereinigter Hinweistext
- `source`, zum Beispiel `web`

Die Meldung wird stationsbezogen gespeichert. Der Stationskontext kann aus der vorherigen Abfahrtsabfrage stammen; eine freie GPS-Position ist nicht erforderlich.

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

### Zufallstoken für Spam-Schutz

Falls Rate-Limits über eine Sitzung hinaus erforderlich sind, kann der Server ein zufälliges, undurchsichtiges Token ausgeben. Es ist kein Hash, keine Geräte-ID und keine Authentifizierung.

- Token serverseitig zufällig erzeugen, nicht aus Gerätewerten ableiten.
- Token nur für Missbrauchsschutz und Rate-Limits verwenden.
- kurze Lebensdauer und Rotation vorsehen.
- Token nicht öffentlich ausgeben und nicht als Nutzerprofil verwenden.
- Löschung und technische Logs getrennt dokumentieren.

Ob die Speicherung eines solchen Tokens im konkreten Setup als unbedingt erforderlich behandelt werden kann und ob eine Ausnahme nach § 25 Abs. 2 Nr. 2 TDDDG greift, muss rechtlich geprüft werden. Das ist keine automatische Freistellung und keine Rechtsberatung.

### Consent und TDDDG

Ein Consent-Banner ist nicht zwingend für jede Kernfunktion. Zuerst wird getrennt, was für die Funktion unbedingt erforderlich ist und was optional ist.

- Der Kern der öffentlichen Ansicht soll ohne nicht erforderliches Tracking funktionieren.
- Ein notwendiger Anti-Abuse-Token ist kein Freibrief für weitere Analyse oder Werbung.
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
- weitere Beobachtungen im gleichen Zeitfenster.

Eine Meldung darf bei Auffälligkeit quarantänisiert werden. Neue Meldungen werden nicht grundsätzlich unsichtbar gemacht.

### 10.3 Mehrheitsbestätigung und Kaltstart

Für harmlose Meldungen kann später eine Mehrheitsbestätigung erprobt werden. Mehrere technisch verschiedene, kurzlebige Tokens können eine Meldung unabhängig voneinander als „noch aktuell“ bestätigen.

Dabei gilt:

- „unabhängig“ darf nicht als sicher bewiesene Personenunabhängigkeit bezeichnet werden.
- Mehrere Tokens können von einer Person erzeugt oder koordiniert werden.
- Bestätigungen sind ein Qualitätssignal, keine amtliche Wahrheit.
- Die konkrete Schwelle wird mit Pilotdaten bestimmt, nicht blind als `2 von 3` festgeschrieben.
- Eine neue Meldung wird zum Kaltstart nicht grundsätzlich blockiert.
- Die Oberfläche kann eine neue Meldung als „noch unbestätigt“ kennzeichnen.
- Tokens werden nicht zu einem dauerhaften Nutzer-Trust-Score zusammengeführt.

### 10.4 Anreize ohne Fehlanreize

Die Anwendung zahlt im MVP kein Geld und vergibt keine Punkte pro Meldung. Stattdessen erhält die meldende Person unmittelbares Feedback:

- Meldung angenommen, abgelehnt oder abgelaufen,
- Bestätigung durch weitere Sitzungen,
- sichtbare Wirkung auf die aktuelle Stationsansicht,
- verständlicher Grund bei Quarantäne oder Entfernung.

Ein späterer „Beitrag wirkt“-Hinweis darf nur aggregiert und datensparsam sein. Er darf kein öffentliches Ranking und kein dauerhaftes Verhaltensprofil voraussetzen.

---

## 11. Echtzeit-Aktualisierung

### 11.1 Stufe 1: Polling

Für den MVP wird zunächst ein periodischer Abruf empfohlen. Ein Intervall von etwa 30 Sekunden ist ein Startwert und wird gegen Datenfrische, Last und Batterieverbrauch gemessen:

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

### 11.4 Deep-Links zu offiziellen Apps

Ein Deep-Link wie `dbnavigator://` ist keine stabile Plattform-Schnittstelle. Vor einer Aufnahme in die Oberfläche muss für iOS, Android, Browser und PWA geprüft werden:

- ob das Schema dokumentiert oder nur beobachtet ist,
- ob die App installiert ist,
- ob der Browser den Link blockiert,
- ob ein sicherer Web-Fallback vorhanden ist,
- ob keine privaten Daten in die URL gelangen.

Deep-Links sind daher nicht Bestandteil des Walking Skeleton. Zuerst wird mit echten Geräten getestet; bis dahin verlinkt die Anwendung auf eine normale, offizielle Web-Adresse.

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
- Plausibilität wird über Station, optional Linie und Richtung sowie Zeit geprüft.
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
| `GET` | `/api/v1/departures/{station_id}` | Kernendpunkt des Walking Skeleton; begrenzter Fahrplan-Adapter |

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
  "category": "elevator_out_of_order",
  "stationId": "example-station",
  "mode": "s_bahn",
  "lineId": "s2",
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

Die Reihenfolge ist absichtlich nicht „erst alle Container, dann sehen wir weiter“. Jeder Schritt muss ein eigenes Ergebnis liefern.

### Sprint 0 – Entscheidungen an einem Abend

**Ziel:** Der Umfang ist klein genug, um ihn wirklich zu bauen.

Aufgaben:

- MVP-Kategorien festlegen: Aufzug defekt, überfüllt, Störung
- `ticket_inspection` ausdrücklich aus dem MVP ausschließen
- Zugumfang bestätigen: U-Bahn, S-Bahn, Regionalverkehr, optional Fernverkehr
- Zielstadt vom Testnetz Münster unterscheiden
- Rollen klären: Entwicklung, Datenquellen, Moderation, Betrieb
- Frontend-Stack nach vorhandenem Können auswählen
- GitHub-Issues für offene Entscheidungen, Quellen und Risiken anlegen
- Definition of Done und Zeitbudget festhalten

**Ergebnis:** Ein einseitiger MVP-Entscheid, der von allen Beteiligten bestätigt ist.

### Sprint 1 – Daten-Spike

**Ziel:** Vor Infrastruktur ist bekannt, ob brauchbare Daten existieren.

Aufgaben:

- kleine Testskripte gegen echte Stationen in Münster schreiben
- `db.transport.rest` prüfen
- DB Timetables API prüfen
- DELFI/GTFS für statische Netz- und Fahrplandaten prüfen
- Vollständigkeit, Aktualität, Antwortzeit und Fehlerfälle vergleichen
- Rate-Limits und Nutzungsbedingungen beobachten
- Lizenz, Attribution und Weiterverwendung dokumentieren
- mindestens eine Station ohne, mit leerer und mit fehlerhafter Antwort testen
- Datenquelle für Walking Skeleton und Datenquelle für statische Netzdaten getrennt bewerten

**Ergebnis:** Ein kurzer Datenbericht mit Messwerten, Beispielantworten, Lizenznotizen und einer begründeten Quellenauswahl.

Münster ist dabei ein Testnetz für den Spike. Daraus folgt noch nicht, dass Münster die spätere Startstadt wird.

### Sprint 2 – Walking Skeleton

**Ziel:** Nach wenigen Tagen existiert ein vorzeigbares, kleines Produkt.

Aufgaben:

- FastAPI-Dienst starten
- `/api/v1/health` und `/api/v1/departures/{station_id}` implementieren
- erlaubte Stations-IDs validieren
- Upstream-Timeout und verständliche Fehlerantworten einbauen
- minimales Frontend mit Stationsauswahl und Abfahrtsliste erstellen
- Docker Compose nur mit den notwendigen Komponenten aufsetzen
- Caddy optional als einfacher Reverse-Proxy für die Demo nutzen
- zunächst ohne Redis, PostgreSQL und Kartenintegration arbeiten
- einfache Logs ohne vollständige Nutzereingaben schreiben

**Ergebnis:** Eine Station kann ausgewählt werden und aktuelle Abfahrten werden angezeigt. Dieses Ergebnis ist bereits demonstrierbar, auch wenn die spätere Meldeplattform nicht umgesetzt werden sollte.

### Sprint 3 – Caching und Lastnachweis

**Ziel:** Die externe Datenquelle wird nicht durch parallele Anfragen belastet.

Aufgaben:

- Redis hinzufügen
- Cache-Schlüssel aus Quelle, Netz, Station, Produkten und Zeitfenster bilden
- frische Cache-Treffer sofort ausliefern
- Request-Coalescing mit kurzem, token-basiertem Lock implementieren
- adaptive TTL für nahe und spätere Abfahrten messen
- Stale-While-Revalidate nur einsetzen, wenn veraltete Daten fachlich vertretbar sind
- Upstream-Timeout, Lock-Ablauf und Redis-Ausfall testen
- Locust-Lasttest mit 100 parallelen Nutzer:innen durchführen

**Abnahmekriterium:** Bei 100 parallelen Anfragen auf denselben Cache-Miss entsteht höchstens ein Upstream-Request. Dieses Ergebnis wird im Testbericht mit Metriken belegt.

### Sprint 4 – Harmlose stationsbezogene Meldungen

**Ziel:** Die erste Schreibfunktion ist nützlich, datensparsam und unkritisch.

MVP-Kategorien:

- `elevator_out_of_order`
- `crowded`
- `disruption`

Aufgaben:

- Station aus dem aktuellen Stations- oder Abfahrtskontext vorauswählen
- Kategorie per Tipp auswählen lassen
- optional Linie, Zugart, Richtung und kurze Ergänzung erfassen
- Meldungen stationsbezogen mit Redis-TTL speichern
- öffentliche Liste um Meldungsalter und Ablauf ergänzen
- Karte erst nach der Liste und nur mit öffentlichen Stationskoordinaten hinzufügen
- beim Stationsabruf höchstens einmal pro Sitzung fragen: „Noch aktuell? Ja / Nein“
- Bestätigung ohne Punkte oder Geld speichern beziehungsweise aggregiert auswerten
- Meldende über Annahme, Ablauf und Wirkung informieren
- keine Kontrollkategorie in die Eingabemaske aufnehmen

**Ergebnis:** Eine Station kann eine harmlose, automatisch ablaufende Meldung erhalten. Andere Nutzer:innen sehen sie, bestätigen sie einmalig und sehen ihren Aktualitätsstatus.

### Sprint 5 – Spam-Schutz und Kaltstart

**Ziel:** Die offene Schreibfunktion bleibt trotz fehlender Konten beherrschbar.

Aufgaben:

- Rate-Limits nach Endpunkt und kurzer technischer Quelle
- zufälliges, undurchsichtiges Token prüfen; kein Geräte-Hash und keine Identität
- Token nur für Missbrauchsschutz, nicht für Ranking oder Profile
- kurze Lebensdauer, Rotation und Löschung dokumentieren
- IPs in Reverse-Proxy- und API-Logs mit eigener kurzer Löschfrist behandeln
- rechtlich prüfen, ob erforderliche Token-Speicherung unter § 25 Abs. 2 Nr. 2 TDDDG fallen kann
- bei Bedarf Bestätigungen durch mehrere technisch verschiedene, kurzlebige Tokens testen
- neue Meldungen beim Kaltstart sichtbar, aber als unbestätigt kennzeichnen
- keine technisch nicht beweisbare „Unabhängigkeit“ als Fakt behaupten
- Quarantäne für auffällige Serien

**Ergebnis:** Ein Spam-Szenario wird begrenzt, ohne einen dauerhaften Trust-Score aufzubauen und ohne die erste Meldung unsichtbar zu machen.

### Sprint 6 – PostgreSQL und dauerhafte Moderation, falls erforderlich

**Ziel:** Erst jetzt wird aus dem TTL-Prototyp ein dauerhafter, betreibbarer Dienst.

PostgreSQL wird nur aufgenommen, wenn mindestens einer dieser Gründe belegt ist:

- Moderationsentscheidungen müssen über Redis-TTL hinaus nachvollziehbar bleiben.
- Berichte und Datenqualität müssen ausgewertet werden.
- mehrere Worker brauchen eine dauerhafte Quelle der Wahrheit.
- Aufbewahrung, Löschung und Backup müssen über den Prototyp hinaus betrieben werden.

Aufgaben:

- Datenmodell und Migrationen
- Statusmaschine für `received`, `published`, `expired`, `quarantined` und `removed`
- Moderationswarteschlange
- Lösch- und Aufbewahrungsjobs
- Backup und Restore
- Integrations- und End-to-End-Tests

### Sprint 7 – Kontrollkategorie als eigene Entscheidung

**Ziel:** Nicht automatisch aus der Inspiration ein rechtlich und strategisch anderes Produkt machen.

Aufgaben vor einer Implementierung von `ticket_inspection`:

- rechtliche Risiken und Verantwortlichkeiten prüfen
- strategisches Ziel der Kategorie schriftlich entscheiden
- Community- und Moderationsregeln erweitern
- Missbrauchs- und Bedrohungsmodell aktualisieren
- Ablaufzeit und Darstellung separat festlegen
- prüfen, ob die Kategorie mit der gewählten Marke und Außenwirkung vereinbar ist
- Entscheidung als eigenes Issue beziehungsweise ADR dokumentieren

**Ergebnis:** Entweder bleibt die Kategorie außerhalb des Produkts oder sie wird bewusst als eigener, geprüfter Scope aufgenommen. Sie wird nicht nur als zusätzlicher Enum-Wert „nebenbei“ aktiviert.

### Sprint 8 – Betrieb und Abschluss

- HTTPS-Reverse-Proxy dokumentieren
- Monitoring für Upstream, Cache, Meldungen und Ablauf einrichten
- Backup und Restore praktisch testen
- Sicherheits- und Datenschutzdokumente gegen den Code prüfen
- Barrierefreiheit und mobile Bedienung prüfen
- Last- und Ausfalltests wiederholen
- Pilotphase mit Feedbackmöglichkeit durchführen
- weitere Zugnetze erst nach Datenqualitätsprüfung hinzufügen

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
4. Kontrollkategorie `ticket_inspection`,
5. Trust-Score,
6. Mehrstadtbetrieb,
7. komplexe Nutzerkonten,
8. räumliche PostGIS-Analysen, wenn Stations- und Netzdaten-IDs ausreichen.

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

- [ ] Eine geprüfte Datenquelle liefert echte Stations- und Abfahrtsdaten.
- [ ] Eine Station kann ausgewählt und in einer Minimaloberfläche angezeigt werden.
- [ ] Eine harmlose Kategorie kann ohne Geräte-GPS erstellt werden.
- [ ] Die Station ist vorausgewählt; Linie und Richtung sind optional und werden nur aus Netzdaten akzeptiert.
- [ ] Beobachtungszeit und Eingangszeit bleiben getrennt.
- [ ] Das Ablaufdatum wird serverseitig gesetzt und über Redis-TTL durchgesetzt.
- [ ] Abgelaufene Meldungen verschwinden aus der aktuellen Ansicht.
- [ ] Ein Stationsabruf fragt höchstens einmal pro Sitzung nach einer Bestätigung.
- [ ] Eine Meldung kann markiert und moderiert werden, sofern Moderation bereits dauerhaft gespeichert wird.
- [ ] `ticket_inspection` ist im MVP nicht auswählbar.

### Technisch

- [ ] API-Fehler haben stabile Fehlercodes.
- [ ] Externe Upstream-Fehler haben Timeout und verständlichen Fallback.
- [ ] Ein Locust-Test mit 100 parallelen Anfragen erzeugt bei einem Cache-Miss höchstens einen Upstream-Request.
- [ ] Lock-Ablauf und Redis-Ausfall sind getestet.
- [ ] Rate-Limits sind aktiv und automatisiert getestet.
- [ ] kein öffentlicher Endpunkt gibt interne oder private Felder aus.
- [ ] Falls PostgreSQL eingeführt wird: Migrationen, Löschfristen, Backup und Restore sind getestet.

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

1. eine geprüfte Datenquelle echte Stationen und Abfahrten liefert,
2. die Anwendung lokal reproduzierbar startet,
3. eine Station ausgewählt und angezeigt werden kann,
4. eine harmlose Meldung serverseitig validiert und mit TTL gespeichert wird,
5. Station, Kategorie, Alter und Ablauf korrekt dargestellt werden,
6. abgelaufene Meldungen nicht mehr öffentlich erscheinen,
7. die Bestätigung „Noch aktuell?“ höchstens einmal pro Sitzung erfolgt,
8. Rate-Limits und Eingabegrenzen aktiv sind,
9. der Cache- und Upstream-Fehlerfall getestet ist,
10. Tests den vollständigen Kernablauf abdecken,
11. `ticket_inspection` nicht Teil des MVP ist,
12. Datenschutz- und Sicherheitsdokumentation zum tatsächlichen Code passt,
13. bekannte Nichtziele und Restrisiken in der README stehen.

Wenn PostgreSQL für Moderation oder Auswertung in den MVP aufgenommen wird, kommen Migration, Backup, Restore und Löschprüfung als zusätzliche Abnahmekriterien hinzu.

---

## 23. Nächster konkreter Schritt

Nicht mit WebSockets, PostGIS, Geräte-Hashes oder Microservices beginnen.

Der nächste Schritt ist der **Daten-Spike**:

1. eine kleine Stationsliste für Münster festlegen,
2. `db.transport.rest`, DB Timetables API und DELFI/GTFS mit Testskripten vergleichen,
3. Vollständigkeit, Aktualität, Latenz, Fehlerfälle und Lizenzen dokumentieren,
4. eine Datenquelle für das Walking Skeleton auswählen.

Erst danach wird FastAPI mit `/api/v1/departures/{station_id}` und einem minimalen Frontend aufgebaut. Redis kommt erst für den Cache- und Lastnachweis hinzu; PostgreSQL und die Kontrollkategorie bleiben spätere Entscheidungen.
