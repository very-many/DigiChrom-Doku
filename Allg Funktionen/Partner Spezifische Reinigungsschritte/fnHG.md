``` pq
(source as table, sheetNames as list, idColumns as list) =>
let
    Sheet = fnSheet(source, sheetNames),
    Trim = fnTrimAllColumns(Sheet),

    Split = fnSplitTableWithPrefix(Trim, "DC", {0, 4}),
    HeaderTableFillDown = Table.FillDown(Split{0}, Table.ColumnNames(Split{0})),
    HeaderTableOnlyHeaders = Table.LastN(HeaderTableFillDown, 1),
    DataTable = Split{1},
    Append = Table.Combine({HeaderTableOnlyHeaders, DataTable}),

    RemoveNullHeader = fnRemoveNullColumnsInFirstRow(Append),
    HandleDupes = fnFixDuplicateHeaders(RemoveNullHeader),

    ColumnNames = Table.ColumnNames(HandleDupes),
    ExistingColumn = List.First(List.Select(idColumns, each List.Contains(ColumnNames, _)), null),
    RemoveTrash = 
        if ExistingColumn <> null
            then 
                let
                    RemovePlaceholder = Table.SelectRows(HandleDupes, each Record.Field(_, ExistingColumn) <> "DCRaw"),
                    RemoveNulls = Table.SelectRows(RemovePlaceholder, each Record.Field(_, ExistingColumn) <> null)
                in
                    RemoveNulls
        else #table({}, {})
in
    RemoveTrash
```

# Infos
**Erklärung:** Spaltet die Tabelle in *Header* und *Data* und führt verschiedene Schritte getrennt aus um *Header* zu finden gesondert/Besser zu behandeln.
**Typische Verwendung:** Reinigung jedes Tabellenblatts von HG
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]
