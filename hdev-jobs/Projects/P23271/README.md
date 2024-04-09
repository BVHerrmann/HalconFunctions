# P23271 Zeichenerkennung Deep OCR

Kunde: Otto Künnecke GmbH

| Eigenschaft | Wert | Hinweis |
| --- | --- | --- |
| BDevEngine Version | - |  |
| HALCON Version | 23.05 |  |
| Lizenz | - |  |
| Host ID | 450100444734303136019f091f156700 | Lizenz-Datei eingecheckt. |

## TODO

- [x] Qualitätskontrolle: Druckfehler bei Zeichen über den Kontrast erkennen

## VisionBox

### Job

`verify`

### Prüfablauf

1. Suche nach dem DMC
2. Prüfen der Qualität des DMC
3. Das Bild anhand des DMC horizontal ausrichten
4. Erneute Suche von DMC durchführung (Die erneute Suche wird durchgeführt, um den neuen Mittelpunkt vom DMC zu bestimmen)
5. Berechnung des Umrechnungsfaktors anhand des DMC (mm zu px)
6. Qualität der einzelnen Buchstaben bewerten
7. Zeilenweise Durchführung der Texterkennung
8. Auswertung

#### Defaultparameter

| Parameter | Wert | Einheit | Hinweis |
| --- | --- | --- | --- |
| InputSN | - | - | Erwartete Seriennummer. |
| InputRow1 | - | - | Erwartete Wörter. |
| InputRow2 | - | - | Erwartete Wörter. |
| InputRow3 | - | - | Erwartete Wörter. |
| InputRow4 | - | - | Erwartete Wörter. |
| DmcSize | 11.85 | mm | Die Größe des DataMatrixCodes. Wird für die Umrechnungfaktor von Pixel in mm benötigt. |
| CharacterWidth | 1.295 | mm | Mit dem Wert wird die Breite des Textfeldes berechnet, die um ein Wort gelegt wird. |
| CharacterHeight | 1.5 | mm | Mit dem Wert wird die Höhe der des Textfeldes gebrechnet, die um ein Wort gelegt wird. |
| RowOffset | 6.775 | mm | Der Abstand von der Mitte des DMC zum Anfang des Textfeldes, dass sich am weitesten oben befindet. |
| ColumnOffset | 34.845 | mm | Der Abstand von der Mitte des DMC zum Anfang des Textfeldes, dass sich am weitesten links befindet. |
| RowDistance | 2.8 | mm | Der senkrechte Abstand zwischen zwei Textfeldern. |
| SerialNumberOffset | 15.2 | mm | Der Abstand vom Anfang des linken Textfeldes bis zum Anfang des Textfeldes mit der Seriennummer. |
| MinDataMatrixCodeQuality | 0 | - | Gibt die Druckqualität des DMC mit einer Note von 0 bis 4 an. Je höher die Zahl desto besser die Qualität. |
| Deviation | 55 | - | Gibt die mittlere Verteilung des Grauwerte eines Buchstabens an. Je höher des Wert, desto schärfer ist der Buchstabe bzw. desto besser ist die Druckqualität des Buchstaben. |

### Hardware

- IPC 520A

Die Kamera ist über ein USB-Kabel an der Rechner angeschlossen. 


