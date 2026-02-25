``` pq
(source as table, ignoreList as list) as list =>
let
    Filtered =
        Table.SelectRows(
            source,
            each
                [Kind] = "Sheet" and
                not List.AnyTrue(
                    List.Transform(ignoreList, (w) => Text.Contains(Text.Trim([Name]), w, Comparer.OrdinalIgnoreCase))
                )
        ),
    Result = Table.Column(Filtered, "Name")
in
    Result
```

# Infos
**Erklärung:** Bekommt eine Excel Datei und eine Liste an Wörtern. Liefert eine Liste mit allen Namen der Tabellenblätter, außer welche, die die Wörter der Wörter-Liste erhalten.
Ist also dafür da um gezielt manche Tabellenblätter zu ignorieren, welche ignoriert werden sollen.
**Typische Verwendung:** Nach dem Import einer Excel, vor dem öffnen der einzelnen Tabellenblätter.
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]