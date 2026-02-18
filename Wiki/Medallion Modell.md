Das **Fabric‑Medallion‑Modell** ist ein **Datenarchitektur‑Konzept in Microsoft Fabric**, das Daten systematisch in **Iron-, Bronze-, Silver- und Gold‑Schichten** organisiert, um **Rohdaten schrittweise zu bereinigten und zu verlässlichen und analysefähigen Daten** weiterzuentwickeln.  
Man braucht es, um **Datenpipelines strukturierter, skalierbarer und besser wartbar** zu gestalten und gleichzeitig **Datenqualität, Transparenz und Wiederverwendbarkeit** sicherzustellen.

# Die Schichten
## Iron
Die [[0 Iron-Schicht|Iron-Schicht]] ist eine reine **Reninigungsschicht**.
Ziel ist es aus jeder Datei der Partner eine bereinigte Tabelle zu kreieren.
### Schritte der Iron-Schicht
- Rohdaten aus ADLS laden
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
- Daten aus Lakehouse
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
