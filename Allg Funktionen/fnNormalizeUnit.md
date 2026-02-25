> [!warning] Work In Progress
> Funktion ist noch WIP und sollte erweitert werden fürs umrechnen der Einheiten

``` pq
(Unit as nullable any) as nullable text =>
let
  Lower = Text.Lower(Unit),
  Hochzahlen = 
    let 
        a = Text.Replace(Lower, "²", "2"),
        b = Text.Replace(a, "³", "3"),
        c = Text.Replace(b, "^", "")
    in
        c,
  
  Mu = Text.Replace(Hochzahlen, "µ", "u"),
  Grad = Text.Replace(Mu, "°", "")
in
  Grad
```

# Infos
**Erklärung:** Normalisiert die Schreibweise von Einheiten
**Typische Verwendung:** Vor dem Umrechnen der Einheiten
**Hauptanwendung:** [[2 Silver-Schicht|Silver]]