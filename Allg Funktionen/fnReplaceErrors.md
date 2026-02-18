``` pq
(InputTable as table) =>
let
  Source = Table.ReplaceErrorValues(
    InputTable,
    List.Transform(Table.ColumnNames(InputTable), each {_, null})
  )
in
  Source
```

# Infos
**Erklärung:** Ersetzt Error Zellen durch null Einträge.
**Typische Verwendung:** Vor Export von unsicheren Tabellen
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[2 Silver-Schicht|Silver]]