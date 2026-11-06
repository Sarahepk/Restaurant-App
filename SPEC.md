# SPEC.md – Restaurantmanagement „Bella Vista“

> **Projekt:** Digitale Verwaltung der Restaurants von Marco Ferretti  
> **Standorte:** Berlin-Kreuzberg und Berlin-Spandau  
> **Grundlage:** Interview 1 (12 Fragen) und Interview 2 (8 Fragen)  
> **Status:** Anforderungsentwurf – offene Punkte sind ausdrücklich gekennzeichnet

## 1. Ziel und Ausgangslage

Marco Ferretti betreibt zwei Restaurants. Reservierungen werden bislang überwiegend telefonisch in getrennten Heften erfasst; zusätzlich gibt es Reservierungsanfragen über Google. Tischzuweisungen beruhen häufig auf Erfahrungswissen. Bestellungen und Abrechnungen erfolgen weitgehend papierbasiert. Speisekarten werden standortweise in Word gepflegt und als gemeinsame PDF veröffentlicht. Stammgastbesuche werden unvollständig in getrennten Listen dokumentiert.

**Ziel:** Eine zentrale, auch mobil einsehbare Anwendung, die zunächst Reservierungen und Tischbelegung zuverlässig organisiert, anschließend Bestellungen und standortspezifische Speisekarten unterstützt und später weitere Funktionen ergänzt.

### Priorisierung

| Priorität | Funktionsbereich | Begründung |
|---|---|---|
| **P1 – zuerst** | Reservierungen, Tischzuweisung, Doppelbuchungsschutz, Standortübersicht | Von Marco als dringendstes Problem benannt |
| **P2 – danach** | Bestellungen und standortspezifische Speisekarten | Beide Prozesse hängen unmittelbar zusammen |
| **P3 – später** | Stammgastverwaltung und Bella-Card | Wichtig, aber nicht für den ersten Einsatz erforderlich |
| **P4 – nachrangig** | Mitarbeiter-/Schichtplanung | Von Marco als nicht dringend bezeichnet; noch nicht spezifiziert |
| **Offen** | Cateringverwaltung | Gewünscht, aber noch keiner Umsetzungsphase zugeordnet |

## 2. Datenmodell: 8 Entitäten

Die folgende Auswahl ist ein **konzeptioneller Modellierungsvorschlag**. Attribute mit `?` sind optional oder noch zu bestätigen.

### E1 – Standort

| Attribut | Bedeutung |
|---|---|
| `standort_id` (PK) | Eindeutiger Schlüssel |
| `name` | Kreuzberg / Spandau |
| `schliesszeit` | Kreuzberg 23:00 Uhr; Spandau 22:00 Uhr |
| `kuechenschluss` | Kreuzberg 22:30 Uhr; Spandau 21:30 Uhr |
| `hat_aussenplaetze` | Außenplätze sind für Kreuzberg bestätigt; Ausprägung in Spandau offen |

### E2 – Tisch

| Attribut | Bedeutung |
|---|---|
| `tisch_id` (PK) | Eindeutiger Schlüssel |
| `standort_id` (FK) | Zugehöriger Standort |
| `tisch_nummer` | Tischbezeichnung vor Ort |
| `kapazitaet` | Sitzplätze; konkrete Werte offen |
| `bereich?` | Beispielsweise Innen- oder Außenbereich |
| `belegungsstatus` | Frei / belegt; genaue Zustandslogik offen |

### E3 – Reservierung

| Attribut | Bedeutung |
|---|---|
| `reservierung_id` (PK) | Eindeutiger Schlüssel |
| `gast_id?` (FK) | Zugeordneter Gast, soweit bekannt |
| `datum` | Reservierungstag |
| `uhrzeit` | Geplante Ankunft |
| `personenanzahl` | Größe der Gruppe |
| `quelle` | Telefon / Google; Walk-in als gesonderter Eintrittsweg |
| `status?` | Beispielsweise angefragt, bestätigt, angekommen, No-show; Werte nicht vollständig bestätigt |
| `aussenplatz_wunsch?` | Wunsch nach Außenplatz |
| `besonderer_anlass?` | Mögliche Zusatzinformation aus dem Telefonat |

