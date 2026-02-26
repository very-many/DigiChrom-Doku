# Code
## `CleanedMapping`
``` pq
let
  Source = GetMapping,

  Partner = "IPA",

  FindValue = List.Last(List.Select(Table.ColumnNames(Source), each Text.StartsWith(_, Partner & "_Spalte"))),
  Rename = Table.RenameColumns(Source, {
    {Partner & "_Variable value", "Partner_Unit_Mapping"},
    {FindValue, "Partner_Value"}
  })
in
  Rename
```

Diese Query nimmt das [[Mapping Dataflow|Bereinigte Mapping]] und benennt verschiedene Spalten um, um später einfacher damit arbeiten zu können.
## `UnitMapping`
``` pq
let
  Source = CleanedMapping,

  // Table Filter To Units
  ChooseColumns = Table.SelectColumns(Source, {"Ontologiename", "Prozess", "Typ", "Master_Unit", "Partner_Value", "Partner_Unit_Mapping"}), //Falls Edgar Header Namen ändert muss in CleanedMapping was geändert werden (oder im df fürs Mapping cleaning)
  ToText = fnConvertAllColumnsToText(ChooseColumns),
  FilterUnits = Table.SelectRows(ToText, each ([Typ] = "UnitSymbol")), 
  ExistsCol = Table.RenameColumns(Table.TransformColumns(FilterUnits, {"Partner_Value", each not (_ = null or _ = ""), type logical}), {"Partner_Value", "Is_Unit_Mapped"}),

  CombinedName = Table.AddColumn(ExistsCol, "Unit_Ontologiename", each Text.From([Prozess]) & "-" & [Ontologiename], type text),

  FilterNameAndMaster = Table.SelectRows(CombinedName, each [Master_Unit] <> null and [Unit_Ontologiename] <> null),
  FilterMappingAndIsMapped = Table.SelectRows(FilterNameAndMaster, each [Partner_Unit_Mapping] <> null or [Is_Unit_Mapped] <> false),

  PartnerUnitActual =
    Table.AddColumn(
        FilterMappingAndIsMapped,
        "Partner_Unit_Actual",
        each 
            if [Is_Unit_Mapped] = true then
                let
                    colName = [Unit_Ontologiename],
                    filteredTable =
                        if Table.HasColumns(GetTable, {colName}) then
                            Table.SelectRows(
                                GetTable,
                                (r) =>
                                    let v = Record.Field(r, colName) in
                                    v <> null and Text.Length(Text.From(v)) > 0
                            )
                        else
                            #table({}, {}),
                    value =
                        if Table.RowCount(filteredTable) > 0 then
                            Record.Field(filteredTable{0}, colName)
                        else
                            null
                in
                    value
            else
                null,
        type nullable any
    ),

  JoinString = Table.AddColumn(
    PartnerUnitActual,
    "Unit_Substring",
    each let p = Text.PositionOf([Ontologiename], "(", Occurrence.First)
          in if p >= 0 then Text.From([Prozess]) & "-" & Text.Range([Ontologiename], p) else null,
    type nullable text
  ),

  MergePartnerUnit = Table.AddColumn(JoinString, "Partner_Unit", each if [Partner_Unit_Actual] <> null then [Partner_Unit_Actual] else [Partner_Unit_Mapping], type any),
  SelectUsed = Table.SelectColumns(MergePartnerUnit, {"Unit_Ontologiename", "Master_Unit", "Partner_Unit", "Is_Unit_Mapped", "Unit_Substring"}),

  //"Fix" for those Multi Mappings
  TemporaryFix = Table.SelectRows(SelectUsed, each not Text.Contains([Unit_Ontologiename], "#"))
in
  TemporaryFix
```

## `ValueMapping`
``` pq
let
  Source = CleanedMapping,

  // Table Filter To Values
  ChooseColumns = Table.SelectColumns(Source, {"Ontologiename", "Prozess", "Typ", "Partner_Value"}),
  FilterValues = Table.SelectRows(ChooseColumns, each ([Typ] = "Value")),

  NoNullName =
    Table.SelectRows(
        FilterValues,
        (row as record) as logical =>
            let vals = Record.FieldValues(row) in not (List.Contains(vals, null) or List.Contains(vals, ""))
    ),

  CombinedName = Table.AddColumn(NoNullName, "Value_Ontologiename", each Text.From([Prozess]) & "-" & [Ontologiename], type text),

  JoinString = Table.AddColumn(
    CombinedName,
    "Value_Substring",
    each let p = Text.PositionOf([Ontologiename], "(", Occurrence.First)
          in if p >= 0 then Text.From([Prozess]) & "-" & Text.Range([Ontologiename], p) else null,
    type nullable text
  ),

  RemoveUnused = Table.RemoveColumns(JoinString, {"Ontologiename", "Prozess", "Typ", "Partner_Value"})
in
  RemoveUnused
```

## `Mapping`
``` pq
let
  Source = Table.NestedJoin(UnitMapping, {"Unit_Substring"}, ValueMapping, {"Value_Substring"}, "ValueMapping", JoinKind.Inner),
  Expand = Table.ExpandTableColumn(Source, "ValueMapping", {"Value_Ontologiename"}),

  RemoveUnused = Table.RemoveColumns(Expand, {"Unit_Substring"}),

  NormalizeUnits = Table.TransformColumns(RemoveUnused, {{"Master_Unit", each fnNormalizeUnit(_), type text},{"Partner_Unit", each fnNormalizeUnit(_), type text}})
in
  NormalizeUnits
```

# Ergebnis

| Unit_Ontologiename                                                       | Master_Unit | Partner_Unit | Is_Unit_Mapped | Value_Ontologiename                                                 |
| ------------------------------------------------------------------------ | ----------- | ------------ | -------------- | ------------------------------------------------------------------- |
| 01_PreparationSubstrate-UnitSymbol (SpecimenDiameter, process data set)  | mm          | mm           | TRUE           | 01_PreparationSubstrate-Value (SpecimenDiameter, process data set)  |
| 10_PlatingChromium-UnitSymbol (TemperatureActual, process data set)      | c           | c            | FALSE          | 10_PlatingChromium-Value (TemperatureActual, process data set)      |
| 01_PreparationSubstrate-UnitSymbol (SpecimenThickness, process data set) | mm          | mm           | TRUE           | 01_PreparationSubstrate-Value (SpecimenThickness, process data set) |
So oder so Ähnlich sieht die Tabelle aus der Query `Mapping` aus.
Die Tabelle hat 5 Spalten:
1. **Unit_Ontologiename:** Der Name der Spalte in der Bronze-Tabelle
2. **Master_Unit:** Die Master-Einheit zu dieser Spalte
3. **Partner_Unit:** Die Einheit, welche der Partner liefert
4. **Is_Unit_Mapped:** Existiert die Spalte bereits in der Tabelle des Partners? (`True/False`)
5. **Value_Ontologiename:** Die zugehörige Value-Spalte, wessen Wert umgerechnet werden muss

-> Hierauf kann aufgebaut werden um die Werte zu den Mastereinheiten umrurechnen.