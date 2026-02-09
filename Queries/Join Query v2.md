```mermaid
flowchart TD

    A[GetExcel] --> B[SheetNames]

    A --> C[TransformTable per Sheet -> fnIPA]

    C --> D[renamedTbl]

    D --> E[ijh]

    E -- Ja --> F[ExperimentTables]

    E -- Nein, aber vorhanden --> G[SecondaryTables]

    F --> H[List.Accumulate Full Outer Join nach JoinCol]

    H --> I[Zwischenergebnis]

    G --> J[Für jedes Secondary: targetCol via Substring-Suche]

    J -->|gefunden| K[Full Outer Join (targetCol vs. JoinCol)]

    J -->|nicht gefunden| I

    K --> I

    I --> L[ReplaceErrorValues -> null]

    L --> M[fnConvertAllColumnsToText]

    M --> N[Finale Tabelle]
```

