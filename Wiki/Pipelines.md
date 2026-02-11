**Pipelines** sind unser main Tool zur Orchestrierung der Dataflows. D.h. Pipelines sind dafür Zuständig die richtigen [[Dataflows]] in der richtigen Reihenfolge zu *reloaden*[^1], den [[Dataflows]] die richtigen Parameter zu geben, logging, ...

# Trigger Pipeline
![[Pipeline Example.excalidraw]]



[^1]: **Wichtig:** Dataflows können *reloaded* werden und *gepublished* werden. Beides ist unterchiedlich!! [[Probleme#Schema drift|Mehr dazu]]