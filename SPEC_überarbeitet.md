# SPEC.md – Restaurantmanagement „Bella Vista“

> **Projekt:** Digitale Verwaltung der Restaurants von Marco Ferretti  
> **Standorte:** Berlin-Kreuzberg und Berlin-Spandau  
> **Grundlage:** Interview 1 (12 Fragen) und Interview 2 (8 Fragen)  
> **Status:** Anforderungsentwurf – offene Punkte sind ausdrücklich gekennzeichnet

## Quellenkonvention: Nummerierung der Interview-Nachrichten

Die Nummern bezeichnen **Marcos Antworten** (nicht einzelne Chatnachrichten zwischen uns). Die Zählung läuft über beide Meetings fort:

- **Meeting #1:** Nachrichten **1–12** = Antworten auf Fragen 1–12.
- **Meeting #2:** Nachrichten **13–20** = Antworten auf Fragen 1–8.

Beispiel: **Nachricht 8 (M1/F8)** bedeutet Marcos Antwort auf die achte Frage des ersten Meetings. Eine Quellenangabe ist kein Verweis auf eine externe Veröffentlichung, sondern auf das Interviewprotokoll. Bei zusammengefassten Regeln werden alle maßgeblichen Antworten angegeben.

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

## 4. Business Rules: genau 5 – mit konkreten Nachrichtennummern

| ID | Geschäftsregel | Nachricht(en) von Marco | Nachweis aus der Antwort |
|---|---|---|---|
| **BR-01** | Ein registrierter Gast erhält nach **10 gezählten Besuchen** an beiden Standorten zusammen die Bella-Card mit **15 % Rabatt**. | **7, 8** (M1/F7, M1/F8); Rabatt erstmals in **6** (M1/F6) | **7:** „10 Besuche“, „15 %“; **8:** Besuche aus Kreuzberg und Spandau werden zusammengerechnet. |
| **BR-02** | Pro Gast zählt **höchstens ein Besuch je Abend**, unabhängig von der Zahl seiner Bestellungen. Ein anonymer, nicht zugeordneter Besuch zählt nicht. | **8, 17** (M1/F8, M2/F5); Ausgangsaussage in **7** (M1/F7) | **8:** Ein Abend = ein Besuch, auch bei Nachbestellungen. **17:** Ohne Zuordnung zu einem Gastprofil keine Besuchsgutschrift. |
| **BR-03** | Ein Gericht darf nur an Standorten angeboten werden, an denen es verfügbar ist. Gemeinsame Gerichte besitzen einen **Standardpreis**, der je Standort überschrieben werden kann. | **9, 10** (M1/F9, M1/F10) | **9:** Grillgerichte nur in Kreuzberg; gemeinsame Gerichte zentral pflegen. **10:** Standardpreis und mögliche standortbezogene Preisabweichungen. |
| **BR-04** | Erscheint ein Gast nicht, wird die reservierte Tischbelegung **2 Stunden nach der gebuchten Uhrzeit** automatisch freigegeben. | **12** (M1/F12) | Beispiel von Marco: Reservierung 20:00 Uhr, bei Nichterscheinen Freigabe um 22:00 Uhr. |
| **BR-05** | Die Küche schließt **30 Minuten vor Restaurantschluss**; die Zeiten sind je Standort konfigurierbar. | **16** (M2/F4) | Kreuzberg: 23:00 / 22:30 Uhr; Spandau: 22:00 / 21:30 Uhr. |

**Hinweis zur Regelabgrenzung:** BR-03 fasst die von Marco gemeinsam beschriebene Standort-Verfügbarkeits- und Preislogik zu **einer** fachlichen Regel zusammen. BR-02 enthält die in Nachricht 17 geklärte Ausnahme anonymer Gäste. Damit bleiben es **genau fünf** Business Rules.

**Zusätzliche funktionale Anforderung, keine sechste Business Rule:** Doppelbuchungen für überlappende Tisch-Zeit-Kombinationen verhindern (**Nachrichten 2, 11, 18**; M1/F2, M1/F11, M2/F6). Die genaue Reservierungsdauer ist noch offen.

## 5. Widersprüche: genau 2 – mit Auflösung

### W-01 – Reservierungen ausschließlich telefonisch? (Nachrichten 1, 19, 20)

