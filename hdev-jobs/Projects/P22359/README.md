# P22359 Teilmaschine 11 & 12

## Beschreibung Glaskantenerkennung und lesen Datamatrixcode

Kunde: Schott AG

| Eigenschaft | Wert | Hinweis |
| --- | --- | --- |
| BDevEngine Version | - |  |
| HALCON Version | 23.05 |  |
| Lizenz | - |  |
| Host ID | - |  |


### OP1.30-BX1 Kameraname für die Erkennung der Glaskanten
### Allgemein:

Auf einer Glasplatte soll laufend die Kantenposition bestimmt werden. Die Außenkanten der Platte werden im Wechsel vermessen einmal die linke Seite und dann die rechte Seite. Jede Seite hat ihre eigene Beleuchtung. Bei der Vermessung der linken Kante wird die rechte Beleuchtung eingeschalten und beim Vermessen der rechten Kante die linke Beleuchtung.
Um maximale Messgenauigkeit für diesen Kameraaufbau zu erzielen, muss dieser entsprechend kalibriert werden. Die Kamerakalibrierung korrigiert die Objektivverzerrungen und ermöglicht die Umrechnung von Bildkoordinaten zu Weltkoordinaten. Die Kamerakalibrierung ist der Regel ein einmaliger Vorgang. Sie wird einmal am Anfanag durchgeführt.
Die Kamerakalibrierung muss wiederholt werden:

1. Änderungen am Objektiv durchgeführt werden (Blende, Focus).
2. Die Position der Kamera/Beleuchtung verändert wird.
3. Kameraparameter werden verändert. Belichtungszeit, ROI, ... etc.
4. Die Kamera wird ausgetauscht.
5. Der Abstand Kamera zu Prüfobjekt ändert sich.

Die Kamerakalibrierung ist in der Regel zweistufig.

1. Bestimmung der internen Kameraparameter
2. Bestimmung der externen Kameraparameter

Interne Kameraparameter:

```text
Die internen Kameraparameter beschreiben die Beschaffenheit der verwendeten Kamera, insbesondere die interne Sensorgröße und die Abbildungseigenschaften der verwendeten Kombination von Objektiv und Kamera.
```

Externe Kameraparameter:

```text
Die externen Kameraparameter beschreiben die Transformation von Welt- in Kamerakoordinaten bzw. den Zusammenhang zwischen 3D Kamera zu 3D Weltkoordinaten.
```
Hier in dieser Anwendung werden die internen und externen in einem Durchlauf berechnet.

### Beschreibung der Kalibrierroutinen

Die Kamera für die Glaskantenerkennung ist eine Zeilenkamera. Die Kamera hat eine Auflösung von 4096 Bildpunkten. Dies entspricht einer Breite von ca. 1300 mm. Die hier verwendete Kalibrierplatte hat 37 Marken in x-Richtung und 9 in y-Richtung. Die Marken haben einen Durchmesser von 10 mm und einen Abstand von 20 mm. Die Kalibrierplatte kann mit einem Plotter/Drucker ausgedruckt werden. Die Datei befindet sich im Jobverzeichnis unter dem Namen CalibrationPlate9_37.pdf

![Bild einer Kalibrierplatte](BildKalibrierplatte.png "")

**Bild1: Kalibrierplatte**

> **_Hinweis:_**  Wird die Kalibrierung gestartet erzeugt das Programm eine Postscript Datei mit dem Namen CalibrationPlate.ps. Die PS-Datei kann dann in ein pdf-Datei konvertiert werden. Die Geometrie der Platte ist im Programm fest hinterlegt. Siehe "Erstellen einer Kalibrieplatte zum Ausdrucken".

Für die Berechnung der Kameraparameter sollte eine Serie von Bildern des Kalibrierobjektes in unterschiedlichen Orientierungen aufgenommen werden. Durch die Serie von Kalibrierbildern sollte jeder Teil des Kamerasichtfeldes mindestens einmal durch eine Platte abgedeckt werden. Die Kalibrierplatte kann dabei auch das gesamte Sichtfeld ausfüllen. Die Orientierung der Platte sollte innerhalb der Serie von Bildern variieren.

