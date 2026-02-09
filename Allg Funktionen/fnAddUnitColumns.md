>[!warning] Depricated
> Wurde durch neue Unit-"Mapping" Methode ausgetauscht, weil kaum ein Partner konsistent die Einheiten angibt.

``` pq
(fnAddUnitColumns as table, optional prefix as text) as table =>
let
    // Standard-Prefix, falls nicht angegeben
    Prefix = if prefix <> null then prefix else "Einheit_",

    // Alle Spaltennamen der Tabelle
    ColumnNames = Table.ColumnNames(fnAddUnitColumns),

    // Spalten mit Einheiten erkennen (Muster: [ ... ])
    UnitColumns = List.Select(ColumnNames, each Text.Contains(_, "[")),

    // Für jede erkannte Einheit eine neue Spalte hinzufügen
    AddUnitColumns = List.Accumulate(
        UnitColumns,
        fnAddUnitColumns,
        (state, currentColumn) =>
            let
                // Einheit aus dem Spaltennamen extrahieren
                UnitText = Text.BetweenDelimiters(currentColumn, "[", "]"),
                // Neuer Spaltenname
                NewColumnName = Prefix & currentColumn,
                // Spalte hinzufügen, alle Werte = Einheit
                AddColumn = Table.AddColumn(state, NewColumnName, each UnitText, type text)
            in
                AddColumn
    )
in
    AddUnitColumns
```

# Infos
**Erklärung:** Extrahiert Einheiten aus Header, indem es ausließt was zwischen "\[...]" steht.  
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]/([[2 Silver-Schicht|Silver]])

# Beispiel

**`Vorher = ...`**

| Zugabe Eisen \[ml] | Column 2 |
| ------------------ | -------- |
| 34                 | null     |
| 54                 | null     |
**`Nachher = fnAddUnitColumns(Vorher)`**

| Zugabe Eisen \[ml] | Column 2 | *Einheit_Zugabe Eisen \[ml]* |
| ------------------ | -------- | -------------------------- |
| 34                 | null     | *ml*                         |
| 54                 | null     | *ml*                         |

