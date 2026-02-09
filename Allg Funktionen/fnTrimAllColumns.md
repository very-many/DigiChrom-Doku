``` pq
(TrimAllColumns as table) =>
let
    ColumnNames = Table.ColumnNames(TrimAllColumns),
    Transformations = List.Transform(
        ColumnNames,
        each {
            _,
            (value) =>
                if value <> null then
                    Text.Trim(Text.Trim(Text.Trim(Text.From(value), " "), "."), "-")
                else
                    null,
            type text
        }
    ),
    Result = Table.TransformColumns(TrimAllColumns, Transformations)
in
    Result
```

# Infos
**Erklärung:** Trimmt Header von ungewollten Zeichen (Vorne und hinten Löschen)
**Typische Verwendung:** Vor Export von (oder zugriff auf) Tabellen mit unsicherem Header
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]/([[2 Silver-Schicht|Silver]])