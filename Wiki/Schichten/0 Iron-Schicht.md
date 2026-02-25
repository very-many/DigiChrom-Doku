Die **Iron-Schicht** ist eine reine **Reninigungsschicht**.
Ziel ist es aus jeder Datei der Partner eine bereinigte Tabelle zu kreieren.
### Schritte der Iron-Schicht
- Rohdaten aus ADLS (Blobstorage) laden
- Rohdaten bereinigen
	- Partnerspezifische Reinigungsfunktion
- Getrennte Daten (über Tabellenblätter) Joinen
	- Anhand von IDs alles zu einer Tabelle Joinen

# Aufgabe
Aufgabe der Typischen [[0 Iron-Schicht|Iron-Schicht]] ist es:
1. [[Partner-Spezifischen Reinigungsschritte|Partner Spezifischen Reinigungsschritt]] auf jedes Tabellenblatt **einer Datei** anwenden
2. Tabellenblätter anhand IDs (Primär, Sekundär, ...) joinen

*diese Schritte passieren in der Regel innerhalb einer [[Join Query v2|Query]]*
## Beispiel
Anhand von [[Betz-Chrom]]:
![[Pasted image 20260120160428.png]]

Die Query, welche hier BC heißt, ist die [[Join Query v2|Join Query]] 



# Funktionen
![[0 Iron Functions.base]]