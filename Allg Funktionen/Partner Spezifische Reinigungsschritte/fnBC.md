>[!info]
>Funktioniert nur bei Eingangs- und Ausgangsgrößen von BC. Charge Dateien sind gesondert zu behandeln.

``` pq
(source as table, sheetNames as list, idColumns as list) =>
let
    Sheet = fnSheet(source, sheetNames),
    Trim = fnTrimAllColumns(Sheet),
    ReplaceErrors = fnReplaceErrors(Trim),

    Split = fnSplitTableWithPrefix(ReplaceErrors, "BC", {1, 6}),
    HeaderTableOnlyHeaders = Table.LastN(Split{0}, 1),
    DataTable = Split{1},
    Append = Table.Combine({HeaderTableOnlyHeaders, DataTable}),
    
    
    RemoveNullHeader = fnKeepLeftBlob(Append),
    HandleDupes = fnFixDuplicateHeaders(RemoveNullHeader),

    ColumnNames = Table.ColumnNames(HandleDupes),
    ExistingColumn = List.First(List.Select(idColumns, each List.Contains(ColumnNames, _)), null),
    RemoveTrash = 
        if ExistingColumn <> null
            then 
                let
                    RemoveNulls = Table.SelectRows(HandleDupes, each Record.Field(_, ExistingColumn) <> null)
                in
                    RemoveNulls
        else #table({}, {}),

    ReplaceEmpty = fnReplaceEmptyCellsWithNull(RemoveTrash, null, true)
in
    ReplaceEmpty
```

# Infos
****Erklärung:** Spaltet die Tabelle in *Header* und *Data* und führt verschiedene Schritte getrennt aus um *Header* zu finden und gesondert/Besser zu behandeln.
**Typische Verwendung:** Reinigung jedes Tabellenblatts von IPA
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]