### Ablauf Berechnung Interner Kameraparameter für die Zeilenkamera

Vor dem Start der Kalibrierung muss der Trigger Mode der Kamera auf Off gesetzt werden, Beleuchtung Links und Rechts gleichzeitig an und die Bandgeschwindigkeit auf 40mm/sec. Die Kamera ist so eingestellt das immer ein Block von 4096 * 256 Bildpunkten übertragen wird. In der Kalibrierphase werden 4 Blöcke zu einem Gesamtbild zusammengesetzt. Die Platte muss vollständig im Bild sichtbar sein damit sie erkannt wird.

Die Kalibrierung erfolgt in drei Schritten:

1. Schritt Initialisierung
2. Schritt Aufnahme mehrerer Bilder (Mindestens 6 Bilder)
3. Schritt Kalibrierung abschließen und Daten speichern

### Beschreibung der einzelnen Prozeduren/Schritte

Schritt 1.
Initialisierungroutine:
### Name
init_line_scan_calibration_stripe_position
### Signatur
init_line_scan_calibration_stripe_position(Image, JobPass, MeasureTimeInMs)
### Beschreibung
- Übergabeparameter  
  - Image: Eingangsbild
- Rückgabewerte
  - JobPass: Wenn keine Platte gefunden steht, hier eine 0 ansonsten eine 1
  - MeasureTimInMs: Rechenzeit des Algorithmus  

Die Initialisierungsroutine instanziieren die Objekte, erstellt die Datei für Kalibrieplatte und löscht evtl. vorhanden alte Daten. Die Funktion liefert erst JobPass gleich eins zurück, wenn das Kalibrierobjekt gefunden wurde. Ist JobPass 1 dann erfolgt der Jobwechsel zu Schritt 2.  

Schritt 2.
Aufnahme mehrerer Bilder
### Name
evaluate_line_scan_calibration_images
### Signatur
evaluate_line_scan_calibration_images (InputImage, JobPass, MarksDiameterScore, OverexposureScore, ContrastScore, HomogeneityScore, FocusScore, NumImageScore, TiltScore, FielOfVieScore, MeanSquareError, CurrentImages, MeasureTimeInMs)
                 
### Beschreibung
- Übergabeparameter  
  - Image: Eingangsbild
- Rückgabewerte
  - JobPass: Wenn Platte gefunden steht, hier eine 1 ansonsten eine 0
  - MarksDiameterScore
  - OverexposureScore
  - ContrastScore
  - HomogeneityScore
  - FocusScore
  - NumImageScore
  - TiltScore
  - FieldOfViewCoverageScore
  - MeanSqaureError

Da der Trigger Mode auf Off gesetzt ist, liefert die Kamera kontinuierlich Bilder mit einer Auflösung von 4096 * 256 Bildpunkten 4 Blöcke werden dann zu einem Gesamtbild zusammengesetzt. In dem Gesamtbild sucht die Prozedur nach der Kalibrierplatte. Wird eine Platte gefunden, berechnet die Funktion mehrere Bildqualitätsmerkmale, die die Qualität der aufgenommenen Kalibrieplatte beschreiben.
Die Qualitätsmerkmale sind in zwei Gruppen aufgeteilt.

1. Merkmale die die Bildqualität beschreiben
2. Merkmale die eine Serie von Bildern beschreiben

- Merkmale für die Bildqualität:

  - **MarksDiamterScore**

    Hier wird geprüft, ob der gemessene Markendurchmesser innerhalb der Toleranz liegt. Wenn der gemessene Markendurchmesser unter 15 Bildpunkten liegt, wird als Score 0% zurückgegeben. Ab einem Markendurchmesser über 20 Bildpunkten dann immer 100%.  

  - **OverexposureScore**

     Hier wird berechnet, ob die aufgenommene Kalibrieplatte überbelichtet oder unterbelichtet ist. Dazu wird vorab in der Initialisierungsfunktion das optimale schwarz-weiß Verhältnis der Kalibrierplatte berechnet. Das Gemessene und das Optimale schwarz-weiß Verhältnis wird in einem Score umgerechnet.

  - **ContrastScore**

    Für jede einzelne Messmarke auf der Kalibrierplatte wird an der Kontur der Marke die schwarz-weiß Änderung berechnet und dann über alle Messmarken gemittelt. Dieser Wert wird dann in einem Score umgerechnet.

  - **HomogeneityScore**

    Beschreibt die Homogenität der Messmarken bezüglich ihrer Helligkeit.

  - **FocusScore**

    Berechnung des Gradienten mit einem Gauß Filter. Über alle Messmarken wird ein mittlerer Gradient berechnet und der dann in einem Score umgewandelt wird.