**Hinweis:** Eine Reservierung kann für größere Gruppen mehrere Tische benötigen. Die konkrete Dauer einer Reservierung ist noch nicht festgelegt.

### E4 – Gast / Kunde

| Attribut | Bedeutung |
|---|---|
| `gast_id` (PK) | Eindeutiger Schlüssel |
| `name` | Gastname bzw. Kundenbezeichnung |
| `telefonnummer?` | Bevorzugtes Suchmerkmal; kann fehlen |
| `besuchsanzahl` | Standortübergreifend gezählte Besuche |
| `bella_card_status` | Anspruch auf Bella-Card |
| `kundentyp?` | Privat / Unternehmen – **Modellierungsannahme** für Catering |

**Hinweis:** Anonyme Tischbestellungen sind zulässig. Sie erzeugen ohne Gastzuordnung keine Bella-Card-Besuchsgutschrift.

### E5 – Mitarbeiter

| Attribut | Bedeutung |
|---|---|
| `mitarbeiter_id` (PK) | Eindeutiger Schlüssel |
| `name` | Name des Mitarbeiters |
| `rolle?` | Beispielsweise Bedienung oder Küche; Rollenmodell offen |

Eine umfassende Schichtplanung ist als spätere Erweiterung erwähnt, aber nicht ausreichend beschrieben.

### E6 – Bestellung

| Attribut | Bedeutung |
|---|---|
| `bestellung_id` (PK) | Eindeutiger Schlüssel |
| `tisch_id` (FK) | Zugeordneter Tisch |
| `mitarbeiter_id` (FK) | Aufnehmende Bedienung |
| `gast_id?` (FK) | Optional für Stammgastzuordnung |
| `zeitpunkt` | Zeitpunkt der Bestellung |
| `status` | Offen / serviert / bezahlt / fertig |
| `zahlungsart?` | Bar / Karte |
| `gesamtbetrag?` | Summe der bestellten Positionen |

**Hinweis:** Die Bedeutung von „fertig“ nach „bezahlt“ ist noch nicht geklärt. Eine Bestellung kann mehrere Gerichte mit jeweiligen Mengen enthalten.

### E7 – Gericht

| Attribut | Bedeutung |
|---|---|
| `gericht_id` (PK) | Eindeutiger Schlüssel |
| `bezeichnung` | Name des Gerichts |
| `standardpreis` | Standortübergreifender Ausgangspreis |
| `kategorie?` | Beispielsweise Grillgericht oder Tagesgericht |

**Standortabhängige Daten:** Verfügbarkeit und gegebenenfalls abweichender Preis werden in der Beziehung `Standort_Gericht` gespeichert. Kreuzberg bietet Grillgerichte an, Spandau aufgrund baulicher Gegebenheiten nicht.

### E8 – Cateringauftrag

| Attribut | Bedeutung |
|---|---|
| `cateringauftrag_id` (PK) | Eindeutiger Schlüssel |
| `kunde_id` (FK) | Auftraggeber, gegebenenfalls Unternehmen |
| `veranstaltungsdatum` | Termin des Events |
| `lieferadresse` | Ort der Lieferung |
| `personenanzahl` | Anzahl der zu bewirtenden Personen |
| `menueauswahl` | Vereinbarte Speisen / Buffet-Auswahl; zunächst vereinfacht modelliert |
| `preis_pro_person?` | Angebotsgrundlage |
| `gesamtbetrag?` | Vereinbarter Gesamtpreis |
| `status` | Angefragt / bestätigt / geliefert / bezahlt |
| `angebotsversion?` | Vereinfachte Nachverfolgung von Überarbeitungen |
| `anzahlung_betrag?` | Erfasste Anzahlung |
| `anzahlung_datum?` | Eingang der Anzahlung |
| `restzahlung_datum?` | Eingang der Restzahlung |
| `lieferzeit?` | Noch zu bestätigender Lieferzeitpunkt |
| `lieferverantwortlich?` | Noch zu klärende Zuständigkeit |

