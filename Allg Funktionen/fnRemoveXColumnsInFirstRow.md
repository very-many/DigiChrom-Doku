``` pq
(RemoveColsByFirstCellContains as table, ContainsText as text) as table =>
let
    FirstRow = Table.FirstN(RemoveColsByFirstCellContains, 1),
    FirstRec = if Table.RowCount(FirstRow) = 0 then [] else FirstRow{0},

    ColsToKeep =
        List.Select(
            Table.ColumnNames(RemoveColsByFirstCellContains),
            (c) =>
                let
                    v = try Record.Field(FirstRec, c) otherwise null,
                    s = if v = null then "" else Text.From(v)
                in
                    not Text.Contains(s, ContainsText, Comparer.OrdinalIgnoreCase)
        ),

    Result = Table.SelectColumns(RemoveColsByFirstCellContains, ColsToKeep, MissingField.Ignore)
in
    Result
```

# Infos
**Erklärung:** Löscht Spalten, welche in der ersten Zelle einen bestimmten Text (Parameter) enthalten.
**Typische Verwendung:** Manche Partner (z.B. HG) haben ganz viele "Müll"-Spalten, welche wir entfernen Müssen
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]

# Beispiel

**`Vorher = ...`**

| Name  | Datum      | Spalte_1 |
| ----- | ---------- | -------- |
| Jonas | 01.01.1970 | null     |
| Edgar | 02.01.1970 | null     |
**`Nachher = fnRemoveXColumnsInFirstRow(Sheet, "Spalte")`**

| Name  | Datum      |
| ----- | ---------- |
| Jonas | 01.01.1970 |
| Edgar | 02.01.1970 |