- Merkmale für eine Sequenz von Bildern:

  - **NumImageScore**

    Zählt die erfolgreich aufgenommen Bilder. Je mehr Bilder aufgenommen desto größer der zurückgegebene Score.
    Erst ab einer Aufnahme von 6 Bildern ist der zurückgegeben Score größer als 0.

  - **TiltScore**

    Gibt an wie viele gekippte Bilder schon aufgenommen. Je mehr Bilder aufgenommen desto größer der zurückgegebene Score.
  
  - **FieldOfViewCoverageScore**

    Gibt an wie gut die einzelnen Kalibrierplatten über das ganze Sichtfeld der Kamera verteilt sind.

Es müssen mindestens 6 Bilder aufgenommen werden ansonsten kann die Kalibrierung nicht abgeschlossen werden. Wenn 6 aufgenommen und der FieldOfViewCoverageScore auf über 75% steht, kann die Kalibrierung mit dem nächsten Schritt beendet werden. Der Tiltscore hat hier für diese Anwendung keine Bedeutung. Alle anderen Qualitätsmerkmale dienen nur der Orientierung und Beschreiben wie gut die Beleuchtung und Kamera eingestellt sind.

Schritt 3.
Kalibrierung abschließen und Daten speichern.
### Name
save_line_scan_camera_parameter_stripe_position
### Signatur
save_line_scan_camera_parameter_stripe_position(Image, JobPass)
### Beschreibung
- Übergabeparameter  
  - Image: Eingangsbild
- Rückgabewerte
  - JobPass: Im Fehlerfall steht hier eine 0 ansonsten eine 1

save_line_scan_camera_parameter_stripe_position führt die eigentliche Kalibrierung durch und speichert die Daten für die internen und externen Kameraparameter.

> **_Hinweis:_** Schritt 2 und 3 können nur gestartet werden, wenn Schritt 1 erfolgreich durchlaufen. Wird nach Schritt 2, Schritt 1 gestartet, werden alle bis dahin aufgenommen Bilder verworfen. Nur Schritt 3 führt die eigentliche Kalibrierung durch und überschreibt die alten Daten. Nachdem die Prozedur die Kalibriedaten erfolgreich gespeichert hat, muss der Trigger Mode der Kamera auf On gesetzt werden. Wird die Funktion von Schritt3 nochmals aufgerufen, erfolgt eine Fehlermeldung, da beim Ersten erfolgreichen Aufruf die Kalibrierobjekte gelöscht wurden.

Typisches Beispiel einer Aufnahmesequenz ![Bild Sequenz Kalibrierbilder](SequenzVonKalibriermarken.png "Bilder Kalibrierplatte")
**Bild2: Beispiel einer Aufnahmesequenz hier 9 x 9**

Prinzip der Kamerakalibrierung Anordnung ![Zkizze Kalibrierung Zeilenkamera](BildKalibrierungZeilenKamera.png "Prinzip Kamerakalibrierung mit Zeilenkamera")
**Bild3: Prinzip Kamerakalibrierung mit Zeilenkamera**

## Beschreibung Erkennung Glaskante

### Allgemein:
   Die Berechnung der Glaskante funktioniert nur wenn Kalibrierdaten vorhanden. Nur mit den Kalibrierdaten ist es möglich die Bildkoordinaten in Weltkoordinaten umzurechnen. Wird die Prozedur aufgerufen und es sind keine Kalibriedaten vorhanden, wird eine Fehlermeldung erzeugt eine Messung ist dann nicht möglich.

   Der Nullpunkt befindet sich in der Mitte des Bildes. Von hier aus wird der Abstand berechnet. Liegt die Glaskante links (in Laufrichtung des Bandes) vom Nullpunkt wird ein negativer Wert an die Steuerung übertragen, liegt die Kante rechts dann ein Positiver. Die Steuerung gibt vor welche Seite vermessen wird.
