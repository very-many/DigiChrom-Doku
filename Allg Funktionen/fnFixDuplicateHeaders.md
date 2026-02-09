``` pq
(FixDuplicateHeaders as table) =>
let
    // Erste Zeile als Liste
    FirstRow = Record.ToList(Table.FirstN(FixDuplicateHeaders, 1){0}),
    OriginalColumns = Table.ColumnNames(FixDuplicateHeaders),
    // Neue Spaltennamen: eindeutige bleiben, Duplikate bekommen _0, _1, ...
    NewColumnNames = List.Accumulate(
        {0..List.Count(OriginalColumns)-1},
        {},
        (state, current) =>
            let
                CurrentValue = FirstRow{current},
                // Wie oft kommt der Wert bisher in der neuen Liste vor?
                Occurrences = List.Count(List.Select(state, each _ = CurrentValue or Text.StartsWith(_, CurrentValue & "_"))),
                NewName = if Occurrences = 0 and List.Count(List.Select(FirstRow, each _ = CurrentValue)) = 1
                          then CurrentValue
                          else CurrentValue & "_" & Text.From(Occurrences)
            in
                state & {NewName}
    ),
    // Tabelle mit neuen Spaltennamen
    Result = Table.RenameColumns(FixDuplicateHeaders, List.Zip({OriginalColumns, NewColumnNames})),
    RemoveTopRows = Table.Skip(Result, 1)
in
    RemoveTopRows
```

# Infos
**Erklärung:** Fixt Doppelte Einträge der ersten Zeile und promoted diese zu header
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]

# Beispiel

**`Vorher = ...`**

| Column1   | Column2   | Column3   |
| --------- | --------- | --------- |
| NewHeader | NewHeader | HeaderABC |
| Entry     | ABC       | 134       |
**`Nachher = fnFixDuplicateHeaders(Vorher)`**

| Column1     | Column2     | Column3   |
| ----------- | ----------- | --------- |
| *NewHeader_0* | *NewHeader_1* | HeaderABC |
| Entry       | ABC         | 134       |