**Belegtes Beispiel:** TechCorp, Weihnachtsfeier für 60 Personen in Berlin-Mitte; von Marco genannter Gesamtbetrag: 2.400 €. Das Angebot wurde mehrfach überarbeitet. Für eine spätere Detailversion wären separate Entitäten für Angebote, Angebotsversionen und Zahlungen denkbar; sie werden hier wegen der Begrenzung auf acht Entitäten **nicht** eingeführt.

## 3. Beziehungen: genau 9

| Nr. | Beziehung | Kardinalität | Erläuterung |
|---|---|---|---|
| R1 | Standort → Tisch | **1:n** | Ein Standort hat mehrere Tische; ein Tisch gehört zu einem Standort. |
| R2 | Reservierung ↔ Tisch | **n:m** | Gruppen können mehrere Tische benötigen; ein Tisch wird zu unterschiedlichen Zeiten reserviert. **Zeitliche Überschneidungen sind zu verhindern.** |
| R3 | Gast → Reservierung | **1:n**, optional auf Reservierungsseite | Ein bekannter Gast kann mehrere Reservierungen haben; Gastzuordnung ist nicht immer möglich. |
| R4 | Standort ↔ Gericht | **n:m** | Ein Gericht kann an mehreren Standorten angeboten werden; Verfügbarkeit und Preisabweichung sind standortbezogen. |
| R5 | Tisch → Bestellung | **1:n** | An einem Tisch können mehrere Bestellungen entstehen. |
| R6 | Mitarbeiter → Bestellung | **1:n** | Eine Bestellung wird einer aufnehmenden Bedienung zugeordnet. |
| R7 | Bestellung ↔ Gericht | **n:m** | Eine Bestellung enthält mehrere Gerichte; ein Gericht kommt in vielen Bestellungen vor. Beziehungsattribut: `menge`. |
| R8 | Gast → Bestellung | **1:n**, optional auf Bestellungsseite | Zugeordnete Bestellungen ermöglichen die Besuchserfassung; anonyme Bestellungen bleiben ohne Gast. |
| R9 | Gast/Kunde → Cateringauftrag | **1:n** | Ein Auftraggeber kann mehrere Cateringaufträge haben; die Verwendung der Gast-Entität für Firmenkunden ist eine **Modellierungsannahme**. |

**Technischer Hinweis:** Die n:m-Beziehungen werden in einer relationalen Datenbank über Zuordnungstabellen realisiert (z. B. `Reservierung_Tisch`, `Standort_Gericht`, `Bestellposition`). Diese Zuordnungstabellen werden im obigen **konzeptionellen** Modell nicht als zusätzliche Kernentitäten gezählt.

## 4. Business Rules: genau 5

