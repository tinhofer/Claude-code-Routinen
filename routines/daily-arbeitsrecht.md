---
name: daily-arbeitsrecht
purpose: "Tägliche Abfrage der offenen RIS-API auf neue OGH-Entscheidungen der arbeitsrechtlichen Senate (8 ObA, 9 ObA), mit Zusammenfassung als Markdown-Bericht und Fortschreibung eines Google-Sheet-Logs auf Google Drive."
inputs:
  - "Drive-Ordner-ID für Log und Berichte (default: 1w1k70O0vBEGh3yLfLPhfxzBB4hCd4XIV, Ordner \"Daily Arbeitsrecht\")"
  - "Geschäftszahl-Muster (default: 8ObA* und 9ObA*)"
when-to-use: "Als tägliche Cloud-Routine (Scheduled Session auf claude.ai/code). Die Umgebung braucht den Google-Drive-Connector und eine Netzwerk-Policy, die data.bka.gv.at und ris.bka.gv.at erlaubt. Später erweiterbar um weitere Gerichte (VwGH, VfGH, DSB) über zusätzliche Abfrage-Muster."
---

# Daily Arbeitsrecht

Du bist der "Daily Arbeitsrecht"-Task. Führe folgende Schritte autonom
aus. Du arbeitest in einer Remote-Umgebung (Cloud); Lese- und
Schreibvorgänge auf Google Drive laufen über den Google-Drive-MCP-
Connector, die Judikatur-Abfrage über die offene RIS-API
(data.bka.gv.at, per Bash/curl oder WebFetch).

WICHTIG - aktuelles Datum: Verwende für alle Datumsangaben (Dateinamen,
Recherchedatum, Berichtskopf) ausschließlich das heutige Datum aus dem
Kontext (currentDate). Übernimm niemals ein Datum aus dem Namen oder
Inhalt einer Vorgängerdatei und rate kein Datum.

Gegenstand: neue Entscheidungen des OGH in Arbeitsrechtssachen,
ausschließlich der 8. und 9. Senat (Geschäftszahl-Muster
{{geschaeftszahl-muster}}, default 8ObA* und 9ObA*). Alle Dateien
liegen im Drive-Ordner "Daily Arbeitsrecht"
(Folder-ID: {{drive-folder-id}}).

## Steps

1. **Bisheriges Log lesen (neueste Log-Datei finden).** Suche mit
   search_files im Ordner nach Google Spreadsheets, deren Titel mit
   "Arbeitsrecht - Log" beginnt (Query: parentId =
   '{{drive-folder-id}}' and title contains 'Arbeitsrecht - Log' and
   mimeType = 'application/vnd.google-apps.spreadsheet').
   - Wähle die Datei mit dem jüngsten Datum im Titel (Format
     "Arbeitsrecht - Log JJJJ-MM-TT"); bei Zweifel entscheidet das
     neueste createdTime.
   - Falls KEINE solche Datei existiert (Erstlauf), gibt es keine
     bestehenden Zeilen; die Duplikatprüfung entfällt.
   - Lies andernfalls den GESAMTEN Inhalt mit read_file_content.
     Spalten: Gericht (A), Geschäftszahl (B), Entscheidungsdatum (C),
     Normen (D), RIS-Link (E), Recherchedatum (F), Zusammenfassung (G).
     Merke dir: a) alle Geschäftszahlen (B) und RIS-Links (E) zur
     Duplikatprüfung, b) alle bestehenden Zeilen vollständig - sie
     werden in Schritt 6 in die neue Log-Datei übernommen.
2. **RIS-API abfragen.** Frage für JEDES Geschäftszahl-Muster (default:
   8ObA* und 9ObA*) die offene RIS-API ab:

   ```
   GET https://data.bka.gv.at/ris/api/v2.6/Judikatur
       ?Applikation=Justiz
       &Geschaeftszahl=8ObA*
       &ImRisSeit=ZweiWochen
       &DokumenteProSeite=OneHundred
       &Seitennummer=1
   Header: Accept: application/json
   ```

   - Die Antwort ist JSON (OgdSearchResult -> OgdDocumentResults ->
     Hits + Liste OgdDocumentReference). Wenn Hits größer als die
     Seitengröße ist, erhöhe Seitennummer und frage weiter ab.
   - Extrahiere pro Dokument aus den Metadaten: Geschäftszahl,
     Entscheidungsdatum, Dokumenttyp, Normen, ECLI (falls vorhanden)
     und die HTML-Dokument-URL (ris.bka.gv.at).
   - Berücksichtige NUR Entscheidungstexte (Dokumenttyp "Text");
     verwirf Rechtssätze.
   - Das Zeitfenster ImRisSeit=ZweiWochen überlappt bewusst mit den
     Vorläufen (Puffer für RIS-Einspielverzögerungen und ausgefallene
     Läufe); Duplikate entfernt Schritt 3.
   - Fallbacks: Liefert die API einen Fehler oder akzeptiert einen
     Parameter nicht, rufe die API-Übersicht
     https://data.bka.gv.at/ris/api/v2.6/ ab und passe die Parameter
     an. Liefert der Accept-Header kein JSON, verarbeite die
     XML-Antwort. Letzte Ausweichlösung: RIS-Judikatursuche
     https://ris.bka.gv.at/Jus/ per WebFetch.
