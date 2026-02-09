``` pq
(RemoveNullColumns as table) =>
let
    FirstRow = Table.FirstN(RemoveNullColumns, 1),
    ColumnsToKeep = List.Select(Table.ColumnNames(RemoveNullColumns), each (Record.Field(FirstRow{0}, _) <> null and Record.Field(FirstRow{0}, _) <> "")),
    Result = Table.SelectColumns(RemoveNullColumns, ColumnsToKeep)
in
    Result
```

# Infos
**Erklärung:** Löscht alle Spalten, welche in der ersten Zeile "null" stehen haben
**Typische Verwendung:** Vorm header promote, je nachdem
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]

# Beispiel

**`Vorher = ...`**

| Column1 | Column2 | Column3 |
| ------- | ------- | ------- |
| abc     | null    |         |
| 524f    | 34      | null    |
**`Nachher = fnRemoveNullColumnsInFirstRow(Vorher)`**

| Column1 |
| ------- |
| abc     |
| 524f    |
