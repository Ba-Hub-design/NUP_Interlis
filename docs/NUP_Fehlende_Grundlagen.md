# NUP: benötigte Grundlagen für Implementierung und Abnahme

Stand: 9. Oktober 2026. Kein Konverter implementiert.
Grundlage: statische [FME-Analyse](Referenzanalyse_Mapping.md) und
[Node-/Schemanachweise](FME_Strukturnachweise.json).

## Priorität und Lieferumfang

**A = vor der ersten fachlichen NUP-Implementierung erforderlich.**
**B = vor der Validierung des vollständigen Arth-Exports erforderlich.**
**C = vor Freigabe des portablen Windows-Mitarbeiterpakets erforderlich.**
Ein allgemeines GUI-/Worker-Gerüst lässt sich ohne Fachdateien vorbereiten.
Das ist aber kein nachgewiesener NUP-Export. Modellprüfung gehört bereits zur
ersten Implementierung und wird nicht bis zur Endabnahme aufgeschoben.

| ID | Fehlende Datei / Grundlage | Priorität | Benötigter Inhalt und Zweck |
|---|---|---|---|
| A1 | ILI-Datei mit `MODEL SZ_Nutzungsplanung_kommunal_V2` | A | Tatsächlich verwendete amtliche Revision einschließlich VERSION, Sprachversion, Klassen, Domänen, Rollen, OID- und Geometrieregeln |
| A2 | Alle direkt und transitiv importierten ILI-Dateien | A | Vollständige lokale Importkette; Modelle müssen ohne Netzabruf kompilieren und validierbar sein |
| A3 | NUP-`Stammdaten.xtf` aus Thema A005c | A | Originale Katalogobjekte und IDs für Rechtsstatus, Verbindlichkeit, Typ_Kanton und Lieferinhalt; aktiver Exporteingang |
| A4 | GDB-Schema-Only-Ausgabe der acht unten genannten Eingänge | A | Tatsächliche Namen, Feldtypen/-längen, Nullzulässigkeit, Domänen/Subtypes, Geometrieschema, CRS, Beziehungen und Schlüssel |
| A5 | Kleine konsistente NUP-Test-FileGDB | A | Nichtleere, freigegebene Beispiele für alle sechs Geometrieklassen sowie benötigte Typ-/Metadatenzeilen; vollständige Referenzketten |
| A6 | Abgabevorgaben / kantonale Validator-Konfiguration, falls vorhanden | A | Benötigte INI/XTF/Erweiterungen, zusätzliche Constraints, Referenzdaten und akzeptierte Warnungen; alternativ ausdrückliche Bestätigung, dass nur das ILI-Modell gilt |
| A7 | Fachentscheid zu den unten aufgeführten FME-Sonderregeln | A | Bestätigtes Zielverhalten bei Geometrieaufbereitung, IDs, Filtern und Fehlern; FME-Verhalten nicht ungeprüft übernehmen |
| B1 | Vollständige, konsistente Arth-NUP-FileGDB bzw. genehmigte NUP-Auswahl | B | Tatsächliche Werte, eindeutige IDs/Lookup-Schlüssel, alle Geometrien und Beziehungen; Arth BFS 1362 |
| B2 | Vollständige FME-NUP-Ausgabe aus genau B1 | B | Vergleich mit demselben Eingang, Datenstand, Parametern und Workbench-Stand; FME-Laufprotokoll und Objektzahlen mitliefern |
| B3 | Kantonale `Arth.xtf` aus Thema A005c | B | Unabhängiger Vergleich; Stand und Modellversion abgleichen. Andere Datenstände sind zu erklären, nicht als Exportfehler vorauszusetzen |
| B4 | Vorhandene Prüfberichte / bekannte NUP-Abweichungen | B | Referenzfehler und fachlich akzeptierte Unterschiede von neuen Fehlern trennen; falls es keine gibt, dies bestätigen |
| C1 | Windows-Testplatz und freigegebene Beispieldaten | C | Zunächst Windows 11 x64 unter normalem Benutzerkonto, ohne benötigte FME-/ArcGIS-/Python-/Java-Installation |
| C2 | Nutzungs-/Weitergaberechte für Modelle, Katalog und Testdaten | C | Für lokal mitgelieferte Ressourcen klären; reale Büro-Testdaten nicht in Mitarbeiter-ZIP aufnehmen |

Für A5 genügt ein kleiner genehmigter Auszug oder eine synthetische GDB auf
dem echten Schema. Die Original-Arth-GDB ist dafür noch nicht zwingend nötig.
Eine CSV allein ersetzt A5 nicht: Sie prüft weder den FileGDB-Leser noch Kurven,
Feature-Datasets, Geometrieauflösung oder native Windows-Laufzeitabhängigkeiten.
Fehlen in Arth bestimmte Klassen, trotzdem deren Schema liefern und zulässige
Leerfälle anhand des Modells klären; erforderliche Technikfälle separat synthetisch
bereitstellen. Ein einfacher erster Einzelklassenversuch ersetzt nicht die
Prüfung aller sechs Klassen.