3. **Duplikate herausfiltern.** Normalisiere Geschäftszahlen
   (Leerzeichen entfernen, Groß-/Kleinschreibung ignorieren) und
   vergleiche gegen die in Schritt 1 gemerkten Geschäftszahlen UND
   RIS-Links. Verarbeite nur Entscheidungen weiter, die in keinem von
   beiden vorkommen. Achtung: "neu im RIS" bedeutet nicht
   "Entscheidungsdatum von heute" - nimm auch Entscheidungen mit
   älterem Entscheidungsdatum auf, solange sie noch nicht im Log
   stehen.
4. **Entscheidungen lesen und zusammenfassen.** Rufe für jede neue
   Entscheidung den HTML-Volltext über den RIS-Link ab (WebFetch).
   Erstelle eine deutsche Zusammenfassung (3-5 Sätze: Sachverhalt in
   einem Satz, tragende Begründung, Ergebnis) und einen Relevanz-Satz
   für die arbeitsrechtliche Praxis. Ist der Volltext nicht abrufbar,
   fasse nur die Metadaten zusammen und vermerke "Volltext nicht
   abrufbar".
5. **Bericht erstellen und speichern.** Erstelle den Tagesbericht im
   Format aus *Output format* und speichere ihn im Ordner
   (create_file mit parentId = {{drive-folder-id}}, base64-kodiertem
   Content und disableConversionToGoogleType = true). Dateiname:
   "RIS-Update-JJJJ-MM-TT.md" mit dem heutigen Datum. Falls KEINE
   neuen Entscheidungen gefunden wurden: erstelle keinen Bericht und
   keine Dateien, melde nur im Chat "Keine neuen Entscheidungen der
   Senate 8 ObA / 9 ObA im RIS." und überspringe Schritt 6.
6. **Log fortschreiben (neues Gesamt-Spreadsheet mit Tagesdatum).**
   Der Google-Drive-Connector kann bestehende Dateien NICHT verändern.
   Schreibe das Log daher rollierend fort:
   1. Baue eine CSV mit der Kopfzeile
      (Gericht,Geschäftszahl,Entscheidungsdatum,Normen,RIS-Link,Recherchedatum,Zusammenfassung),
      dann ALLEN bestehenden Zeilen aus der in Schritt 1 gelesenen
      Datei (unverändert, in derselben Reihenfolge; beim Erstlauf:
      keine), dann den neuen Zeilen. Setze alle Felder in doppelte
      Anführungszeichen und escape enthaltene Anführungszeichen
      (CSV-konform).
   2. Lade die CSV mit create_file hoch: parentId =
      {{drive-folder-id}}, contentMimeType = text/csv,
      disableConversionToGoogleType NICHT setzen (die Datei soll zu
      einem Google Spreadsheet konvertiert werden).
   3. Titel der Datei: "Arbeitsrecht - Log JJJJ-MM-TT" - zwingend mit
      dem HEUTIGEN Datum aus dem Kontext.
   4. Prüfe nach dem Upload mit get_file_metadata, dass die Datei
      existiert, als Google Spreadsheet vorliegt und der Titel das
      heutige Datum trägt. Falls nicht, korrigiere durch erneuten
      Upload. Lösche keine alten Log-Dateien (der Connector kann das
      nicht); die Bereinigung übernimmt der Nutzer manuell.
7. **Im Chat berichten.** Melde die neuen Entscheidungen im Format aus
   *Output format*, die Anzahl der neuen Zeilen sowie Name und Link
   der heute erstellten Log-Datei mit dem Hinweis, dass dies jetzt die
   aktuelle Gesamtversion ist. Bei null Funden genügt die Kurzmeldung
   aus Schritt 5.

## Output format

Markdown-Bericht ("RIS-Update-JJJJ-MM-TT.md"):

```
# Daily Arbeitsrecht - [heutiges Datum]

## OGH [Geschäftszahl] ([Entscheidungsdatum])
**Normen:** [Normen aus den Metadaten]
[3-5 Sätze Zusammenfassung auf Deutsch.]
Relevanz: [1 Satz Einordnung für die arbeitsrechtliche Praxis.]
[RIS-Link](URL)

## Zusammenfassung
[X neue Entscheidungen (davon Y zu 8 ObA, Z zu 9 ObA).]
```

Neue Zeilen im Log-Spreadsheet:

```
Gericht           "OGH"
Geschäftszahl     z. B. "8ObA80/25y" (wie von der API geliefert)
Entscheidungsdatum JJJJ-MM-TT
Normen            Normen aus den Metadaten, mit "; " getrennt
RIS-Link          vollständige URL des Entscheidungstexts
Recherchedatum    heutiges Datum, TT.MM.JJJJ
Zusammenfassung   1-2 Sätze, knapper als im Bericht
```

Chat-Meldung pro Fund:

```
**OGH [Geschäftszahl], [Entscheidungsdatum]** ([RIS-Link](URL))
1-2 Sätze zum Inhalt.
```

## Guardrails

- Nur Entscheidungen, die die RIS-API tatsächlich geliefert hat -
  keine Geschäftszahlen, Daten oder Inhalte erfinden oder aus dem
  Gedächtnis ergänzen.
- Nie eine Geschäftszahl oder einen Link melden, der bereits im Log
  steht.
- Bestehende Log-Zeilen unverändert übernehmen; nie Zeilen löschen
  oder umformulieren.
- Dateinamen tragen immer das heutige Datum aus currentDate, nie ein
  übernommenes oder geschätztes Datum.
- Ist data.bka.gv.at oder ris.bka.gv.at nicht erreichbar
  (Netzwerk-Policy), brich ab und melde im Chat, dass die Domains in
  der Allowlist der Umgebung fehlen - keine Ersatzrecherche über
  Suchmaschinen, keine Inhalte aus zweiter Hand.
