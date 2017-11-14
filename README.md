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
Die Grundlagen der Treiberentwicklung für eingebettete System, die in der letzten Aufgabe thematisiert wurden, sollen in dieser Aufgabe genutzt werden, um einen weiteren Aktortyp auf unserere Plattform anzusteuern: den Motor.

--{{2}}--
Das Ziel dieser Aufgabe wird es sein, die Ansteuerung beider Motoren zu implementieren. Wenn ihr die Aufgabe abgeschlossen habt, sollte es möglich sein, beide Motoren unabhängig voneinander vorwärts und rückwärts, sowie in unterschiedlichen Geschwindigkeiten, bewegen zu lassen.

--{{3}}--
Dementsprechend ist das Hauptthema in dieser Aufgabe, die Motoransteuerung. 

## Themen und Ziele

--{{1}}--
Die Ansteuerung von Motoren geschieht im wesentlichen durch die Generierung eines PWM-signals [(PWM = Pulsweitenmodulation)](https://en.wikipedia.org/wiki/Pulse-width_modulation). Daher wird dieses ein Kernthema dieser Aufgabe sein.

--{{2}}--
Da ihr zur Ansteuerung der beiden Motoren entsprechende PWM-Signale generieren müsst, wird ein weiteres Kernthema die Programmierung von [Timer/Counter](https://www.tutorialspoint.com/embedded_systems/es_timer_counter.htm) sein.

--{{3}}--
Letztlich kann ein Motor aufgrund der auftretenden Ströme nicht direkt an die Ausgänge eines Mikrocontrollers angeschlossen werden. Stattdessen wird ein Motortreiber benötigt. Trotzdem die Nutzung des Treibers transparent geschieht, bildet auch dieser eine Schwerpunkt in dieser Aufgabe.

--{{4}}--
Das Ziel der Aufgabe ist es, die zuvor angesprochenen Komponenten zu nutzen, um eine Motoransteuerung für die beiden Motoren unserer Roboterplattform zu implementieren. Dazu müsst ihr entsprechend einer Bewegungsrichtung und Geschwindigkeitsvorgabe ein PWM-Signal generieren und es an den Motor(-treiber) weiterleiten.

**Themen:**
* Motoransteuerung und Motortreiber
* Pulsweitenmodulation
* Timer/Counter

**Ziel(e):**
* Ansteuerung von Motoren:
  * PWM-Generierung
  * Vorwärts- sowie Rückwärtsbewegung
  * Einstellbare Geschwindigkeiten



## Weitere Informationen

**Elektromotoren:**
* [Elektromotoren](https://en.wikipedia.org/wiki/Electric_motor)
* [Gleichstrommotoren](https://en.wikipedia.org/wiki/DC_motor)
* Einführung in Gleichstrommotoren:

  !![Einführung in Gleichstrommotoren](https://www.youtube.com/embed/LAtPHANEfQo)<!--
    width: 560px;
    height: 315px;
  -->

**Pulseweitenmodulation:**
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

**Hinweis:**

**Verwendet bei der Bearbeitung der Aufgabe keine Funktionen aus der Arduino-Bibliothek. Lediglich die Funktionen der `Serial`-Klasse zur Ansteuerung der seriellen Schnittstelle, sowie der Funktion `millis()` zur einfachen Zeitmessung können genutzt werden.**


**Teilaufgaben**

* *2.1* Implementiert die Aktivierung/Deaktivierung der Motoren und die zugehörigen Callback-Funktionen des Arduinoview-Interfaces
* *2.2* Implementiert die Generierung eines PWM-Signals zur Geschwindigkeits Regelung beider Motoren.

## Aufgabe 2.1

**Ziel:**

**Teilschritte:**

## Aufgabe 2.2

**Ziel:**

**Teilschritte:**


# Quizze

--{{1}}--
Wie auch in der letzten Aufgabe haben wir noch ein paar kurze Fragen an euch.

## Pulsweitenmodulation

**Welche Arten der Pulsweitenmodulation können unterschieden werden?**

  [[X]] Fast PWM
  [[ ]] Detailed PWM
  [[X]] Clear on Compare Match PWM
  [[X]] Phase Correct PWM
  [[ ]] Backward PWM
  [[ ]] Forward PWM
  
**Was wird durch *Duty Cycle* angegeben?**

  [(X)] Der Anteil eines Intervalls, in dem das PWM-Signal einen High-Pegel hat
  [( )] Der Anteil eines Intervalls, in dem das PWM-Signal einen Low-Pegel hat
  [( )] Der Anteil eines Intervalls, in dem das PWM-Signal keiner Vorgabe folgt
  
  
**Welche Einheit hat *Duty Cycle*?**

  [( )] ms
  [( )] ns
  [(X)] %
  [( )] Hz
  
## Timer

**Welchen maximalen Wert kann der Timer/Counter1 des AVR ATmega32U4 annehmen?**


    [( )] 1000
    [(X)] 65535
    [( )] 65536
    [( )] 32768
    [( )] 42
    [[[

Trotzdem 42 eine Antwort auf das Lebens, das Universum, auf Alles ist, können wir die Antwort hier nicht gelten lassen.

]]]
  
  
## Motortreiber


