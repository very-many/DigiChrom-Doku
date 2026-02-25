# Code
```pq
let
    Source = GetExcel,
    // Get the names of all Sheets
    SheetNames = fnFilterSheets(Source, {"ignore", "einheit", "ignorieren", "tabelle", "Vgl", "Data1"}),
    idColumns = {"Experiment"},

    //Function: Transform table
    TransformTable = (sheetName as text) =>
        let
            // Spezifischer "Reinigungsschritt" für alle Blätter
            tbl = fnHSAA(Source, {sheetName}, idColumns),
            colNames = Table.ColumnNames(tbl),
            joinColOriginal = 
                List.First(
                    List.Select(idColumns, each List.Contains(colNames, _)),
                    null),
            TrimmedSheetName = Text.Trim(Text.Trim(Text.Trim(Text.From(sheetName), " "), "."), "-"),
            renamedTbl = Table.TransformColumnNames(tbl, each TrimmedSheetName & "_" & _),
            joinColRenamed = if joinColOriginal <> null then TrimmedSheetName & "_" & joinColOriginal else null,
            result = [Table = renamedTbl, JoinCol = joinColRenamed, OriginalJoin = joinColOriginal]
        in
            result,

    Processed = List.Transform(SheetNames, each TransformTable(_)),
    ExperimentTables = List.Select(Processed, each [OriginalJoin] = idColumns{0}),
    SecondaryTables = List.Select(Processed, each [OriginalJoin] <> null and [OriginalJoin] <> idColumns{0}),

    // Schritt 1: Experiment-ID-Tabellen joinen
    JoinedExperiment = List.Accumulate(
        ExperimentTables,
        [Table = #table({}, {}), JoinCol = null],
        (state, current) =>
            if state[JoinCol] = null then
                current
            else
                let
                    Combined = Table.Join(state[Table], state[JoinCol], current[Table], current[JoinCol], JoinKind.LeftOuter)
                in
                    [Table = Combined, JoinCol = state[JoinCol]]
    ),

    // Schritt 2: Sekundäre Tabellen joinen
    FinalJoinRecord = List.Accumulate(
        SecondaryTables,
        JoinedExperiment,
        (state, current) =>
            let
                existingCols = Table.ColumnNames(state[Table]),
                targetCol = if current[OriginalJoin] <> null then
                                List.First(List.Select(existingCols, each Text.Contains(_, current[OriginalJoin])), null)
                            else
                                null,
                Combined = if targetCol <> null then
                                Table.Join(state[Table], targetCol, current[Table], current[JoinCol], JoinKind.LeftOuter)
                           else
                                state[Table]
            in
                [Table = Combined, JoinCol = state[JoinCol]]
    ),

    FinalJoin = if FinalJoinRecord[Table] <> null then FinalJoinRecord[Table] else #table({}, {}),


    ReplaceErrors = fnReplaceErrors(FinalJoin),
    Text = fnConvertAllColumnsToText(ReplaceErrors),
    Sanitize = fnSanitizeTable(Text)
in
    Sanitize
```

# Erklärung
Die **Join Query v2**:
1. Lädt jedes Tabellenblatt (welches nicht auf der liste in [[fnFilterSheets]])
2. Reinigt jedes geladene Tabellenblatt jeweils mit dem [[Partner-Spezifischen Reinigungsschritte|Partner-Spezifischen Reinigungsschritt]]
3. Schreibt den Namen jedes Tabellenblatts vor die Werte des jeweiligen Tabellenblatts. Speichert sich dabei, welches der Werte ID/Schlüsselwerte sind *(Die Namen der Spalten, welche ID/Schlüsselwerte halten, müssen in der `idColumns` Liste angegeben werden)*
4. Joint alle geladenen und bereinigten Tabellenblätter, welche aus der Liste `idColumns` den ersten Wert (primärer Schlüssel/ID) als Schlüssel enthält
5. Joint auf die bereits aus dem vorherigen Schritt gejointe Primärschlüsseltabelle alle Tabellen, welche nicht Primärschlüssen, sondern nur Sekundärschlüssel enthalten
6. Ersetzt alle Zellen mit Errors durch null ([[fnReplaceErrors]])
7. Konvertiert alle Spaltentypen zu Text ([[fnConvertAllColumnsToText]])
8. Tauscht verbotene Zeichen im Header durch erlaubte aus ([[fnSanitizeTable]])

## Usage
Query kann genau so für jede Datei einer Standard [[0 Iron-Schicht|Iron Schicht]] verwendet werden.
Angepasst werden muss:
- **SheetNames bzw. [[fnFilterSheets]]:** ggf. mehr oder weniger Wörter in den Namen der Tabellenblätter filtern
- **idColumns:** an den Partner anpassen, dabei darauf achten, dass die Primäre ID bzw. der Primärschlüssel (meistens die Experiment-ID) an erster stelle stehen muss, jede weitere ID, mit welcher im nachhinein gejoined werden muss, muss dahinter geschrieben werden.

## Mermaid Diagramm
Die Query ist ein wenig komplex, darum hier eine Darstellung als Mermaid Diagramm:
```mermaid
flowchart TD

    A[Source aus GetExcel] --> B[SheetNames ermitteln]
    B --> C[Liste idColumns]

    %% TransformTable
    subgraph S1[TransformTable pro Sheet]
        direction TB
        T1[fnIPA auf Sheet anwenden]
        T2[colNames bestimmen]
        T3[joinColOriginal wählen]
        T4[SheetName trimmen]
        T5[Spaltenpräfix anwenden]
        T6[joinColRenamed berechnen]
        T7[Record zurückgeben]
        T1 --> T2 --> T3 --> T4 --> T5 --> T6 --> T7
    end

    B --> D[Processed = TransformTable je Sheet]

    D --> E[ExperimentTables filtern]
    D --> F[SecondaryTables filtern]

    %% Join Experiment Tables
    subgraph S2[Schritt 1: Experiment-ID Tabellen joinen]
        direction TB
        G1[Initial: leere Tabelle]
        G2[List.Accumulate über ExperimentTables]
        G3[Erstes Element übernimmt State]
        G4[Vollständiger Outer Join]
        G1 --> G2 --> G3 --> G4
    end

    E --> G1

    %% Join Secondary Tables
    subgraph S3[Schritt 2: Sekundär-Tabellen joinen]
        direction TB
        I1[State = JoinedExperiment]
        I2[List.Accumulate über SecondaryTables]
        I3[targetCol über Text.Contains finden]
        I4[Wenn gefunden: FullOuter Join]
        I5[Sonst State beibehalten]
        I1 --> I2 --> I3 --> I4 --> I5
    end

    G4 --> I1

    I5 --> J[FinalJoin erstellen]

    J --> K[ReplaceErrorValues anwenden]
    K --> L[Alle Spalten zu Text konvertieren]
    L --> M[Finales Ergebnis]
```

