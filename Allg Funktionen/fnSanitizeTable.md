``` pq
(SourceTable as table, optional Reverse as logical) as table =>
let
    // Characters to replace and their replacements
    // Maybe add " " back
    OldChars = {"[", ",", ";", "{", "}", "(", ")", "=", "]"},
    NewChars = {"<", "'", ":", "\", "/", "`", "´", "+", ">"},

    // Decide which mapping to use based on Reverse flag
    Mapping = if Reverse = true then List.Zip({NewChars, OldChars}) else List.Zip({OldChars, NewChars}),

    // Custom function that replaces text using paired lists
    ReplaceForDelta = (InputText as text) as text =>
        List.Accumulate(
            Mapping,
            InputText,
            (state, pair) => Text.Replace(state, pair{0}, pair{1})
        ),

    // Get original column names and apply replacements
    OriginalColumnNames = Table.ColumnNames(SourceTable),
    RenamedColumnNames = List.Transform(OriginalColumnNames, each ReplaceForDelta(_)),
    RenamedTable = Table.RenameColumns(SourceTable, List.Zip({OriginalColumnNames, RenamedColumnNames}))
in
    RenamedTable
```

# Infos
**Erklärung:** Tauscht verbotene Zeichen im Header durch erlaubte aus oder andersherum.
**Typische Verwendung:** Bei jedem Exportieren von Tabellen. Oft beim einlesen von Tabellen ausm Lakehouse.
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]/[[2 Silver-Schicht|Silver]]/[[3 Gold-Schicht|Gold]]
