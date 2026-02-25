``` pq
(AddMissingAndSort as table, ExpectedColumns as list) =>
let
    // Get current columns
    CurrentColumns = Table.ColumnNames(AddMissingAndSort),

    // Find missing columns
    MissingColumns = List.Difference(ExpectedColumns, CurrentColumns),

    // Add missing columns with null values
    TableWithAllColumns = List.Accumulate(
        MissingColumns,
        AddMissingAndSort,
        (state, col) => Table.AddColumn(state, col, each null)
    ),

    // Reorder columns to match ExpectedColumns list
    OrderedTable = Table.ReorderColumns(TableWithAllColumns, ExpectedColumns)
in
    OrderedTable
```

# Infos
**Erklärung:** Fügt fehlende Spalten hinzu (welche fürs Schema gebraucht werden)
**Typische Verwendung:** Exportieren ins Lakehouse (in Iron und eig nur verwendet in BC-Charges)
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]

# Beispiel

**`Vorher = ...`**

|     |     |
| --- | --- |
|     |     |
|     |     |
**`Nachher = `**

|     |     |     |
| --- | --- | --- |
|     |     |     |
|     |     |     |

