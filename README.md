<!--

author:   Konstantin Kirchheim

email:    konstantin.kirchheim@ovgu.de

version:  1.0.0

language: de_DE

narrator:  Deutsch Female

-->

# Einführung

Willkommen zurück im eLearning-System *eLab*.

--{{1}}--
Die Grundlagen der Treiberentwicklung für eingebettete System, die in der letzten Aufgabe thematisiert wurden, sollen in dieser Aufgabe genutzt werden, um einen weiteren Aktortyp auf unserer Plattform anzusteuern: die Motoren.

--{{2}}--
Das Ziel dieser Aufgabe wird es sein, die Ansteuerung beider Motoren zu implementieren. Wenn ihr die Aufgabe abgeschlossen habt, sollte es möglich sein, beide Motoren unabhängig voneinander vorwärts und rückwärts, sowie in unterschiedlichen Geschwindigkeiten, bewegen zu lassen.

## Themen und Ziele

--{{1}}--
Die Ansteuerung von Motoren umfasst zwei Eigenschaften, die Drehrichtung sowie die Drehgeschwindigkeit. Vor allem die Drehgeschwindigkeit wird durch die Generierung eines entsprechenden PWM-signals [(PWM = Pulsweitenmodulation)](https://en.wikipedia.org/wiki/Pulse-width_modulation) beeinflusst. Daher wird dieses ein Kernthema dieser Aufgabe sein.

--{{2}}--
PWM-Signale werden oftmals durch [Timer/Counter](https://www.tutorialspoint.com/embedded_systems/es_timer_counter.htm) generiert. Die Konfiguration und Programmierung dieser bildet daher ein weiteres Thema dieser Aufgabe.

--{{3}}--
Letztlich kann ein Motor aufgrund der auftretenden Ströme nicht direkt an die Ausgänge eines Mikrocontrollers angeschlossen werden. Stattdessen wird ein Motortreiber benötigt. Über diesen kann sowohl eine Geschwindigkeits-, sowie eine Richtungssteuerung der Motoren stattfinden.

--{{4}}--
Das Ziel der Aufgabe ist es, die zuvor angesprochenen Komponenten zu nutzen, um eine Motoransteuerung für die beiden Motoren unserer Roboterplattform zu implementieren. Dazu müsst ihr entsprechend einer vorgegebenen Geschwindigkeit und Bewegungsrichtung PWM-Signale für beide Motoren generieren und sie an die Motoren weiterleiten.

**Themen:**

* Pulsweitenmodulation
* Timer/Counter
* Motoransteuerung und Motortreiber

**Ziel(e):**

* Ansteuerung von Motoren:
  * PWM-Generierung
  * Vorwärts- sowie Rückwärtsbewegung
  * Einstellbare Geschwindigkeiten



## Weitere Informationen

--{{1}}--
Da sich diese Aufgabe mit der Ansteuerung von Elektromotoren beschäftigt, kann es hilfreich sein, sich zunächst über deren Aufbau und ihre verschiedenen Varianten zu informieren.

--{{2}}--
Direkt relevant für die Ansteuerung sind vor allem H-Brücken, bzw. Vierquadrantensteller. Diese elektrischen Schaltungen erlauben die Drehrichtungsänderung und die Geschwindigkeitssteuerung der Motoren.

--{{3}}--
Neben den Hardware-Komponenten, ist auch die Theorie zur Ansteuerung der Motoren von Bedeutung. Zentral ist hier die Pulsweitenmodulation, sowie ihre Generierung mit Hilfe von Timern/Countern.

--{{4}}--
Wie immer findet ihr hier aber auch die spezifischen Datenblätter für die verbauten Komponenten. Dazu kommt in dieser Aufgabe das [Datenblatt des Motortreibers](http://www.st.com/content/ccc/resource/technical/document/datasheet/59/d0/ce/41/56/bb/4b/10/DM00034699.pdf/files/DM00034699.pdf/jcr:content/translations/en.DM00034699.pdf).

**Elektromotoren:**

* [Elektromotoren](https://en.wikipedia.org/wiki/Electric_motor)
* [Gleichstrommotoren](https://en.wikipedia.org/wiki/DC_motor)
* [H-Brücke/Vierquadrantensteller](https://de.wikipedia.org/wiki/Vierquadrantensteller)
* Einführung in Gleichstrommotoren:

  !![Einführung in Gleichstrommotoren](https://www.youtube.com/embed/LAtPHANEfQo)<!--
    width: 560px;
    height: 315px;
  -->

**Pulsweitenmodulation:**

* [Wikipedia](https://en.wikipedia.org/wiki/Pulse-width_modulation)
* [Mikrocontroller.net](https://www.mikrocontroller.net/articles/Pulsweitenmodulation)
* [Motoransteuerung mit PWM](https://www.mikrocontroller.net/articles/Motoransteuerung_mit_PWM)

**Timer:**

* [Tutorial](https://www.mikrocontroller.net/articles/AVR-Tutorial:_Timer)
* [Unterschied zwischen Timer und Counter](https://www.tutorialspoint.com/embedded_systems/es_timer_counter.htm)

**PKeS:**

* [Datenblatt Motortreiber](http://www.st.com/content/ccc/resource/technical/document/datasheet/59/d0/ce/41/56/bb/4b/10/DM00034699.pdf/files/DM00034699.pdf/jcr:content/translations/en.DM00034699.pdf)
* [Schaltbelegungsplan](https://github.com/liaScript/PKeS0/blob/master/materials/robubot_stud.pdf?raw=true)
* [Arduinoview](https://github.com/fesselk/Arduinoview/blob/master/doc/Documetation.md)
* [Datenblatt des AVR ATmega32U4](http://www.atmel.com/Images/Atmel-7766-8-bit-AVR-ATmega16U4-32U4_Datasheet.pdf)

# Aufgabe 2

--{{1}}--
In der *zweiten* praktischen Aufgabe sollt ihr die Ansteuerung der Motoren unserer Roboter-Plattform implementieren. Da es sich auch hier um eine einfache Treiberentwicklung handelt, geben wir euch, wie auch in der letzten Aufgabe, die Header-Datei *Motor.h* vor. Darin sind die zu implementierenden Funktionen deklariert.

--{{2}}--
Es bietet sich an die Funktionen in zwei verschiedenen Teilaufgaben zu implementieren.

--{{3}}--
Da ihr in dieser Aufgabe mit den elektrischen Motoren Kräfte auf die Umgebung des Roboters ausüben könnt, muss zunächst die Funktionalität zum **Deaktivieren der Motoren** implementiert werden. Erst danach solltet ihr die Aktivierung der Motoren implementieren. So soll es euch möglich sein die Motoren bei potenziell unsicherem Verhalten auszuschalten.

--{{4}}--
In der zweiten Teilaufgabe wird es dann darum gehen, die eigentliche Bewegung zu generieren.

**Hinweis:**

**Verwendet bei der Bearbeitung der Aufgabe keine Funktionen aus der Arduino-Bibliothek. Lediglich die Funktionen der `Serial`-Klasse zur Ansteuerung der seriellen Schnittstelle, sowie der Funktion `millis()` zur einfachen Zeitmessung können genutzt werden.**


**Teilaufgaben**

* *2.1* Implementiert die Aktivierung/Deaktivierung der Motoren und die zugehörigen Callback-Funktionen des Arduinoview-Interfaces.
* *2.2* Implementiert die Geschwindigkeits- und Drehrichtungssteuerung für beide Motoren.

Die Aufgabe ist bis zu der Woche vom **27.11.-01.12.2017** vorzubereiten.

## Aufgabe 2.1

--{{1}}--
Wie bereits angedeutet, muss zur Bearbeitung dieser Aufgabe zunächst sichergestellt werden, dass ihr die Motoren jederzeit stoppen könnt. Aus diesem Grund ist eure Code-Vorlage bei dieser Aufgabe nicht kompilierfähig - ihr müsst zunächst die Funktion `deactivateMotors` implementieren.

--{{2}}--
Nachdem ihr die Deaktivierung der Motoren mit der entsprechenden Callback-Funktion verknüpft habt, gilt es im zweiten Teilschritt das Gegenstück zu implementieren. Fügt dazu eurem Arduinoview-Interface einen *Start*-Button hinzu und verknüpft die Callback-Funktion mit der API-Funktion `activateMotors`.

**Ziel:**
Implementiert die Funktionen `deactivateMotors` und `activateMotors` und verknüpft sie mit den entsprechenden Callback-Funktionen im Arduinoview-Interface.

**Teilschritte:**

1. Legt eine *Motor.cpp*-Datei an und implementiert die Funktion `deactivateMotors()`. Danach sollte eure Code-Vorlage kompilierfähig sein.
2. Implementiert die Funktion `activateMotors`.
3. Fügt dem Arduinoview-Interface einen *Start*-Button hinzu und verknüpft ihn mit der Funktion `activateMotors()`.


## Aufgabe 2.2

--{{1}}--
Nachdem wir nun die Motoren aktivieren und vor allem **deaktivieren** können, besteht das Ziel der zweiten Teilaufgabe darin, die eigentliche Bewegung zu generieren.

--{{3}}--
Im ersten Teilschritt solltet ihr, vorbereitend auf die Generierung der PWM-Signale, die Funktion `initMotors()` implementieren. Da zur Generierung eines PWM-Signals Timer genutzt werden sollen, muss auch deren Initialisierungen in dieser Funktion stattfinden. Achtet dabei darauf, dass die Frequenz der PWM-Signale außerhalb des hörbaren Bereichs (500 Hz - 15 kHz) liegt.

--{{4}}--
Im zweiten Teilschritt geht es darum, die Motoren zu bewegen. Dazu haben wir euch die Funktion `setMotors(int8_t left, int8_t right)` vorgegeben. Nutzt die Funktion um die generierten PWM-Signale entsprechend den Werten für den linken und rechten Motor anzupassen. Achtet dabei auch auf die Drehrichtung der Motoren. 

--{{5}}--
Zur Abnahme der Aufgabe sollen positive Werte von *left* und *right* zu einer Vorwärtsbewegung, negative Werte zu einer Rückwärtsbewegung des jeweiligen Motors/Rads führen (zur Orientierung: die 7-Segment-Anzeigen sind links, an der Vorderseite des Roboters sind die Infrarot-Sensoren angebracht). 

--{{6}}--
Zur Vorgabe von Geschwindigkeits- und Richtungswerten für beide Räder, haben wir eurem Arduinoview-Interface ein *Joystick* hinzugefügt. Durch Klicken und Ziehen des Cursors im blauen Quadrat könnt ihr die Vorgaben kontinuierlich ändern.


**Ziel:**
Ansteuerung beider Motoren mittels PWM-Signale und Drehrichtungsvorgabe.

**Hinweise zur PWM-Generierung:**

1. Vermeidet Frequenzen im hörbaren Bereich, d.h. zwischen 500 Hz und 15 kHz.
2. Wie viele Timer sind zur PWM-Generierung für **beide** Motoren notwendig? Wie können wir die Zahl der genutzten Timer möglichst gering halten?

**Teilschritte:**

1. Implementiert die Funktion `initMotors()`. Achtet darauf, dass die sich für das PWM-Signal ergebende Frequenz außerhalb des hörbaren Bereichs (500 Hz - 15 kHz) liegt.
2. Implementiert die Funktion `setMotors(int8_t left, int8_t right)`. 

## Bonusaufgabe B.1

--{{1}}--
Da unsere Funktionalität zur Ansteuerung der Motoren lediglich auf Timern/Counter basiert und nur das Arduinoview-Interface auf den regulären Programmablauf zurückgreift, ist die CPU nur bei Änderungen der Geschwindigkeit oder der Drehrichtung beteiligt. Können wir diesen Umstand nutzen, um den Energieverbrauch unseres Systems weiter zu minimieren?

--{{2}}--
Als Bonusaufgabe könnt ihr die verschiedenen Schlafmodi des AVR ATmega32U4 studieren. In welchem Modus wird Energie gespart ohne die zuvor implementierte Funktionalität zu limitieren?

**Ziel:**
Versetzt die CPU in einen Ruhezustand ohne die zuvor implementierte Funktionalität zu limitieren.

**Teilschritte:**

1. Modifiziert die `loop()`-Funktion um Energie zu sparen.

# Quizze

--{{1}}--
Wie auch in der letzten Aufgabe haben wir noch ein paar kurze Fragen an euch.

## Pulsweitenmodulation

**Welche Modi der genannten Pulsweitenmodulation unterstützt der verwendeten Mikrocontroller?**


  [[X]] Fast PWM
  [[ ]] Detailed PWM
  [[X]] Phase Correct PWM
  [[ ]] Backward PWM
  [[ ]] Forward PWM
  
**Was wird durch *Duty Cycle* angegeben?**


  [(X)] Der Anteil eines Intervalls, in dem das PWM-Signal einen High-Pegel hat
  [( )] Der Anteil eines Intervalls, in dem das PWM-Signal einen Low-Pegel hat
  [( )] Der Anteil eines Intervalls, in dem das PWM-Signal keiner Vorgabe folgt
  [[[

Weitere Informationen zum *Duty Cycle* könnt ihr [hier](https://en.wikipedia.org/wiki/Duty_cycle) finden.

]]]

**Welche Einheit hat *Duty Cycle*?**


  [( )] ms
  [( )] ns
  [(X)] %
  [( )] Hz
  [[[

Weitere Informationen zum *Duty Cycle* könnt ihr [hier](https://en.wikipedia.org/wiki/Duty_cycle) finden.

]]]

**Mit welchen *Stellschrauben* kann die Frequenz des PWM-Signals angepasst werden?**


  [[X]] Taktquelle für den Timer
  [[X]] Prescalerkonfiguration
  [[ ]] aktueller Wert des Vergleichsregisters
  [[ ]] Konfiguration der COMnAm-Flags
  [[X]] gewählter Timer
  [[[

Selbstverständlich beeinflusst die Duty-Cycle Konfiguration und die Orientierung der Ausgabe (invertiert/nicht-invertiert) die Frequenz des PWM Signales NICHT!

]]]

## Timer

**Welchen maximalen Wert kann der Timer/Counter0 des AVR ATmega32U4 annehmen?**


    [( )] 512
    [( )] 65535
    [( )] 512
    [(X)] 255
    [[[
    
Bei dem Timer/Counter0 handelt es sich um einen 8-Bit Timer. Daher ist der maximale Wert durch $2^8-1$ gegeben.

]]]

**Welchen maximalen Wert kann der Timer/Counter1 des AVR ATmega32U4 annehmen?**


    [( )] 1023
    [(X)] 65535
    [( )] 512
    [( )] 255
    [[[

Bei dem Timer/Counter1 handelt es sich um einen 16-Bit Timer. Daher ist der maximale Wert durch $2^16-1$ gegeben.

]]]

**Welches Verhalten zeigt ein Timerbaustein im CTC-Mode?**

    [(X)] Rücksprung des Counters beim Erreichen des Vergleichswertes auf 0
    [( )] Vergleichswertunabhängige Frequenz bei den Ausgaben auf den zugerhörigen PINs
    [( )] Auf- und Abzählen des Counters
    [[[

Clear-to-Compare bezeichnet einen Timermode, bei dem der Counter jeweils nach dem Erreichen des Vergleichswertes resetet wird. Verallgemeinert kann gesagt werden, dass der *Wecker* neu aufgezogen wird.

]]]

  
## Motortreiber

**Wie wird eine H-Brücke noch genannt?**

  [[ ]] Transistor Brücke
  [[X]] Vierquadrantensteller
  [[X]] Vollbrücke
  

**Welche Arten des Bremsens können über eine H-Brücke realisiert werden?**

  [[ ]] Stillstand, die Magnetfelder im Motor werden nicht gewechselt, die Spulen richten sich entlang dem Magnetfeld aus.
  [[X]] Leerlauf, der Motor wird durch sonstige Energieverluste langsamer.
  [[X]] Gegenschub, die Drehrichtung des Motors wird gewechselt.
  [[X]] Kurzschluss, der Motor induziert eine seiner eigenen Bewegung entgegenstehende Spannung.
  
**In der konkreten Anwendung wurde ein [STM L6206 Motortreiber](http://www.st.com/content/ccc/resource/technical/document/datasheet/59/d0/ce/41/56/bb/4b/10/DM00034699.pdf/files/DM00034699.pdf/jcr:content/translations/en.DM00034699.pdf) integriert. Welche Aufgabe haben die mit "Sense" bezeichneten Eingänge?**

  [[ ]] Messung der internen Temperatur im Treiberbaustein
  [[ ]] Messung der Spannung an den Motoren
  [[X]] Messung des Stromflusses durch die Motoren
  [[ ]] Messung der Drehgeschwindigkeit des Motors
  [[[
    
Die Sense-Eingänge können mit einem niederohmigen Shunt-Widerstand benutzt werden, um den Stromfluß durch den Motor zu erfassen. 

]]]

**Mit den PROGCLA und PROGCLB sind jeweils mit einem 10kOhm Widerstand gegen GND verbunden. Damit kann die maximale Stromstärke definiert werden, die der Treiber liefert, bevor er sich abschaltet. Ermitteln Sie aus dem Datenblatt den mit dem Widerstand spezifizierten Wert.**


  [[X]] 2 bis 2.5 A
  [[ ]] 3 und 3.5 A
  [[ ]] 4 und 4.5 A
  [[[
  
Das Kapitel 4.3 beschreibt unter dem Stichwort "Non-dissipative overcurrent detection and protection" die Wirkung und den Hintergrund des Überstromschutzes des Treibers. Die Formel I charakterisiert das Verhalten für R = 0 und Formel II für eine beliebigen Widerstandswert. Figure 11 fasst diese Aussage grafisch und erlaubt es für 10kOhm den zugehörigen Wert zu ermitteln.

]]]

