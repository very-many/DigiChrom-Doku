Okay zugegeben sind es 2 Queries. Aber könnte man auch ein eine stecken.
# Code
## Mapping nach Partner sortieren
```pq
let
  Source = GetMapping,

  NameColumn   = List.Last(List.Select(Table.ColumnNames(GetMapping), each Text.StartsWith(_, Partner & "_Name"))),
  TabColumn    = List.Last(List.Select(Table.ColumnNames(GetMapping), each Text.StartsWith(_, Partner & "_Tab"))),
  ColColumn = List.Last(List.Select(Table.ColumnNames(GetMapping), each Text.StartsWith(_, Partner & "_Spalte"))),
  SourceColumn = "Full_Name",



  FilterRows = Table.SelectRows(
    Source,
    each let nm = Record.Field(_, NameColumn) in nm <> null and nm <> ""
  ),
  
  CombinedName = Table.AddColumn(
    FilterRows,
    "NameCombined",
        (r) =>
            let
                nm = Record.Field(r, NameColumn),
                tb = Record.Field(r, TabColumn)
            in
                if tb = null or tb = "" then nm else tb & "_" & nm,
        type nullable text
  ),

  FinalMapping = Table.SelectColumns(CombinedName, {"NameCombined", SourceColumn}),
  Rename = Table.RenameColumns(FinalMapping, {{SourceColumn, "Rename"}})
in
  Rename
```
## Als `Record` Gruppieren
```pq
let
  mapping = Mapping,
  SourceFieldName = "NameCombined",
  TargetFieldName = "Rename",



  // Gruppieren und Rename-Liste aufbereiten
  Grouped = Table.Group(
      mapping,
      {"NameCombined"},
      {
          {
              "RenameList",
              (t as table) =>
                  let
                      raw      = Table.Column(t, "Rename"),
                      asText   = List.Transform(raw, each if _ is null then null else Text.From(_)),
                      trimmed  = List.Transform(asText, each if _ = null then null else Text.Trim(_)),
                      noNull   = List.RemoveNulls(trimmed),
                      noEmpty  = List.Select(noNull, each _ <> ""),
                      distinct = List.Distinct(noEmpty)
                  in
                      distinct,
              type {text}
          }
      }
  ),

  // Für Record-Konvertierung vorbereiten
  RenameCols = Table.RenameColumns(Grouped, {{"NameCombined", "Name"}, {"RenameList","Value"}}),

  // Tabelle -> Record
  ResultRecord = Record.FromTable(RenameCols)
in
  ResultRecord
```