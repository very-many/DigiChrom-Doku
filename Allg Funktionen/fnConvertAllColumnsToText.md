``` pq
(ConvertToText as table) =>
let
    ColumnNames = Table.ColumnNames(ConvertToText),
    ColumnTypes = List.Transform(ColumnNames, each {_, type text}),
    Result = Table.TransformColumnTypes(ConvertToText, ColumnTypes)
in
    Result
```

# Infos
**Erklärung:** Wandelt alle Spalten in `type text` um.
**Typische Verwendung:** Vor Export der Tabelle. (Manchmal auch beim Import, wenn man die Tabellen typen benötigt)
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]/([[2 Silver-Schicht|Silver]])