## ILI-Dateien: konkret, ohne erfundene Abhängigkeiten

Empfohlene Paketablage: `modelle/` mit der NUP-ILI und sämtlichen Importen,
ergänzt um Herkunft/Downloadadresse, Datum und verwendetem Modellstand.
Der tatsächliche Dateiname muss nicht dem Modellnamen entsprechen.
Entscheidend ist die MODEL-Deklaration im Dateiinhalt.

Die Workbench benutzt `%DATA` zur Modellauflösung. Daraus lassen sich weder
die genaue Modellrevision noch die Importnamen verlässlich ableiten. Deshalb
werden insbesondere CHBase-, Units- oder CoordSys-Versionen **nicht geraten**.
Die MOpublic-Modelle aus dem AV-Projekt ersetzen keine NUP-Modelldateien.

Falls verfügbar: die ILI-Dateien aus der bisher funktionierenden NUP-FME-
Umgebung plus amtlicher Herkunftsnachweis liefern. Lokale Sonderänderungen
kenntlich machen. Wir prüfen danach die Importkette, Modellkompilation,
Prüfsummen und Vereinbarkeit mit Katalog/Referenz-XTF.

## NUP-Stammdaten: wirklicher Eingang, kein bloßes Soll

Benötigt: `kataloge/NUP/Stammdaten.xtf`, entsprechend der vorhandenen Quelle
`http://data.geo.sz.ch/public/Themen/A005c/Stammdaten.xtf`.
Der erneute HTTPS-Abruf am 9. Oktober 2026 bleibt vom Proxy mit HTTP 403
blockiert. Sichere Dateibereitstellung ist daher eine geeignete Alternative;
TLS-Prüfungen werden nicht abgeschaltet.

Die Workbench liest:

| Readerklasse | Benötigte deklarierte Felder |
|---|---|
| `SZ_Nutzungsplanung_kommunal_V2.Stammdaten.Katalogeintrag` | `xtf_id`, `Code`, `Name`, `SortierNr`, `Bemerkung` |
| `SZ_Nutzungsplanung_kommunal_V2.Stammdaten.Typ_Kanton` | `xtf_id`, `Code`, `Name`, `Abkuerzung`, `SortierNr`, `Bemerkung` |

`Katalogeintrag` wird im FME-Graph auf Lieferinhalt, Rechtsstatus und
Verbindlichkeit verteilt. Die IDs werden in die Ausgabe und deren Rollen
übernommen. Die aktuelle Datei muss auf ihre tatsächliche Klassenstruktur
geprüft werden; diese Readerdeklaration ist kein Inhaltsnachweis des noch
nicht heruntergeladenen Katalogs.

## FileGDB: acht konkrete Eingänge

Die folgenden Feldtypen sind **FME-Deklarationen**, noch keine geprüften GDB-
Eigenschaften. „Liefern“ bedeutet Schema und, im Testauszug, repräsentative
Werte bereitstellen. Ob ein Attribut laut ILI zwingend befüllt sein muss,
entscheidet erst die Modellprüfung; ein `text(1000)` ist kein Pflichtfeldnachweis.

| Tabelle / Feature Class | Für fachliches Mapping zu liefern | Weitere im FME-Reader deklarierte Felder |
|---|---|---|
| `NUP/Grundnutzung_Zonenflaeche` | Geometrie; `xtf_id` text(255), `Code` text(10), `Rechtsstatus` text(255), `Bemerkung` text(1000) | `LES_Typ` text(255), `OBJECTID` |
| `NUP/Linienbezogene_Festlegung` | Geometrie; `xtf_id` text(255), `Code` text(10), `Rechtsstatus` text(255), `Bemerkung` text(1000) | `OBJECTID` |
| `NUP/Objektbezogene_Festlegung` | Geometrie; `xtf_id` text(255), `Code` text(10), `Rechtsstatus` text(255), `Bemerkung` text(1000) | `OBJECTID` |
| `NUP/Ueberlagernde_Festlegung` | Geometrie; `xtf_id` text(255), `Code` text(10), `Rechtsstatus` text(255), `Bemerkung` text(1000) | `OBJECTID` |
| `NUP/Wirkbereich_Linie` | Geometrie; `xtf_id` text(255), `Code` text(10), `Rechtsstatus` text(255), `Bemerkung` text(1000), `rLinienbezogene_Festlegung` text(200) | `OBJECTID`, `Shape_Length` double, `Shape_Area` double |
| `NUP/Wirkbereich_Punkt` | Geometrie; `xtf_id` text(255), `Code` text(10), `Rechtsstatus` text(255), `Bemerkung` text(1000), `rObjektbezogene_Festlegung` text(200) | `OBJECTID` |
| `NUP_Typ` | `xtf_id` text(200), `Code` text(10), `Code_Kanton` text(10), `Bezeichnung` text(80), `Abkuerzung` text(12), `Verbindlichkeit` text(255), `Nutzungsziffer` double, `Nutzungsziffer_Art` text(40), `Doklink` text(1023), `Bemerkung` text(1000), `Symbol` text(2048) | `GemeindeNr` int, `OBJECTID` |
| `NUP_TM_Datenbestand` | `xtf_id` text(200), `Lieferinhalt` text(200), `Stand` date, `Bemerkung` text(1000) | `OBJECTID` |

