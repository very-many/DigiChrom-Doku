``` pq
// fnRemovePrefixedColumns
(RemoveFromTables as list, prefix as text) as list =>
let
    TransformTables =
        List.Transform(
            RemoveFromTables,
            (t as table) =>
                let
                    colsToRemove =
                        List.Select(
                            Table.ColumnNames(t),
                            each Text.StartsWith(_, prefix)
                        )
                in
                    Table.RemoveColumns(t, colsToRemove)
        )
in
    TransformTables
```

# Infos
**Erklärung:** Entfernt Spalten mit dem Präfix `prefix`
**Typische Verwendung:** Um ungemappte Spalten in Gold rauszuwerfen
**Hauptanwendung:** [[3 Gold-Schicht|Gold]]

# Beispiel

**`Vorher = ...`**

| 01_Plating | xx_TUI_Versuch | 01_Plating_Bearbeiter |
| ---------- | -------------- | --------------------- |
| 1          | TUI001         | Hans                  |
| 2          | TUI002         | Bernhardt             |
**`Nachher = fnRemovePrefixedRows(Vorher, "xx_")`**

| 01_Plating | 01_Plating_Bearbeiter |
| ---------- | --------------------- |
| 1          | Hans                  |
| 2          | Bernhardt             |