Für die Bestimmung der Glaskante steht folgende Prozedur zur Verfügung:
### Name
glas_edge_detection
### Signatur
glas_edge_detection (InputImage, LeftSide, RoiLeftOffsetInMM, RoiLeftWidthInMM, RoiRightOffsetInMM, RoiRightWidthInMM, EdgeTreshold, JobPass, CurrentPosInMM, LastPosInMM, DistanceInMM, EdgeResult, SecondEdgeResult, CameraLeft)
### Beschreibung
- Übergabeparameter
  - Image: Eingangsbild
  - LeftSide: Wenn LeftSide = 1 dann Kante von der linken Seite beginnend suchen, wenn 0 dann Kante von der rechten Seite aus beginnend suchen
  - RoiLeftOffsetInMM: Startposition linkes Messfenster
  - RoiLeftWidthInMM: Breite linkes Messfenster
  - RoiRightOffsetInMM: Startposition rechtes Messfenster
  - RoiRightWidthInMM: Breite rechtes Messfenster
  - EdgeThreshold: Minimaler Kantenschwellwert
- Rückgabeparameter
  - JobPass: Im Fehlerfall steht hier eine 0 ansonsten eine 1
  - CurrentPosInMM: Aktuelle Position der Kante
  - LastPosInMM: Vorherige Position der Kante
  - DistanceInMM: Abstand der Kanten
  - EdgeResult: Die gemessene Kantenstärke/Amplitude
  - SecondEdgeResult: Zweitgrößter Amplitudenwert
  - CameraLeft: Info für die Steuerung

Die Steuerung gibt mit LeftSide vor welche Seite vermessen wird. Die 4 nachfolgenden Parameter bestimmen den Messfensteroffset und die Messfensterbreite pro Seite. Wobei der Offset der Abstand von der Nullposition aus gesehen ist. Siehe Bild 4 und Bild 5.

Vermessung linke Glaskante ![Bild Vermessung linke Glaskante](LinkeGlaskante.png "Bilde linke Glaskante")
**Bild 4: Beispiel linke Glaskante rechte Beleuchtung an**

Vermessung rechte Glaskante ![Bild Vermessung rechte Glaskante](RechteGlaskante.png "Bilde rechte Glaskante")
**Bild 5: Beispiel rechte Glaskante linke Beleuchtung an**

Die grüne Line im Bild ist die Nullposition. Die Position der Glaskante ist der Abstand Kante (Gelbes Kreuz) zum Nullpunkt. Befindet sich die Kante auf der linken Seite wird ein negativer Wert zurückgegeben, ansonsten ein positiver.

Das Bild innerhalb des gelben Messfensters ist ein vorverarbeitetes Bild. Dort wird eine Kontrastverbesserung und eine Histogrammspreizung durchgeführt.

EdgeThreshold ist der minimale Kantenschwellwert, der erreicht werden muss damit die Kante als Kante erkannt wird.

Neben den berechneten Kantenposition wird auch noch die zuletzt gemessene Position zurückgegeben und daraus die Glasbreite berechnet.

EdgeResult ist die Kante mit dem größten Kontrast. Zusätzlich wird noch der zweithöchste Kantenkontrast ausgegeben, wenn er über dem Schwellwert liegt. Dieser Wert dient nur zur Orientierung für die Bestimmung der Kantenschwelle (EdgeThreshold), wenn sie existiert, wird die Position als kleines gelbes Kreuz mit eingezeichnet.

CameraLeft beschreibt die Zugehörigkeit der Messergebnisse. Steht dort eine 1 bedeutet das die Messergebnisse zu der linken Seite gehören und wenn eine 0 dann zur rechten Seite. Dies ist eine zusätzliche Information für die Steuerung.

## Beschreibung Setzen der Zeilenfrequenz der Kamera

Die Auflösung in Transportrichtung wird durch die Zeilenfrequenz der Kamera bestimmt, die festlegt, wie viele Zeilen pro Sekunde generiert werden sollen. Sie lässt sich aus der Bandgeschwindigkeit und gewünschter Auflösung erreichen.