Die Geometrie ist eine native Feature-Geometrie; ein Quellfeld namens
`Geometrie` ist aus dem FME-Reader nicht belegt. Die tatsächlichen Feature-
Dataset-Pfade und Geometriespaltennamen gehören deshalb in die Schemaausgabe.
Die beiden Wirkbereiche sind im FME-Writer als `xtf_surface` deklariert,
Objektbezogene_Festlegung als `xtf_coord`; ILI und GDB müssen dies bestätigen.

`OBJECTID`, Shape_Length und Shape_Area sind technische Quellattribute,
keine bestätigten INTERLIS-Zielfelder. `LES_Typ` ist für eine spätere LES-
Implementierung relevant, aber keine automatisch verpflichtende NUP-Rolle.
Es wird sogar im NUP-Writer-Schema genannt; ob es im gültigen NUP-Modell
existiert bzw. vom Writer ignoriert wird, ist mit ILI zu klären.
`GemeindeNr` muss zur Prüfung des Gemeindeumfangs mitgeliefert werden:
Die Workbench deklariert das Feld, belegt aber keinen entsprechenden Filter.

### Schlüssel und Verknüpfungen im Testauszug

- Geometrie.`Code` → `NUP_Typ.Code` → dessen `xtf_id` als `rTyp`.
- `NUP_Typ.Code_Kanton` → Katalog-`Typ_Kanton.Code` → Katalog-ID als `rTyp_Kanton`.
- `NUP_Typ.Verbindlichkeit` → Katalog-Code → Katalog-ID als `rVerbindlichkeit`.
- Geometrie.`Rechtsstatus` → Rechtsstatus-Katalog-Code → Katalog-ID als `rRechtsstatus`.
- Metadaten.`Lieferinhalt` → FME-GetWord-Normalisierung → Katalog-`Name`
  → Katalog-ID als `rLieferinhalt`.
- Wirkbereich-Rollen → vorhandene `xtf_id` der zugehörigen Linien-/Punktfestlegung.

Die Beispiele müssen diese Ketten vollständig enthalten. Beim Anonymisieren
IDs über alle Tabellen konsistent ersetzen; Katalog-IDs nicht willkürlich
ändern. Namen und Freitext lassen sich bereinigen, soweit dadurch getestete
Lookup-Werte nicht verändert werden.

## Inhalt der Schema-Only-Ausgabe und kleinen Test-GDB

Schemaausgabe als JSON/CSV oder native Schema-XML: für jedes Objekt vollständiger
Pfad, Feldnamen/-typen/-längen, Nullzulässigkeit, Defaults, Domains/Subtypes,
Relationship Classes, Geometriespalte/-typ, Multipart, Kurvenfähigkeit, Z/M,
CRS einschließlich WKT/EPSG sowie XY-/Z-Auflösung und Toleranzen. Dazu
Objektzahlen und, ohne vertrauliche Werte, Angaben über vorhandene Leerklassen.
Keine neue ArcGIS-/FME-Installation ist auf Mitarbeiter-PCs erforderlich;
eine solche Schemaausgabe kann mit vorhandenen Büro-Werkzeugen erstellt werden.

Die Test-GDB sollte mindestens enthalten: eine zusammenhängende normale
Grundnutzungsfläche, Überlagerung, Linie, Punkt, beide flächenhaften Wirkbereiche,
benötigte Typobjekte und einen Datenbestand mit echtem Datum. Zusätzlich
repräsentative Kurven, Innenring, Multipart, kleine Fläche/Rundungsgrenze sowie
ein Overlay-/Dissolve-Beispiel mit nachvollziehbarer Attributwahl. Falls diese
Fälle im echten Schema unzulässig sind, nicht künstlich als gültige Beispiele
einfügen, sondern getrennte Negativfälle kennzeichnen.

