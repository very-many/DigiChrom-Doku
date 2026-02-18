>[!warning] Depricated 
> Wurde im [[Mapping v2]] ersetzt, da dort das Mapping an sich überarbeitet wurde.

``` pq
(FixDuplicateRows as table, ColumnsToCheck as list) =>
let
    // Index hinzufügen
    IndexedTable = Table.AddIndexColumn(FixDuplicateRows, "RowIndex", 0, 1, Int64.Type),

    RenamedRows = Table.FromRecords(
        List.Transform(
            Table.ToRecords(IndexedTable),
            (row) =>
                let
                    NewRow = Record.TransformFields(
                        row,
                        List.Transform(
                            ColumnsToCheck,
                            (col) =>
                                let
                                    CurrentValue = Record.Field(row, col),
                                    PreviousValues = List.FirstN(Table.Column(IndexedTable, col), row[RowIndex]),
                                    Occurrences = List.Count(List.Select(PreviousValues, each _ = CurrentValue)),
                                    TotalOccurrences = List.Count(List.Select(Table.Column(IndexedTable, col), each _ = CurrentValue)),
                                    NewValue = if TotalOccurrences > 1 then CurrentValue & "_" & Text.From(Occurrences) else CurrentValue
                                in
                                    {col, each NewValue}
                        )
                    )
                in
                    NewRow
        )
    ),

    FinalTable = Table.RemoveColumns(RenamedRows, {"RowIndex"})
in
    FinalTable
```

# Infos
**Erklärung:** Fixt Doppelte Einträge gegebener Spalten
**Typische Verwendung:** Zum Promoten "unsicherer" Header (z.B. bei Headern von Partnern). Muss beim Renaming beachtet werden!
**Hauptanwendung:** [[0 Iron-Schicht|Iron]] (Mapping Table bereinigen)

# Beispiel

**`Vorher = ...`**

| Column1 | Column2 | Column3   |
| ------- | ------- | --------- |
| Entry   | Entry   | NewHeader |
| Entry   | ABC     | 134       |
**`Nachher = fnFixDuplicateEntries(Vorher, {"Column1", "Column2"})`**

| Column1 | Column2 | Column3   |
| ------- | ------- | --------- |
| *Entry_0* | Entry   | NewHeader |
| *Entry_1* | ABC     | 134       |

