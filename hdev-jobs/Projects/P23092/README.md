# P23092 Prüfarbeitsplatz Sauganschluss

Kunde: Kayser Automotive Systems Klodzko Sp.z.o.o.

| Eigenschaft | Wert | Hinweis |
| --- | --- | --- |
| BDevEngine Version | - |  |
| HALCON Version | 23.11 | |
| Lizenz | - |  |
| Host ID | 03534453453332478092f2cae0015a00 | Lizenz-Datei eingecheckt. |

## TODO


## VisionSensor Cube

ID: VSPV-2211-120195

### Job
Die Procedur enthält zwei Prüfschritte.

1. Prüfung ist O-Ring vorhanden oder nicht.
2. Ist Datamatrixcode lesbar oder nicht.

Wenn eine der Prüfungen NIO ist, dann wird in JobPass eine 0 zurückgegeben. Nur wenn beide IO dann steht in JobPass eine 1. Wenn JobPass 0 zurückgegeben wurde ist entweder der O-Ring nicht vorhanden oder der Code ist nicht lesbar oder beides.

### Beschreibung der Prozedur

Für die Prüfung des O-Rings müssen zwei Messfenster positioniert werden, da das Bauteil grau oder schwarz sein kann. Das eine Messfenster befindet sich an der Position des O-Rings und das Zweite auf dem Bauteil. Das Messfenster auf dem Bauteil prüft, ob das Bauteil schwarz oder grau ist.

#### Wenn Bauteil schwarz

Ist der Grauwert an der Position des O-Ringes innerhalb von PartValueMin und PartValueMax, wird JobPass 0 zurückgegeben. Ist der Grauwert größer als PartValueMax wird JobPass 1 zurückgegeben.  

#### Wenn Bauteil grau

Ist der Grauwert an der Position des O-Ringes innerhalb von ORingValueMin und ORingValueMax, wird JobPass 1 zurückgegeben. Ist der Grauwert kleiner als ORingValueMin wird JobPass 0 zurückgegeben.

### Name
check_oring_and_code
### Signatur
check_oring_and_code(Image, CodeType, PartRoiX, PartRoiY, RoiWidth, RoiHeight, PartValueMin, PartValueMax, ORingRoiX, ORingRoiY, ORingValueMin, ORingValueMax, JobPass, PartValue, ORingValue, IsPartBlack, CodeResult)
### Beschreibung
- Übergabeparameter  
  - Image: Eingangsbild
  - CodeType: Hier steht der Code Type. Default ist 'QR Code'
  - PartRoiX: X-Position des Messfensters (linke obere Ecke) für das Bauteil. Default 870
  - PartRoiY: Y-Position des Messfensters (linke obere Ecke) für das Bauteil. Default 230
  - RoiWidth: Breite des Messfensters für das Bauteil und den O-Ring. Default 450
  - RoiHeight: Höhe des Messfensters für das Bauteil und den O-Ring. Default 100
  - PartValueMin: Minimaler Grauwert für das Bauteil. Default 0
  - PartValueMax: Maximaler Grauwert für das Bauteil. Default 60
  - ORingRoiX: X-Position des Messfensters (linke obere Eck) für den O-Ring. Default 870
  - ORingRoiY: Y-Position des Messfensters (linke obere Eck) für den O-Ring. Default 430
  - ORingValueMin: Minimaler Grauwert für den O-Ring. Default 90
  - ORingValueMax: Maximaler Grauwert für den O-Ring. Default 255
- Rückgabewerte
  - JobPass: Wenn O-Ring vorhanden und Code lesbar steht hier eine 1 ansonsten eine 0
  - PartValue: Gemessener mittlerer Grauwert des Bauteils
  - ORingValue: Gemessener mittlerer Grauwert des O-Rings
  - IsPartBlack: Ist Bauteil schwarz dann 1 ansonsten 0
  - CodeResult: Ergebnis Datamatrixcode, wenn Code nicht lesbar ist der Inhalt von CodeResult leer.

> **_Hinweis:_** Die Einheit der Messfensterposition, Messfensterbreite und Messfensterhöhe ist Pixel.

Die Messfenster sind in drei gleich große Bereiche aufgeteilt. Der mittlere Teil wird für die Berechnung des Grauwertes nicht verwendet. Da das Bauteil rund ist entstehen im mittleren Teil durch die Auflicht Beleuchtung starke Reflektionen. Fällt der gemessenen Wert für das Bauteil in den Bereich von PartValueMin und PartValueMax, wird das Bauteil als schwarz klassifiziert, ist der gemessene Wert größer als PartValueMax wird es als grau klassifiziert.
Ist der gemessene Wert für den O-Ring zwischen ORingValueMin und ORingValueMax wird IO ausgegeben. Der Wert für PartValueMin bleibt auf 0 und der Wert für ORingMax bleibt auf 255. Die Werte für PartValueMax und ORingValueMin müssen evtl. angepasst werden, da sie von der eingestellten Blende und der Belichtungszeit der Kamera abhängig sind.

![Bild Messfensteranordnung](BildORingMessfenster.png "Bild Messfensteranordnung")

**Bild 1: Beispiel Messfensteranordnung graues Bauteil**

Das Messfenster wird durch das linke obere Eck (PartRoiX, PartRoiY), die Breite in X (PartRoiWidth) und Höhe in Y (PartRoiHeight) beschrieben. Die X-Position beider Messfenster sollte identisch sein (Siehe Bild 1).
