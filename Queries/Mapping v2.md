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
Die **Query** nimmt die 
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

## `Rename`
```
let
    // 0) Eingaben – bei dir ersetzen/verbinden:
    Source = GetIron,
    MappingRecord = MappingRecord,

    // 1) Spaltennamen säubern (Trim)
    T0 = Table.TransformColumnNames(Source, Text.Trim),

    // 2) Namenlisten
    AllCols   = Table.ColumnNames(T0),
    MapKeys   = Record.FieldNames(MappingRecord),
    // Nur die Keys verarbeiten, deren Spalte es wirklich gibt (fehlende Keys werden still übersprungen)
    PresentKeys = List.Intersect({MapKeys, AllCols}),
    WrongInMapping = List.Difference(MapKeys, PresentKeys),
    // Unmapped = alle Quellspalten, die nicht durch Mapping (vorhandene Keys) betroffen sind
    Unmapped = List.Difference(AllCols, PresentKeys),

    // 3) Anwendung des Mappings:
    //    - Für jeden vorhandenen Key die Zielnamenliste säubern (Trim, RemoveNull/Empty, Distinct)
    //    - Bei >1 Ziel: Duplizieren für Ziele ab Index 1; Original in erstes Ziel umbenennen
    Renamed =
        List.Accumulate(
            PresentKeys,
            T0,
            (state as table, k as text) =>
                let
                    rawTargets       = Record.Field(MappingRecord, k),
                    targetsAsList    = if rawTargets is list then rawTargets else { Text.From(rawTargets) },
                    targetsText      = List.Transform(targetsAsList, each if _ is null then null else Text.From(_)),
                    targetsTrimmed   = List.Transform(targetsText, each if _ is null then null else Text.Trim(_)),
                    targetsNoNull    = List.RemoveNulls(targetsTrimmed),
                    targetsNoEmpty   = List.Select(targetsNoNull, each _ <> ""),
                    targetsDistinct  = List.Distinct(targetsNoEmpty),
                    /* rawTargets       = Record.Field(MappingRecord, k),
                    targetsAsList    = if rawTargets is list then rawTargets else { Text.From(rawTargets) },
                    targetsDistinct  = List.Distinct(targetsAsList), */

                    // Hinweis: wenn keine gültigen Ziele vorhanden sind, überspringen wir optional
                    // -> falls du stattdessen Fehler willst, ersetze den nächsten Zweig durch "error ..."
                    AfterMap =
                        if List.Count(targetsDistinct) = 0 then
                            state      // überspringen, da keine Ziele definiert
                        else
                            let
                                // Kopien für Ziele ab Index 1
                                AfterDups = List.Accumulate(
                                    List.Skip(targetsDistinct, 1),
                                    state,
                                    (s as table, targetName as text) =>
                                        Table.DuplicateColumn(s, k, targetName)
                                ),
                                // Original k -> erstes Ziel umbenennen
                                AfterRename = Table.RenameColumns(AfterDups, {{k, targetsDistinct{0}}}, MissingField.Ignore)
                            in
                                AfterRename
                in
                    AfterMap
        ),

    // 4) Nicht gemappte Spalten mit Präfix versehen (ohne Doppelpräfix)
    Prefix ="xx_" & Partner & "_",
    RenamePairsUnmapped =
        List.Transform(
            Unmapped,
            (c) => { c, if Text.StartsWith(c, Prefix) then c else Prefix & c }
        ),
    Prefixed = Table.RenameColumns(Renamed, RenamePairsUnmapped, MissingField.Ignore),

    // 5) Optional: Duplikat-Namen prüfen
    FinalCols = Table.ColumnNames(Prefixed),
    _dupCheck =
        let
            distinctCount = List.Count(List.Distinct(FinalCols)),
            totalCount    = List.Count(FinalCols)
        in
            if distinctCount = totalCount then null
            else error "Ergebnis enthält doppelte Spaltennamen nach dem Mapping (z. B. Zielkollisionen).",

    Result = fnSanitizeTable(Prefixed)
in
    Result
```