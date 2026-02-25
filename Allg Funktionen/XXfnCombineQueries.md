``` pq
(TablesWithNames as list) =>
let
    Records = List.Transform(
        TablesWithNames,
        each
            let
                TablePart = _[Table],
                Prefix = _[Name],
                RecordResult = if Table.RowCount(TablePart) > 0 then
                    let
                        RowValues = Table.ToRows(TablePart){0},
                        ColNames = Table.ColumnNames(TablePart),
                        PrefixedNames = List.Transform(ColNames, each if Prefix <> "" then Prefix & "_" & _ else _),
                        RecordWithPrefix = Record.FromList(RowValues, PrefixedNames)
                    in
                        RecordWithPrefix
                else
                    []
            in
                RecordResult
    ),

    NonEmptyRecords = List.RemoveNulls(List.Select(Records, each _ <> [])),
    CombinedRecord = if List.Count(NonEmptyRecords) > 0 then Record.Combine(NonEmptyRecords) else [],
    Result = if CombinedRecord <> [] then Table.Transpose(Record.ToTable(CombinedRecord)) else #table({}, {})
in
    Result
```

# Infos
**Erklärung:** Joint verschiedene Queries hintereinander *OHNE* ID Colums (be careful) 
**Typische Verwendung:** Joinen der Queries in BC-Charges, da diese nur Einzeiler sind und somit sowieso zusammengehören.
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]