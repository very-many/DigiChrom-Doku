``` pq
(table as table, rowIndices as list) as table =>
let
    // Get column names
    colNames = Table.ColumnNames(table),

    // Ensure rowIndices are numbers and within range
    maxIndex = Table.RowCount(table) - 1,
    validIndices = List.Sort(List.Select(List.Transform(rowIndices, each Number.From(_)), each _ >= 0 and _ <= maxIndex)),

    // Extract rows to merge
    rowsToMerge = List.Transform(validIndices, (idx) => Record.ToList(table{idx})),

    // If no valid rows, return original table
    result =
        if List.Count(rowsToMerge) = 0 then
            table
        else
            let
                // Merge column-wise, converting everything to text
                mergedRowValues = List.Transform({0..List.Count(colNames)-1}, (colIndex) =>
                    let
                        columnValues = List.Transform(rowsToMerge, each Text.From(_{colIndex})),
                        nonNullValues = List.RemoveNulls(columnValues),
                        mergedText = Text.Combine(nonNullValues, "_")
                    in
                        mergedText
                ),

                // Add index column for filtering
                indexedTable = Table.AddIndexColumn(table, "Index", 0, 1, Int64.Type),

                // Remove original rows
                remainingRows = Table.SelectRows(indexedTable, each not List.Contains(validIndices, [Index])),

                // Drop index column
                remainingRowsNoIndex = Table.RemoveColumns(remainingRows, {"Index"}),

                // Create new row
                newRow = Record.FromList(mergedRowValues, colNames),

                // Insert new row at the position of the first selected index
                insertPosition = List.Min(validIndices),
                finalTable = Table.InsertRows(remainingRowsNoIndex, insertPosition, {newRow})
            in
                finalTable
in
    result
```

# Infos
**Erklärung:** Nimmt übergebene Zeilen und fasst diese in einer Zeile zusammen, indem die Inhalte mit "\_" getrennt hintereinander stehen
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]/[[1 Bronze-Schicht|Bronze]]/([[2 Silver-Schicht|Silver]])

# Beispiel

**`Vorher = ...`**

| Column1  | Column2  | Column3  |
| -------- | -------- | -------- |
| Metadata | Metadata | Metadata |
| File     | File     | null     |
| 1        | 2        | 3        |
**`Nachher = fnMergeRows(Vorher, {0,1,2})`**

| Column1         | Column2         | Column3    |
| --------------- | --------------- | ---------- |
| *Metadata_File_1* | *Metadata_File_2* | *Metadata_3* |


