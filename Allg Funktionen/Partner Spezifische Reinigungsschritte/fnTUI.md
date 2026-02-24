>[!warning]
> **fnTUI** funktioniert (noch) nicht nach [[fnHSAA|Referenzschema]]. 

``` pq
(source as table, sheetNames as list, idColumns as list) =>
let
    Sheet = fnSheet(source, sheetNames),
    Trim = fnTrimAllColumns(Sheet),
    ReplaceErrors = fnReplaceErrors(Trim),

    RemoveNullHeader = fnRemoveNullColumnsInFirstRow(ReplaceErrors),
    HandleDupes = fnFixDuplicateHeaders(RemoveNullHeader),

    ColumnNames = Table.ColumnNames(HandleDupes),
    ExistingColumn = List.First(List.Select(idColumns, each List.Contains(ColumnNames, _)), null),
    RemoveTrash = 
        if ExistingColumn <> null
            then 
                let
                    RemovePlaceholder = Table.SelectRows(HandleDupes, each Record.Field(_, ExistingColumn) <> "DCRaw"),
                    RemoveNulls = Table.SelectRows(RemovePlaceholder, each Record.Field(_, ExistingColumn) <> null)
                in
                    RemoveNulls
        else #table({}, {})

in
    RemoveTrash
```

# Infos
**Erklärung:** So ziemlich Standard Reinigungsschritte.
**Typische Verwendung:** Reinigung jedes Tabellenblatts der TUI
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]
