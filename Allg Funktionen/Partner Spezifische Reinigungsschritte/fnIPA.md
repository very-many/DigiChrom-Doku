>[!warning]
> **fnIPA** funktioniert (noch) nicht nach [[fnHSAA|Referenzschema]]. 

``` pq
(source as table, sheetNames as list, idColumns as list) =>
let
    Navigation = fnSheet(source, sheetNames),

    SafeTable = if (Type.Is(Value.Type(Navigation), type table)) then Navigation else #table({}, {}),

    Trim = fnTrimAllColumns(SafeTable),
    RemoveFirstRow = Table.Skip(Trim, 1),

    RemoveNullHeader = fnRemoveNullColumnsInFirstRow(RemoveFirstRow),
    HandleDupes = fnFixDuplicateHeaders(RemoveNullHeader),

    ColumnNames = Table.ColumnNames(HandleDupes),

    ExistingColumn = List.First(List.Select(idColumns, each List.Contains(ColumnNames, _)), null),

    FilterIDs =
        if ExistingColumn <> null then
            Table.SelectRows(HandleDupes, each Record.Field(_, ExistingColumn) <> null)
        else
            HandleDupes,

    ReplaceEmpty = fnReplaceEmptyCellsWithNull(FilterIDs, null, true)
in
    ReplaceEmpty
```

# Infos
**Erklärung:** Standard Reinigungsschritte + Löscht erste Zeile
**Typische Verwendung:** Reinigung jedes Tabellenblatts von IPA
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]
