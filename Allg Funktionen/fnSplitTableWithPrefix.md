``` pq
(MyTable as table, Prefix as text, ColumnRange as list) =>
let
    // Spaltennamen holen
    ColNames = Table.ColumnNames(MyTable),

    // Start- und Endindex aus der Liste
    StartIndex = ColumnRange{0},
    EndIndex = ColumnRange{1},

    // Sicherstellen, dass die Indizes gültig sind
    ValidStart = if StartIndex < 0 then 0 else StartIndex,
    ValidEnd = if EndIndex > List.Count(ColNames) - 1 then List.Count(ColNames) - 1 else EndIndex,

    // Spaltennamen im angegebenen Bereich
    SelectedCols = List.Range(ColNames, ValidStart, ValidEnd - ValidStart + 1),

    // Für jede Zeile prüfen, ob irgendeine Spalte im Bereich mit Prefix beginnt
    MatchList = List.Transform(
        Table.ToRows(MyTable),
        each List.AnyTrue(
            List.Transform(
                List.FirstN(List.Skip(_, ValidStart), ValidEnd - ValidStart + 1),
                (val) => if val <> null then Text.StartsWith(Text.From(val), Prefix) else false
            )
        )
    ),

    // Index des ersten Treffers
    FirstMatchIndex = List.PositionOf(MatchList, true),

    // Tabellen splitten
    AboveTable = if FirstMatchIndex = -1 then #table({}, {}) else Table.FirstN(MyTable, FirstMatchIndex),
    BelowTable = if FirstMatchIndex = -1 then #table({}, {}) else Table.Skip(MyTable, FirstMatchIndex),

    Result = {AboveTable, BelowTable}
in
    Result
```

# Infos
**Erklärung:** Spaltet Input-Table in Above- und Below-Table
**Typische Verwendung:** Spalten von Tabelle in Header- und Data-Part. Am besten dafür den Präfix der Experiment ID verwenden.
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]

# Beispiel

**`Vorher = ...`**

| Column1 | Column2    | Column3 |
| ------- | ---------- | ------- |
| null    | Experiment | Hülle   |
| Test    | 1          | Gummi   |
| null    | IPA0001    | null    |
**`Nachher = fnSplitTableWithPrefix(Vorher, "IPA", {0, 4})`**
`Nachher{0}:`

| Column1 | Column2    | Column3 |
| ------- | ---------- | ------- |
| null    | Experiment | Hülle   |
| Test    | 1          | Gummi   |
`Nachher{1}:`

| *Column1* | *Column2* | *Column3* |
| ------- | ------- | ------- |
| *null*    | *IPA0001* | *null*    |