| ID | Verbindliche Geschäftsregel | Quelle / Einordnung |
|---|---|---|
| **BR-01** | Ein registrierter Gast erhält nach **10 gezählten Besuchen** standortübergreifend die Bella-Card und damit **15 % Rabatt**. | Von Marco ausdrücklich bestätigt |
| **BR-02** | Pro Gast zählt **höchstens ein Besuch je Abend**, unabhängig von der Anzahl seiner Bestellungen. Ein nicht zugeordneter anonymer Besuch zählt nicht. | Von Marco nach Rückfrage ausdrücklich klargestellt |
| **BR-03** | Ein Gericht darf nur an einem Standort angeboten bzw. bestellt werden, an dem es verfügbar ist. Für gemeinsame Gerichte gilt ein **Standardpreis**, der standortbezogen überschrieben werden kann. | Ausdrücklich gewünschte Speisekartenlogik; als eine zusammenhängende Regel formuliert |
| **BR-04** | Erscheint zu einer Reservierung innerhalb von **2 Stunden nach der gebuchten Uhrzeit** kein Gast, wird die reservierte Tischbelegung automatisch freigegeben. | Marcos ausdrücklich genannte No-show-Regel |
| **BR-05** | Der **Küchenschluss liegt 30 Minuten vor dem Restaurantschluss** und ist pro Standort einstellbar. Aktuell: Kreuzberg 22:30 Uhr, Spandau 21:30 Uhr. | Von Marco ausdrücklich bestätigt |

**Zusätzliche funktionale Anforderung, keine weitere Business Rule:** Doppelbuchungen eines Tisches für überlappende Zeiträume sollen verhindert werden. Die dafür benötigte Reservierungsdauer und die Behandlung von Tischkombinationen müssen noch präzisiert werden.

## 5. Widersprüche: genau 2 – mit Auflösung

### W-01 – Reservierungen ausschließlich telefonisch?