Fehlerfälle können später synthetisch ergänzt werden: doppelte IDs, fehlende
Lookup-Treffer, verwaiste Rollen, ungültige Datumswerte und falsche Gemeinde.
Sie müssen nicht alle bereits vom Büro geliefert werden. Die Originaldaten
bleiben unverändert; Arbeitskopien und anonymisierte Auszüge werden getrennt
erstellt und vor Repository-Übernahme geprüft.

## Fachentscheidungen vor dem ersten NUP-Export

1. **Geometriepolitik:** FME linearisiert mit 0.0005 Koordinateneinheiten und
   rundet XYZ auf drei Dezimalstellen. Soll dies fachlich übernommen werden,
   oder verlangt/erlaubt die ILI einen anderen Kurven-/Präzisionsvertrag?
2. **Grundnutzungsaufbereitung:** Ist Overlay mit automatischer Cleaning-
   Toleranz, anschließendes Dissolve nach xtf_id und Attributübernahme aus einem
   Feature erforderlich? Benötigt werden belegte Vorher/Nachher-Fälle.
3. **IDs/Baskets:** Bestands-xtf_id erhalten; Kollisionen und OID-Domänen prüfen.
   FME-BIDs sind `ch.sz.a005c.geobasisdaten.1362`,
   `ch.sz.a005c.transfermetadaten.1362` und
   `ch.sz.a005.stammdaten.2000-01-01`. Katalog und Abgabevorgaben müssen diese
   Regeln bestätigen; das Datum ist nicht automatisch der aktuelle Datenstand.
4. **Gemeindeumfang:** Arth BFS 1362, unabhängig von Dateinamen und FME-Default
   1341. Ist die GDB ausschließlich Arth oder braucht sie bestätigte Filter?
5. **Normalisierung/Fehlerpolitik:** Rechtsstatus-Normalisierung nur im FME-
   Grundnutzungspfad; GetWord-Regel für Lieferinhalt; Datumsreparatur und
   Dublettenbehandlung fachlich festlegen. Keine stille Korrektur oder Löschung.

Diese Entscheidungen lassen sich teilweise aus A1–A5 beantworten. Es ist
keine separate Vorabfrage zu jeder Regel nötig; nur verbleibende Widersprüche
werden danach zur fachlichen Klärung vorgelegt.

## Was zunächst nicht benötigt wird

Für die erste NUP-Implementierung sind weder GWR-/LES-/SNP-ILI noch deren
Kataloge oder vollständige Fachreferenzen erforderlich. Ihre Architektur
wird durch getrennte Profile, tabellenspezifische Abhängigkeiten, Katalogadapter
und gemeinsame Job-/Prüfsteuerung vorbereitet. Ein NUP-Lauf prüft diese
anderen Fachbereiche nicht.

Die kantonale Arth-XTF und die vollständige FME-Ausgabe sind keine laufenden
Exporteingänge. Sie werden zum unabhängigen Fachvergleich gebraucht. Fehlt
eine gleichzeitige FME-Referenz, kann die Modellvalidierung dennoch durchgeführt
werden; die behauptete Übereinstimmung zum bisherigen FME-Verhalten bleibt
aber offen. FME-Ausgangsvalidierung ist bei NUP gespeichert deaktiviert, deshalb
muss auch die Referenz zunächst mit den bestätigten Modellen geprüft werden.

Aus dem AV-Aufbereiter bleiben portable Laufzeitpaketierung, GUI/Worker,
Diagnose, Hashmanifeste, reproduzierbarer Windows-Build und reale Relokations-
abnahme die technischen Vorbilder. INTERLIS-Modellierung, Katalogbeziehungen,
XTF-Erzeugung und Geometriebehandlung werden eigenständig für NUP umgesetzt.
AVs Esri-GDB-Writer-Prüfer und MOpublic-Konverter sind keine NUP-Pflichtabhängigkeiten.

## Empfohlenes nächstes Paket

```text
NUP_Grundlagen/
  modelle/                 NUP-ILI und vollständige Importkette
  kataloge/NUP/            Stammdaten.xtf
  schema/                  Schema-Only-Ausgabe der acht Eingänge
  testdaten/               kleine konsistente Test.gdb, optional als ZIP
  pruefvorgaben/           vorhandene kantonale Konfiguration / Hinweise
  herkunft.json            Versionen, Herkunft, Datenstand, BFS 1362
```

Dateinamen und Unterordner sind ein Liefer-Vorschlag, keine bereits vorhandenen
Artefakte. Zugangsdaten, interne Pfade und personenbezogene Inhalte gehören
nicht in Herkunftsdatei oder Berichte. Vollständige Arth-GDB, Referenz-XTFs
und Windows-Testdaten können anschließend getrennt nachgereicht werden.
