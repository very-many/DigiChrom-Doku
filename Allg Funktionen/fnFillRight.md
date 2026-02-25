``` pq
(tbl as table, rowNumber as number) as table =>
  let
      colNames = Table.ColumnNames(tbl),
      rows     = Table.ToRows(tbl),
      rowCount = List.Count(rows),

      out =
          if rowNumber < 0 or rowNumber >= rowCount then
              tbl
          else
              let
                  targetRow = rows{rowNumber},

                  // Nach rechts auffüllen (Left-Carry)
                  state0 = [last = null, out = {}],
                  stateF =
                      List.Accumulate(
                          targetRow,
                          state0,
                          (s, cur) =>
                              let
                                  newLast = if cur = null then s[last] else cur,
                                  newOut  = s[out] & { if cur = null then s[last] else cur }
                              in
                                  [last = newLast, out = newOut]
                      ),
                  filledRow = stateF[out],

                  // Nur die eine Zeile ersetzen
                  newRows = List.ReplaceRange(rows, rowNumber, 1, { filledRow }),

                  // Tabelle aus Zeilen + Spaltennamen wiederherstellen (ohne Typ-Zwang)
                  outTbl  = Table.FromRows(newRows, colNames)
              in
                  outTbl
  in
      out
```

# Infos
**Erklärung:** FillDown aber nach rechts in auswählbarer Zeile
**Typische Verwendung:** Reinigung der Mapping Tabelle
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]

# Beispiel

**`Vorher = ...`**

| Column1 | Column2 | Column3 |
| ------- | ------- | ------- |
| TUI     | null    | BC      |
| Name    | Spalte  | Name    |
**`Nachher = fnFillRight(Vorher, 0)`**

| Column1 | Column2 | Column3 |
| ------- | ------- | ------- |
| TUI     | *TUI*   | BC      |
| Name    | Spalte  | Name    |

