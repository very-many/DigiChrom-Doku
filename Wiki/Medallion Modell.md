Das **Fabric‑Medallion‑Modell** ist ein **Datenarchitektur‑Konzept in Microsoft Fabric**, das Daten systematisch in **(Iron-) Bronze-, Silver- und Gold‑Schichten** organisiert, um **Rohdaten schrittweise zu bereinigten, verlässlichen und analysefähigen Daten** weiterzuentwickeln.  
Man braucht es, um **Datenpipelines strukturierter, skalierbarer und besser wartbar** zu gestalten und gleichzeitig **Datenqualität, Transparenz und Wiederverwendbarkeit** sicherzustellen.

# Die Schichten
## Iron
Die [[0 Iron-Schicht|Iron-Schicht]] ist eine reine **Hilfsschicht**. Sie existiert nicht wie die anderen Schichten für jeden Partner. Sie existiert nur dann, wenn ein Partner seine Daten in mehreren Dateien liefert (um besser damit umzugehen falls eine der Dateien fehlt) (siehe: BC)
### Schritte der Iron-Schicht
Die Schritte der Bronzeschicht, außer Merge und Rename, also:
- Rohdaten laden
- Rohdaten bereinigen
	- Zellen Trimmen
	- Errors Nullen
	- Header Zeile finden und Promoten
	- Nicht verwendbare Spalten/Zeilen Löschen
- Getrennte Daten (über Tabellenblätter) Joinen
	- Anhand von IDs alles zu einer Tabelle Joinen
- Sanitize names

## Bronze
Ziel der [[1 Bronze-Schicht|Bronze-Schicht]] ist es Daten zu **bereinigen** und zu **vereinheitlichen**.
### Schritte der Bronze-Schicht
- Rohdaten laden
- Rohdaten (und Mapping Tabelle) bereinigen
	- Zellen Trimmen
	- Errors Nullen
	- Header Zeile finden und Promoten
	- Nicht verwendbare Spalten/Zeilen Löschen
- Getrennte Daten (über Dateien oder Tabellenblätter) Mergen oder Joinen
	- Anhand von IDs alles zu einer Tabelle Joinen
- Schema Bauen
	- 1 - n Beziehungen Auflösen (Spalten duplizieren die im Mapping mehrfach vorkommen)
	- Einheiten hinzufügen
	- Spalten nach Mapping benennen/umbenennen
- Sanitize names
## Silver
Ziel der [[2 Silver-Schicht|Silver-Schicht]] ist es die **Daten unter den Partnern** miteinander zu **vereinheitlichen**.


## Gold
Ziel der [[3 Gold-Schicht|Gold-Schicht]] ist es die Daten **speziell** für die Use-Cases **aufzubereiten**.
### Use-Cases
Bzw. welche "Exports" haben wir? 
- Append der Partner Paare
- Append aller Partner
- Partner einzeln

-> Alles jeweils  mit Ontologie- und Klarnamen

# Zusammenfassung der Arbeitsschritte

| X Iron | 🥉 Bronze               | 🥈 Silver | 🥇 Gold |
| ------ | ----------------------- | --------- | ------- |
| 1.     | 1. Daten Laden<br>2. Da |           |         |