- **Frühe Aussage (Meeting #1):** Reservierungen kommen ausschließlich per Telefon herein.
- **Spätere Aussage (Meeting #2):** Zusätzlich können Gäste über Google bzw. Google Maps Reservierungen anfragen.
- **Auflösung:** Telefon ist der **bevorzugte**, aber nicht der einzige Kanal. Die App soll **Telefon**, **Google-Anfrage** und **Walk-in** als Herkunft bzw. Eintrittsweg unterscheidbar erfassen.
- **Offen:** Ob Google-Anfragen automatisch übernommen oder manuell eingetragen werden und wann eine Anfrage als verbindlich bestätigt gilt.

### W-02 – Zählt jede Bestellung als Besuch?

- **Frühe Aussage (Meeting #1):** Marco beschrieb zunächst, dass jede einem bekannten Gast zugeordnete Bestellung den Besuchszähler erhöht.
- **Spätere Aussage nach gezielter Rückfrage (Meeting #1):** Mehrere Bestellungen desselben Gastes an einem Abend zählen **zusammen nur als ein Besuch**; Besuche aus beiden Standorten werden addiert.
- **Auflösung:** Für die Bella-Card wird **ein Besuch pro Gast und Abend** gezählt, nicht eine Gutschrift pro Bestellung. Auch manuell erfasste Besuche spontaner, identifizierter Gäste können berücksichtigt werden.
- **Einordnung:** Dies ist eine **inhaltliche Inkonsistenz der ursprünglichen Zählbeschreibung**, die durch Marcos Präzisierung aufgelöst wurde; sie ist weniger ausdrücklich als W-01.

## 6. Neue Anforderung aus Meeting #2: genau 1

### N-01 – Cateringverwaltung

Die **neu eingeführte Hauptanforderung** ist die Verwaltung von Catering für Firmenevents. Marco erwähnte sie erstmals auf die Frage nach seinen nächsten Plänen.

**Gewünschter Funktionsumfang:**

- Cateringauftrag mit Kunde, Veranstaltungsdatum, Lieferadresse, Personenzahl und Menüauswahl anlegen.
- Angebote mit Preis pro Person erstellen und Änderungen nachvollziehen.
- Auftragsstatus anzeigen: **angefragt → bestätigt → geliefert → bezahlt**.
- Anzahlungen und Restzahlungen einschließlich Eingangszeitpunkten nachvollziehen.
- Offene organisatorische Punkte wie endgültiges Menü und Lieferzuständigkeit sichtbar machen.

**Abgrenzung:** Die später angesprochene Schichtplanung wird hier **nicht als zweite neue Hauptanforderung** ausgewiesen, sondern als beiläufig erwähnte, nachrangige Erweiterung. Die Cateringverwaltung bleibt die genau eine neue Hauptanforderung aus Meeting #2.

## 7. Funktionale Anforderungen im Überblick

| ID | Anforderung | Priorität |
|---|---|---|
| F-01 | Reservierungen für beide Standorte zentral anzeigen und verwalten | P1 |
| F-02 | Tische ausdrücklich zuweisen und Doppelbuchungen verhindern | P1 |
| F-03 | Aktuelle Tischbelegung und verfügbare Plätze mobil anzeigen | P1 |
| F-04 | Walk-ins direkt erfassen und No-show-Freigabe unterstützen | P1 |
| F-05 | Herkunft eines Eintrags (Telefon, Google, Walk-in) anzeigen | P1 |
| F-06 | Bestellungen mit Tisch, Gerichten, Mengen und Bedienung erfassen | P2 |
| F-07 | Bestellstatus und Zahlungsart sichtbar machen | P2 |
| F-08 | Standortbezogene Speisekarten mit gemeinsamen Gerichten und Preisabweichungen pflegen | P2 |
| F-09 | Stammgäste über Telefonnummer **oder Namen** suchen | P3 |
| F-10 | Besuche automatisch oder bei Walk-ins manuell zuordnen und Bella-Card-Rabatt anwenden | P3 |
| F-11 | Cateringangebote und -aufträge samt Status und Zahlungen verwalten | Noch nicht priorisiert |
| F-12 | Schließ- und Küchenschlusszeiten standortweise einstellen | Funktionsumfang bestätigt; Umsetzungsphase offen |

## 8. Offene Fragen und Annahmen

1. **Reservierungsdauer:** Wie lange blockiert eine bestätigte Reservierung einen Tisch, und wie werden mehrere zusammengestellte Tische behandelt?
2. **Google-Anfragen:** Wie gelangen sie technisch in die App, und wer bestätigt sie?
3. **No-show:** Woran erkennt das System, ob Gäste tatsächlich eingetroffen sind?
4. **Späte Reservierungen:** Welche Gerichte sind zu welchen Zeiten noch bestellbar, und welche Hinweise sollen Gäste erhalten?
5. **Bestellstatus:** Was bedeutet „fertig“ im Unterschied zu „bezahlt“? Sind Stornierungen möglich?
6. **Besuchszählung:** Wie wird bei mehreren Bestellungen, mehreren Tischen oder Namensgleichheit eine doppelte Besuchsgutschrift vermieden?
7. **Catering:** Welche Angebotsversionen, Teilzahlungen und Lieferverantwortlichkeiten müssen im Detail dokumentiert werden? Wann soll das Modul umgesetzt werden?
8. **Kundenmodell:** Ob Firmenkunden und Restaurantgäste tatsächlich in einer gemeinsamen Entität geführt werden sollen, muss fachlich bestätigt werden.
9. **Schichtplanung:** Nur als spätere Idee erwähnt; keine hinreichend konkreten Anforderungen erhoben.

## 9. Abnahmekriterien für den ersten Ausbauschritt (Vorschlag)

- Reservierungen aus Kreuzberg und Spandau sind in einer zentralen Ansicht unterscheidbar.
- Einer Reservierung können geeignete Tische zugeordnet werden.
- Eine bereits belegte Tisch-Zeit-Kombination kann nicht versehentlich erneut verbindlich vergeben werden.
- Walk-ins sind im Belegungsbild sichtbar.
- Die Herkunft eines Eintrags kann nachvollzogen werden.
- Die Zwei-Stunden-No-show-Regel ist nach bestätigter Nichterscheinung anwendbar.

> **Modellierungsvermerk:** Diese Abnahmekriterien operationalisieren Marcos Wünsche. Einzelheiten wie die technische Anbindung an Google, die Zeitlogik der Tischbelegung und die exakte Statusmaschine müssen vor der Implementierung validiert werden.
