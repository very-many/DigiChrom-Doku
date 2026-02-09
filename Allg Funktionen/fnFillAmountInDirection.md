``` pq
(table as table, rowIndices as list, columnNames as list, amount as number, direction as text) as table =>
let
    colNames = Table.ColumnNames(table),
    rowCount = Table.RowCount(table),

    updatedRows = List.Transform(
        {0..rowCount-1},
        (i) =>
            let
                currentRow = Record.ToList(table{i}),
                newRow =
                    if List.Contains(rowIndices, i) then
                        if direction = "right" then
                            // Für jede Spalte nach rechts füllen
                            List.Accumulate(
                                columnNames,
                                currentRow,
                                (state, colName) =>
                                    let
                                        colIndex = List.PositionOf(colNames, colName),
                                        replaced = List.ReplaceRange(state, colIndex+1, amount, List.Repeat({state{colIndex}}, amount))
                                    in
                                        replaced
                            )
                        else if direction = "left" then
                            // Für jede Spalte nach links füllen
                            List.Accumulate(
                                columnNames,
                                currentRow,
                                (state, colName) =>
                                    let
                                        colIndex = List.PositionOf(colNames, colName),
                                        replaced = List.ReplaceRange(state, colIndex-amount, amount, List.Repeat({state{colIndex}}, amount))
                                    in
                                        replaced
                            )
                        else
                            currentRow
                    else if (direction = "bottom" or direction = "top") then
                        let
                            offset = if direction = "bottom" then -amount else amount,
                            sourceIndex = i + offset,
                            filledRow =
                                if sourceIndex >= 0 and sourceIndex < rowCount and List.Contains(rowIndices, sourceIndex) then
                                    List.Accumulate(
                                        columnNames,
                                        currentRow,
                                        (state, colName) =>
                                            let
                                                colIndex = List.PositionOf(colNames, colName),
                                                sourceRow = Record.ToList(table{sourceIndex}),
                                                replaced = List.ReplaceRange(state, colIndex, 1, {sourceRow{colIndex}})
                                            in
                                                replaced
                                    )
                                else
                                    currentRow
                        in
                            filledRow
                    else
                        currentRow
            in
                newRow
    ),

    result = Table.FromRows(updatedRows, colNames)
in
    result
```

# Infos
**Erklärung:** Kopiert Inhalt von Zellen (Ausgewählt über Zeilen und Spalten) um `x` Zellen in Richtung `y`. Dabei "füllt" die Funktion die übergangenen Felder auch.
**Parameter:** 
- `table` = vorheriger Schritt
- `rowIndices` = Liste aller Zeilen
- `columnNames` = Liste aller Spaltennamen
	- (Spaltennamen + Zeilennamen zum ermitteln der zu kopierenden Zellen)
- `amount` = Anzahl der zu kopierenden stellen
- `direction` = Richtung in welche kopiert werden soll
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]
# Beispiel

**`Vorher = ...`**

| Column1         | Column2 | Column3 |
| --------------- | ------- | ------- |
| null            | ABC     | DEF     |
| TextZumKopieren | null    | Trash   |
**`Nachher = fnFillAmountInDirection(Vorher, {1}, {"Column1"}, 2, "right")`**

| Column1         | Column2         | Column3         |
| --------------- | --------------- | --------------- |
| null            | ABC             | DEF             |
| TextZumKopieren | *TextZumKopieren* | *TextZumKopieren* |