```text
 Zeilenfrequenz = V[mm/sec]/PixelSize[mm]
```
Mit:

```text
     V[mm/sec]     Geschwindigkeit des Transportbandes
     PixelSize[mm] Bildpunktgröße einer Bildzeile in Millimeter
```

 Jede Änderung der Geschwindigkeit des Transportbandes muss dann immer auch die Zeilenfrequenz neu gesetzt werden. Den aktuellen Wert der Zeilengröße für Glaskantererkennung liefert die Prozedur:

### Name
get_line_size_in_millimeter_stripe_position
### Signatur
get_line_size_in_millimeter_stripe_position(Image, JobPass, LineSizeInMillimeter)
### Beschreibung
- Übergabeparameter
  - Image: Eingangsbild
- Rückgabeparameter
  - JobPass: Im Fehlerfall steht hier eine 0 ansonsten eine 1
  - LineSizeInMillimeter: Größe einer Bildzeile in Millimeter

Die berechnete Zeilenfrequenz muss dann als Kameraparameter unter dem Namen 'AcquisitionLineRateAbs' gesetzt werden. Die hier zurückgegebene Zeilengröße liegt bei 0,345 mm. Dieser Wert wird in der Kalibrierroutine berechnet. Die Belichtungszeit für die Kamera zur Glaskantenerkennung ist auf 0,4 ms eingestellt. Die maximale Geschwindigkeit beträgt 600 mm/sec daraus ergibt sich eine maximal einzustellende Zeilenfrequenz von:

```text
 1739[Zeilen/sec] = 600[mm/sec]/0,345[mm]
```
Bei der eingestellten Belichtungszeit von 0,4ms pro Zeile ergibt die eine Gesamtzeit von:

```text
 696[ms] = 0.4[ms]*1739[Zeilen]
```
Dieser Wert liegt unter einer Sekunde und so ist sichergestellt, dass bei der maximalen Bandgeschwindigkeit alle Zeilen aufgenommen werden.


## Erstellen einer Kalibrieplatte zum Ausdrucken

### Name
gen_calibration_plate
## Signatur
gen_calibration_plate(Image, XNum, YNum, MarkDistInMM, DiameterRatio, FileName)
## Beschreibung
- Übergabeparameter
  - Image: Eingangsbild
  - XNum: Anzahl Marken in x
  - YNum: Anzahl Marken in y
  - MarkDistinMM: Abstand der Marken in Millimeter
  - DiameterRatio: Ist das Verhältnis Durchmesser zu Markenabstand. Durchmesser = MarkDist * DiameterRatio
  - FileName: Name der Postscriptdatei zum Ausdrucken
- Rückgabeparameter
  - JobPass: Im Fehlerfall steht hier eine 0 ansonsten eine 1

Die Datei befindet sich nach dem Aufruf im aktuellen Jobverzeichnis.

Momentan wird eine Kalibrierplatte mit folgender Geometrie verwendet:
XNum = 37
YNum = 9
MarkDistInMM  = 20.0
DiameterRatio = 0.5
Dies entspricht einer Platte wie oben in Bild 1 gezeigt.

  
### OP2.30-BX1 Kameraname für das Lesen des Datamatrixcodes
### Allgemein:

Während der Produktion wird ein Datamatrixcode auf das Glas gedruckt. Die Kamera überprüft, ob der Datamatrixcode lesbar ist. Die hier verwendete Zeilenkamera hat eine Auflösung von 8096 Bildpunkten bei eine Sichtfeld von ca. 570 mm. Die Kamera ist so eingestellt, dass sie eine Blocklänge von 1200 hat. Dies entspricht dann einer Auflösung von 8096 x 1200. Daraus ergibt sich eine Bildhöhe von ca. 85 mm in dem der Code gesucht wird.

> **_Hinweis:_** Eine Kamerakalibrierung ist hier nicht zwingend erforderlich, da hier keine Umrechnung von Pixelkoordinaten zu Weltkoordinaten benötigt wird. Die Linsenverzerrungen am Rand des Bildes durch das Objektiv können hier in diesem Fall vernachlässigt werden. Die Bildpunktgröße ergibt sich dann hier aus der Auflösung der Kamera und dem Sichtfeld.