- **Frühe Aussage (Meeting #1):** Reservierungen kommen ausschließlich per Telefon herein.
- **Spätere Aussage (Meeting #2):** Zusätzlich können Gäste über Google bzw. Google Maps Reservierungen anfragen.
- **Auflösung:** Telefon ist der **bevorzugte**, aber nicht der einzige Kanal. Die App soll **Telefon**, **Google-Anfrage** und **Walk-in** als Herkunft bzw. Eintrittsweg unterscheidbar erfassen.
- **Offen:** Ob Google-Anfragen automatisch übernommen oder manuell eingetragen werden und wann eine Anfrage als verbindlich bestätigt gilt.

### W-02 – Zählt jede Bestellung als Besuch? (Nachrichten 7, 8)

- **Frühe Aussage (Meeting #1):** Marco beschrieb zunächst, dass jede einem bekannten Gast zugeordnete Bestellung den Besuchszähler erhöht.
- **Spätere Aussage nach gezielter Rückfrage (Meeting #1):** Mehrere Bestellungen desselben Gastes an einem Abend zählen **zusammen nur als ein Besuch**; Besuche aus beiden Standorten werden addiert.
- **Auflösung:** Für die Bella-Card wird **ein Besuch pro Gast und Abend** gezählt, nicht eine Gutschrift pro Bestellung. Auch manuell erfasste Besuche spontaner, identifizierter Gäste können berücksichtigt werden.
- **Einordnung:** Dies ist eine **inhaltliche Inkonsistenz der ursprünglichen Zählbeschreibung**, die durch Marcos Präzisierung aufgelöst wurde; sie ist weniger ausdrücklich als W-01.

## 6. Neue Anforderung aus Meeting #2: genau 1

### N-01 – Cateringverwaltung (Nachrichten 13, 14, 15)

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

## 10. Quellenregister – alle 20 Antworten im Überblick

| Nachricht | Meeting / Frage | Kernaussage von Marco |
|---|---|---|
| **1** | M1/F1 | Reservierungen angeblich nur telefonisch; getrennte Hefte; zentrale Übersicht gewünscht. |
| **2** | M1/F2 | Becker-Familie, 8 Personen; fehlende Tischzuweisung und Konflikt mit anderen Belegungen. |
| **3** | M1/F3 | Erfahrungsbasierte Tischvergabe; Standortunterschiede und Außenplatzwünsche. |
| **4** | M1/F4 | Papierbestellungen, Speisekartenprobleme, fehlende Übersicht bei Abwesenheit. |
| **5** | M1/F5 | Bestellzettel, manuelle Abrechnung, fehlende Auswertungen. |
| **6** | M1/F6 | Stammgasterkennung, Telefonnummer, Bella-Card und 15 % Rabatt. |
| **7** | M1/F7 | Schwelle von 10 Besuchen; lückenhafte Excel-Listen; zunächst unpräzise Zählung je Bestellung. |
| **8** | M1/F8 | Ein Besuch je Abend; Besuche aus beiden Standorten werden addiert. |
| **9** | M1/F9 | Standortbezogene Gerichte; Word-Speisekarten; zentrale Pflege gemeinsamer Gerichte. |
| **10** | M1/F10 | Word intern, PDF online; Standardpreise und Standortabweichungen. |
| **11** | M1/F11 | Mobiles Dashboard, Doppelbuchungen, spontane Gäste. |
| **12** | M1/F12 | Walk-ins; automatische Tischfreigabe nach 2 Stunden bei No-show. |
| **13** | M2/F1 | Zukunftspläne; **erstmals Catering für Firmenevents** als neue Hauptanforderung. |
| **14** | M2/F2 | TechCorp-Beispiel; Angebote, Datenfelder, Auftragsstatus, Anzahlung. |
| **15** | M2/F3 | Bar-/Kartenzahlung; Bestellstatus; Catering-Anzahlung und Restzahlung. |
| **16** | M2/F4 | Standortbezogene Schließzeiten; Küchenschluss 30 Minuten vorher. |
| **17** | M2/F5 | Stammgast-Suche über Name/Telefon; manuelle Walk-in-Erfassung; anonyme Besuche zählen nicht. |
| **18** | M2/F6 | Prioritäten: Reservierungen, dann Bestellungen/Speisekarten, dann Stammgäste; Schichtplanung nachrangig. |
| **19** | M2/F7 | Hinweis auf einen weiteren Reservierungskanal entgegen früherer Aussage. |
| **20** | M2/F8 | Google-Reservierungsanfragen; Telefon bevorzugt; Herkunft Telefon/Google/Walk-in unterscheiden. |

**Quellenbasis:** ausschließlich die im Interview übermittelten Antworten von Marco. Nicht ausdrücklich bestätigte Details sind im Dokument als Annahme, Vorschlag oder offene Frage gekennzeichnet.
