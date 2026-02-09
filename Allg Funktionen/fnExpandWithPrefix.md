>[!warning] Depricated 
> Wurde in [[Join Query v2]] ersetzt, da dort Join an sich überarbeitet wurde.

``` pq
(inputTable as table, columnName as text, prefix as text) as table =>
let
    // Ensure the column exists
    ColumnNames = Table.ColumnNames(inputTable),
    CheckColumn = List.Contains(ColumnNames, columnName),

    Result =
        if CheckColumn then
            let
                // Get the first non-null nested table
                NonNullRows = Table.SelectRows(inputTable, each Record.Field(_, columnName) <> null),
                NestedTable = if Table.RowCount(NonNullRows) > 0 then Record.Field(NonNullRows{0}, columnName) else #table({}, {}),

                // Get column names from nested table
                NestedColumns = Table.ColumnNames(NestedTable),

                // Apply prefix dynamically
                PrefixedColumns = List.Transform(NestedColumns, each prefix & _),

                // Expand nested table with prefixed names
                ExpandedTable = Table.ExpandTableColumn(inputTable, columnName, NestedColumns, PrefixedColumns)
            in
                ExpandedTable
        else
            error "Column '" & columnName & "' not found in the input table."
in
    Result
```

# Infos
**Erklärung:** Nimmt Spalte vom typ `table` und "expanded" diese (klappt diese auf). Jede dabei neu entstandene Spalte erhält den als Parameter gelieferten Präfix.
**Typische Verwendung:** Zum ausklappen nach Join.
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]
