``` pq
(Table as table, GroupColumn as text, PrefixColumn as text) as table =>
let
    // 1. Gruppieren nach der ersten Spalte (z. B. "Schritt")
    Grouped = Table.Group(Table, {GroupColumn}, {{"AllData", each _, type table}}),

    // 2. Für jede Gruppe: dynamisch Spalten umbenennen und zu einem Record zusammenführen
    Expanded = Table.TransformColumns(Grouped, {
        "AllData", each
            let
                // Hole alle Spalten außer GroupColumn und PrefixColumn
                OtherCols = List.RemoveItems(Table.ColumnNames(_), {GroupColumn, PrefixColumn}),
                // Wandle die Tabelle in eine Liste von Records um
                RowsAsList = Table.ToRecords(_),
                // Erzeuge eine Liste von Records mit umbenannten Keys
                RenamedRecords = List.Transform(RowsAsList, (row) =>
                    let
                        prefix = Text.From(Record.Field(row, PrefixColumn)),
                        renamed = Record.FromList(
                            List.Transform(OtherCols, (col) => Record.Field(row, col)),
                            List.Transform(OtherCols, (col) => prefix & "_" & col)
                        )
                    in
                        renamed
                ),
                // Kombiniere alle Records in einen einzigen Record
                CombinedRecord = Record.Combine(RenamedRecords)
            in
                CombinedRecord,
        type record
    }),

    // 3. Expandieren der Records zu Spalten
    Result = Table.ExpandRecordColumn(Expanded, "AllData", Record.FieldNames(Expanded{0}[AllData]))
in
    Result
```

# Infos
**Erklärung:** Gruppiert untereinander geschriebene Zeilen in hinereinander geschriebene Spalten und gibt ihnen einen Präfix.
**Typische Verwendung:** Mehrfach ausgeführte Schritte der BC-Charges in eine Zeile schreiben
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]

# Beispiel

**`Vorher = ...`**

|     |     |     |
| --- | --- | --- |
|     |     |     |
|     |     |     |
**`Nachher = `**

|     |     |     |
| --- | --- | --- |
|     |     |     |
|     |     |     |