Prozedur für das Lesen des Datamatrixcodes
### Name
find_data_matrix_line_scan
## Signatur
find_data_matrix_line_scan (InputImage, SymbolType, RoiOffsetInMM, RoiWidthInMM, JobPass, ResultCode, CodeXPosInMM, Contrast, CellDefects, ContrastSNR, PatternDefects)
              
## Beschreibung
- Übergabeparameter
  - Image: Eingangsbild
  - SymbolType: hier "Data Matrix ECC 200"
  - RoiOffsetInMM: Position des Messfensters in Millimeter
  - RoiWidthInMM: Messfensterbreite in Millimeter
- Rückgabeparameter
  - JobPass: Im Fehlerfall steht hier eine 0 ansonsten eine 1
  - ResultcCode: Wenn JobPass=1 dann steht hier der gelesene Text
  - CodeXposInMM: Aktuelle Codeposition, dient dazu die Messfensterposition genau zu setzen
  - Contrast, CellDefects, ContrastSNR, PattenDefects: Sind Qualitätsmerkmale, die beschreiben, wie gut der Code gefunden wurde.

![Bild gefundener Datamatrixcode](Datamatrixcode.png "Bilde gefundener Datamatrixcode")

**Bild 4: Beispiel Datamatrixcode erkannt**

Der Suchbereich kann über die Parameter RoiOffsetInMM und RoiWidthInMM eingeschränkt werden. Der Nullpunkt befindet sich links oben im Kamerabild. 

Nachdem der Code gefunden wurde, liefern die Qualitätsmerkmale, Informationen darüber wie gut der Code gedruckt wurde bzw. gelesen.

Der Contrast gibt den prozentualen Kontrast zwischen den als hell bzw. dunkel klassifizierten Pixeln an.

ContrastSNR ist das entsprechende Signal-Rausch-Verhältnis. 

Der Wert für CellDefects gibt den Prozentsatz der Modulpixel an, die falsch klassifiziert wurden.

Der Wert für PatternDefects gibt den Prozentsatz der Findern pattern Pixel an, die nicht als Markierungen klassifiziert werden würden.

Der oben im Bild gefundene Code hat einen Kontrast von 63% ein Signal-Rausch-Verhältnis von 2.7 und bei PatternDefects einen Wert von 40%.

## Beschreibung Setzen der Zeilenfrequenz der Kamera

Wie oben beschrieben muss auch hier beim Ändern der Bandgeschwindigkeit auch die Zeilenfrequenz der Kamera geändert werden. Den aktuellen Wert der Bildpunktgröße für die Kamera liefert die Prozedur:

### Name
get_line_size_in_millimeter_DMC
### Signatur
get_line_size_in_millimeter_DMC(Image, JobPass, LineSizeInMillimeter)
### Beschreibung
- Übergabeparameter
  - Image: Eingangsbild
- Rückgabeparameter
  - JobPass: Im Fehlerfall steht hier eine 0 ansonsten eine 1
  - LineSizeInMillimeter: Größe einer Bildzeile in Millimeter

> **_Hinweis:_**  Die berechnete Zeilenfrequenz muss dann als Kameraparameter unter dem Namen 'AcquisitionLineRateAbs' gesetzt werden. Die Kamera hier hat eine Auflösung von 8096 Bildpunkte bei einem Sichtfeld von ca. 570 mm. Daraus ergibt sich eine Bildpunktgröße von 0,071 mm. Die Belichtungszeit ist auf 0,11 ms eingestellt. Die maximale Geschwindigkeit beträgt 600 mm/sec daraus ergibt sich eine maximal einzustellende Zeilenfrequenz von:

```text
 8450[Zeilen/sec] = 600[mm/sec]/0,071[mm]
```

Bei der eingestellten Belichtungszeit von 0,11 ms pro Zeile ergibt die eine Gesamtzeit von:

```text
 930[ms] = 0.11[ms]*8450[Zeilen]
```
Dieser Wert liegt unter einer Sekunde und so ist sichergestellt, dass bei der maximalen Bandgeschwindigkeit alle Zeilen aufgenommen werden.

