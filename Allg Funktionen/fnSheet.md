``` pq
(ExcelTable as table, SheetCandidates as list) =>
let
    // Get all sheet names from the Excel workbook
    SheetNames = Table.Column(ExcelTable, "Item"),

    // Find the first sheet name from the candidate list that exists in the workbook
    SelectedSheet = List.First(
        List.Select(SheetCandidates, each List.Contains(SheetNames, _)),
        null
    ),

    // Load sheet if found
    RawTable = if SelectedSheet <> null
               then ExcelTable{[Item = SelectedSheet, Kind = "Sheet"]}[Data]
               else #table({}, {}),

    // Check if it's the placeholder case: one column "Column1" and one row "Keine Daten vorhanden"
    IsPlaceholder = Table.ColumnCount(RawTable) = 1
                    and Table.RowCount(RawTable) = 1
                    and Text.Lower(Text.Trim(RawTable{0}[Column1])) = "keine daten vorhanden",

    // Return empty table if placeholder detected
    Result = if IsPlaceholder then #table({}, {}) else RawTable
in
    Result
```

# Infos
**Erklärung:** Öffnet Tabellenblatt einer Excel Tabelle. Existiert die erste in der Liste der Kandidaten nicht, so wird die nächste genommen. Existiert keine -> liefert eine Leere Tabelle
**Typische Verwendung:** Bei jedem Öffnen eines Excel Tabellenblatts
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]