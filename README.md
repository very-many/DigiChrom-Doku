# DigiChrom-Doku

Dokumentation für das **DigiChrom**-Projekt – eine Datenpipeline auf Basis von **Microsoft Fabric**, die Daten verschiedener Partner verarbeitet und transformiert. Dieses Repository ist ein [Obsidian](https://obsidian.md)-Vault und enthält die gesamte projektbezogene Dokumentation.

## Inhalt

| Ordner / Datei       | Beschreibung                                                                                         |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| **Wiki/**            | Zentrale Architekturdokumentation (Medallion-Modell, Dataflows, Pipelines, Schichten, Versionierung) |
| **Allg Funktionen/** | Wiederverwendbare Funktionen (`fn*`) für Datenbereinigung und -transformation                        |
| **Aufbau/**          | Partnerspezifische Aufbau-Dokumentation                                                              |
| **Queries/**         | Beispiel-Queries (Joins, Mappings)                                                                   |
| **00_Archive/**      | Archivierte Vorlagen und Diagramme                                                                   |


## In Obsidian öffnen

1. [Obsidian](https://obsidian.md) herunterladen und installieren.
2. Dieses Repository klonen:
   ```bash
   git clone https://github.com/very-many/DigiChrom-Doku.git
   ```
3. Obsidian starten und **Open folder as vault** (Ordner als Vault öffnen) wählen.
4. Den geklonten Ordner `DigiChrom-Doku` auswählen.
5. Falls Obsidian fragt, ob Community-Plugins aktiviert werden sollen, mit **Trust author and enable plugins** bestätigen – die Vault-Konfiguration (`.obsidian/`) ist bereits im Repository enthalten.