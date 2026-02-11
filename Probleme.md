# Schema drift
**Schema drift** ist, wenn die Tabellenheader einer Ausgabe eines Dataflows sich ändern.
Also Dataflow läuft normal durch -> Tabelle wird erzeugt mit bestimmten Headern. Die Header die zum Zeitpunkte des **Pubish**[^1] in der Vorschau sind bzw. durchlaufen, sind Teil des festen Schemas.
Ändern sich diese Header (bspw. durch Parametrisierung der Dataflows), nennt man dies Schema drift.

Schema drift ist an sich für unser Projekt nicht unbedingt schlecht. Eher im Gegenteil. Es ist erwünscht und der Code ist extra so gebaut, dass er diese "drifts" standhält.
Durch die Menschlichkeit der Partner, die **Mapping-Tabelle** und die stetige **Weiterentwicklung** des Projekts ändern sich unsere Tabellenheader und damit auch unser Schema regelmäßig.

**TLDR:** Schema drift = Änderung der Tabellenheader (Löschen, Neue Hinzufügen, Umbenennen)

## Warum ist das Schlecht?
Es ist auf dem Papier alles extra so angelegt und Programmiert dass es laufen sollte.
Auch ist es geduldet dass es zu Schema drift kommt.

***Problem ist:** Dataflows in Microsoft Fabric unterstützen keine Schemachanges on **Refresh[^2]***

### Lösung
Glaub mir, ich hab so viel versucht. Es ist nicht möglich das unter der Nutzung von Dataflows zu lösen. Wenn du es schaffst, sei glücklich.

**Edgar meint:** Wir sollen das Problem einfach ignorieren und normal mit Dataflows weiter arbeiten.

**Ich sage:** Verwende Python/Pyspark Notebooks. Bitte. Tuh dir das mit Dataflows nicht an. Damit funktioniert das Endziel einer Datafactory gescheit. Außerdem erleichterst du dich SEHR bei Erstellung und Verwendung von Funktionen, du hast Packages zur Verfügung, welche dinge nochmal mehr erleichtern, vor allem bei sowas wie das Umrechnen von Einheiten.

[^1]: **Reload** = Manuell Save & Run drücken im Dataflow
[^2]: **Refresh** = Dataflow wird über [[Pipelines|Orchestrierung]] ausgelöst