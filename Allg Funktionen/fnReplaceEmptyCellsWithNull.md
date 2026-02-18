``` pq
let
  fx_BlanksToNull =
  (
      tableIn as table,
      optional optionalColumns as nullable list,
      optional treatWhitespaceOnly as nullable logical
  )
  as table =>
  let
      _treatWS = if treatWhitespaceOnly = null then true else treatWhitespaceOnly,

      _allCols = Table.ColumnNames(tableIn),
      _targetCols =
          if optionalColumns = null then _allCols else List.Intersect({ _allCols, optionalColumns }),

      // → Typ sicher aus echter Spalte ermitteln
      _typeByName = (col as text) as type =>
          try Value.Type(tableIn[col]) otherwise type any,

      _transforms =
          List.Transform(
              _targetCols,
              (c as text) =>
                  let
                      _colType = _typeByName(c),
                      _fn = (v as any) as any =>
                          if v = null then null
                          else if Value.Is(v, type text) then
                              let
                                  _txt = v as text,
                                  _isEmpty = if _treatWS then Text.Trim(_txt) = "" else _txt = ""
                              in
                                  if _isEmpty then null else v
                          else
                              v
                  in
                      { c, _fn, _colType }
          ),

      result =
          if List.IsEmpty(_transforms)
          then tableIn
          else Table.TransformColumns(tableIn, _transforms, null, MissingField.Ignore)
  in
      result
in
  fx_BlanksToNull
```

# Infos
**Erklärung:** Ersetzt ""  (oder optional auch "   ") Zellen druch null.
**Typische Verwendung:** In Partnerspezifischer Bereinigung
**Parameter:** 
- `tableIn` = vorheriger Schritt
- `optionalColumns` (optional) = Liste von Spaltennamen; wenn null -> alle Spalten
- `treatWhitespaceOnly` (optional) = 
	- true = auch "   " -> null, 
	- false = nur "" -> null; 
		- Default = true
**Hauptanwendung:** [[0 Iron-Schicht|Iron]]