## Beschreibung der Proceduren Allgemeine Form

Die hier werden die Prozeduren beschrieben der allgemeinen From, sie werden von den oben beschriebenen Prozeduren aufgerufen und können für andere Projekte genutzt werden.  

## Beschreibung Allgemeine Form für die Kamerakalibrierung
### Name
init_line_scan_calibration
### Signatur
init_calibrate_camera (Image, CameraType, FocusInMillimeter, SxInMicrometer, SyInMicrometer, Vx, Vy, Vz, XNum, YNum, MarkDistInMM, DiameterRatio, JobPass, MeasureTimeInMs)
### Beschreibung
- Übergabeparameter  
  - Image: Eingangsbild
  - CameraType: 'line_scan_division' oder 'area_scan_division'. Zeilekamera oder Flächenkamera
  - FocusInMillimeter: Brennweite des Objektivs
  - SxInMicrometer, SyInMicrometer: Kamerasensorgröße
  - Vx,Vy,Vz: Bewegungsrichtung Einheit mm/pixel. Nur für Zeilenkamera
  - XNum, YNum, MarkDistInMM, DiameterRatio: Geometrie der Kalibrierplatte
- Rückgabewerte
  - JobPass: Wenn keine Platte gefunden steht, hier eine 0 ansonsten eine 1
  - MeasureTimInMs: Benötigte Rechenzeit zum Suchen der Kalibrierplatte.
  
### Name
evaluate_calibration_images
### Signatur
evaluate_calibration_images (Image, WarningThresholdInPercent, NoImageTest, NoSequenceTest, JobPass, MarksDiameterScore, OverexposureScore, ContrastScore, HomogeneityScore, FocusScore, NumImagesScore, TiltScore, FieldOfViewScore, MeanSquareError, CurrentImages, MeasureTimInms)</l>
### Beschreibung
- Übergabeparameter  
  - Image: Eingangsbild
   - WarningThresholdInPercent: Schwellwert für die Beurteilung der Qualitätsmerkmale
   - NoImageTest: Wrid der Wert auf 0 gesetzt wird JobPass nur dann eine 1 gesetzt, wenn alle Merkmale über den   Schwellwert liegen
   - NoSequenceTest:Wrid der Wert auf 0 gesetzt wird JobPass nur dann eine 1 gesetzt wenn alle Merkmale über den   Schwellwert liegen
- Rückgabewerte
  - JobPass: Wenn Platte gefunden steht, hier eine 1 ansonsten eine 0
  - MarksDiameterScore
  - OverexposureScore
  - ContrastScore
  - HomogeneityScore
  - FocusScore
  - NumImageScore
  - TiltScore
  - FieldOfViewCoverageScore
  - MeanSqaureError

### Name
save_intern_camera_parameter
### Signatur
save_intern_camera_parameter(Image, FileNameInternalCamParameter, FileNameExternalCamParameter, JobPass)
### Beschreibung
- Übergabeparameter  
  - Image: Eingangsbild
  - FileNameInternalCamParameter: Dateiname für die internen Kameraparameter
  - FileNameExternalCamParameter: Dateiname für die externen Kameraparameter
- Rückgabewerte
  - JobPass: Im Fehlerfall steht hier eine 0 ansonsten eine 1

### Name
find_data_matrix
## Signatur
find_data_matrix (RoiImage, SymbolType, JobPass, ResultCode, CodeXPos, CodeYPos, Contrast, CellDefects, ContrastSNR, PatternDefects)
## Beschreibung
- Übergabeparameter
  - Image: Eingangsbild
  - SymbolType: hier "Data Matrix ECC 200"
- Rückgabeparameter
  - JobPass: Im Fehlerfall steht hier eine 0 ansonsten eine 1
  - ResultcCode: Wenn JobPass=1 dann steht hier der gelesene Text
  - CodeXpos: X-Codeposition in Pixel
  - CodeYpos: Y-Codeposition in Pixel
  - Contrast, CellDefects, ContrastSNR, PattenDefects: Sind Qualitätsmerkmale, die beschreiben, wie gut der Code gefunden wurde.