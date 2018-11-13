<!--

author:   Konstantin Kirchheim

email:    konstantin.kirchheim@ovgu.de

version:  2.0.0

language: de

narrator:  Deutsch Female


@run_main
<script>
events.register("@0", e => {
		if (!e.exit)
    		send.lia("output", e.stdout);
		else
    		send.lia("eval",  "LIA: stop");
});

send.handle("input", (e) => {send.service("@0",  {input: e})});
send.handle("stop",  (e) => {send.service("@0",  {stop: ""})});


send.service("@0", {start: "CodeRunner", settings: null})
.receive("ok", e => {
		send.lia("output", e.message);

		send.service("@0", {files: {"main.c": `@input`}})
		.receive("ok", e => {
				send.lia("output", e.message);

				send.service("@0",  {compile: "gcc main.c -o a.out", order: ["main.c"]})
				.receive("ok", e => {
						send.lia("log", e.message, e.details, true);

						send.service("@0",  {execute: "./a.out"})
						.receive("ok", e => {
								send.lia("output", e.message);
								send.lia("eval", "LIA: terminal", [], false);
						})
						.receive("error", e => { send.lia("log", e.message, e.details, false); send.lia("eval", "LIA: stop"); });
				})
				.receive("error", e => { send.lia("log", e.message, e.details, false); send.lia("eval", "LIA: stop"); });
		})
		.receive("error", e => { send.lia("output", e.message); send.lia("eval", "LIA: stop"); });
})
.receive("error", e => { send.lia("output", e.message); send.lia("eval", "LIA: stop"); });

"LIA: wait";
</script>

@end







@sketch
<script>
events.register("mc_stdout", e => { send.lia("output", e); });
events.register("mc_start", e => { send.lia("eval", "LIA: terminal"); });


let compile = `arduino-builder -compile -logger=machine -hardware /usr/local/share/arduino/hardware -tools /usr/local/share/arduino/tools-builder -tools /usr/local/share/arduino/hardware/tools/avr -built-in-libraries /usr/local/share/arduino/libraries -libraries /usr/local/share/arduino/libraries -fqbn="Robubot Micro:avr:microbot" -ide-version=10807 -build-path $PWD/build -warnings=none -prefs=build.warn_data_percentage=100 -prefs=runtime.tools.avr-gcc.path=/usr/local/share/arduino/packages/arduino/tools/avr-gcc/4.8.1-arduino5/avr -prefs=runtime.tools.avr-gcc-4.8.1-arduino5.path=/usr/local/share/arduino/packages/arduino/tools/avr-gcc/4.8.1-arduino5/avr sketch/sketch.ino`;


send.service("@0arduino", {start: "CodeRunner", settings: null})
.receive("ok", e => {

		send.lia("output", e.message);
    send.service("@0arduino", {files: {"sketch/sketch.ino": `@input(0)`, "sketch/Motor.h": `@input(1)`, "sketch/Motor.cpp": `@input(2)`, "build/": ""}})
		.receive("ok", e => {

				send.lia("output", e.message);
				send.service("@0arduino",  {compile: compile, order: ["sketch.ino", "Motor.h", "Motor.cpp"]})
				.receive("ok", e => {

						send.lia("log", e.message, e.details, true);
            if(!window["bot_selected"]) { send.lia("eval", "LIA: stop"); }
            else {

              document.getElementById("button_"+window.bot_selected).disabled = true;
              send.service("c",
                { connect: [["@0arduino", {"get_path": "build/sketch.ino.hex"}], ["mc", {"upload": null, "target": window["bot_selected"]}]]
                }
              );

              send.handle("input", (e) => {
                send.service("mc",  {id: "bot_stdin",
                           action: "call",
                           params: {procedure: "com.robulab.target."+window["bot_selected"]+".send_input", args: [0,  String.fromCharCode(0)+btoa(e) ] }})});

              send.handle("stop",  (e) => {
                document.getElementById("button_"+window.bot_selected).disabled = false;

                send.service("mc", {id: "stdio0",
                                   action: "unsubscribe",
                                   params: {id: window["stdio0"], args: [] }});

                send.service("mc", {id: "stdio1",
                                   action: "unsubscribe",
                                   params: {id: window["stdio1"], args: [] }});

                send.service("mc",  {id: "bot_disconnect."+window["bot_selected"],
                           action: "call",
                           params: {procedure: "com.robulab.target.disconnect", args: [window["bot_selected"]] }})});


                delete window.stdio0;
                delete window.stdio1;
            }
				})
				.receive("error", e => { send.lia("log", e.message, e.details, false); send.lia("eval", "LIA: stop"); });
		})
		.receive("error", e => { send.lia("output", e.message); send.lia("eval", "LIA: stop"); });
})
.receive("error", e => { send.lia("output", e.message); send.lia("eval", "LIA: stop"); });

"LIA: wait";
</script>

<script>
function aduinoview_init() {

  let arduino_view_frame = document.getElementById("arduinoviewer");

  window["arduino_view_frame"] = arduino_view_frame;

  arduino_view_frame.onload = function () {

    arduino_view_frame.contentWindow.ArduinoView.init = function () {

      arduino_view_frame.contentWindow.ArduinoView.sendMessage = function(e) {

        send.service("mc",  {id: "bot_stdin2",
                   action: "call",
                   params: {procedure: "com.robulab.target."+window["bot_selected"]+".send_input", args: [1,  String.fromCharCode(0)+btoa(e) ] }});
      };

      arduino_view_frame.contentWindow.ArduinoView.onInputPermissionChanged(true);
    };
  };

  arduino_view_frame.contentWindow.ArduinoView.sendMessage = function(e) {

    send.service("mc",  {id: "bot_stdin2",
               action: "call",
               params: {procedure: "com.robulab.target."+window["bot_selected"]+".send_input", args: [1,  String.fromCharCode(0)+btoa(e) ] }});
  };

  arduino_view_frame.contentWindow.ArduinoView.onInputPermissionChanged(true);

}

function update() {
  if(!window["bot_selected"]) {
    for(let i=0; i<window.bot_list.length; i++) {
      let btn = document.getElementById("button_"+window.bot_list[i].target);

      if(btn === null) {
        let cmdi = document.getElementById("bot_list");

        btn = document.createElement("BUTTON");
        btn.id = "button_" + window.bot_list[i].target;
        btn.innerHTML = window.bot_list[i].name;
        btn.onclick = function() {
           let id = window.bot_list[i].target;

           send.service("mc", {id: "bot_connect."+id,
                      action: "call",
                      params: {procedure: "com.robulab.target.connect", args: [id] }});
         };

         cmdi.appendChild(btn);
         cmdi.appendChild(document.createTextNode(" "));
      }

      btn.style.backgroundColor = "";

      if(window.bot_list[i].owner == ""){
        btn.className = "lia-btn";
        btn.disabled = false;
      }
      else {
        btn.className = "lia-btn";
        btn.disabled = true;
      }
    }
  }
  else {
    for(let i=0; i<window.bot_list.length; i++) {
      let btn = document.getElementById("button_"+window.bot_list[i].target);
      if(window.bot_list[i].target == window["bot_selected"]){
        btn.className = "lia-btn";
        btn.disabled = false;
        btn.style.backgroundColor = "#ADFF2F";
      }
      else {
        btn.className = "lia-btn";
        btn.disabled = true;
      }
    }
  }
}


function subscriptions() {
    events.register("mc", e => {
       if(typeof(e) === "undefined")
          return;

       if(!!e.subscription) {
         if (e.subscription.startsWith("com.robulab.livestream")) {
            //console.log("vid", e.parameters.args[0].data);
            window.cam.PutData(new Uint8Array(e.parameters.args[0].data));
         }
         else if (e.subscription == "com.robulab.target.changed") {
           let args = e.parameters.args;
           for(let i=0; i<args.length; i++) {
             for(let j=0; j<window.bot_list.length; j++) {
               if(args[i].target == window.bot_list[j].target) {
                 window.bot_list[j] = Object.assign(window.bot_list[j], args[i])
               }
             }
           }
           update();
         } else if (e.subscription.endsWith(".0.raw_out")) {
           events.dispatch("mc_stdout", String.fromCharCode.apply(this, e.parameters.args[0].data));
         } else if (e.subscription.endsWith(".1.raw_out")) {
           window["arduino_view_frame"].contentWindow.ArduinoView.onArduinoViewMessage(
             String.fromCharCode.apply(this, e.parameters.args[0].data) );
         }
         else {
          //console.log("problem", e);
         }
       }
       else if(e.id == "bot_list") {
         window["bot_list"] = e.ok;
         let cmdi = document.getElementById("bot_list");

         for (let i = 0; i< window.bot_list.length; i++) {
            let btn = document.createElement("BUTTON");
            btn.id = "button_" + window.bot_list[i].target;
            btn.innerHTML = window.bot_list[i].target;
            btn.onclick = function() {
              let id = window.bot_list[i].target;

              send.service("mc", {id: "bot_connect."+id,
                           action: "call",
                          params: {procedure: "com.robulab.target.connect", args: [id] }});
              };

              cmdi.appendChild(btn);
              cmdi.appendChild(document.createTextNode(" "));

              send.service("mc", {id: "bot_name."+i,
                           action: "call",
                           params: {procedure: "com.robulab.target."+ window.bot_list[i].target +".get_name", args: [] }});
             }
           update();
         }
         else if( e.id.startsWith("bot_flash1") ) {
           let [cmd, target, file] = e.id.split(" ");
           send.service("mc", {upload: file, target: target, id: e.ok});
         }
         else if( e.id.startsWith("bot_flash2") ) {
           let [cmd, target, id] = e.id.split(" ");
           send.service("mc", {finish: parseInt(id), target: target});
         }
         else if ( e.id.startsWith("bot_flash3") && !e.ok ) {
           aduinoview_init();
           let [cmd, target, id] = e.id.split(" ");

           send.service("mc", {id: "bot_stdio.0.target",
                               action: "subscribe",
                               params: {topic: "com.robulab.target."+target+".0.raw_out", args: [] }});

           send.service("mc", {id: "bot_stdio.1.target",
                               action: "subscribe",
                               params: {topic: "com.robulab.target."+target+".1.raw_out", args: [] }});

           events.dispatch("mc_start", "");


        }

        else if (e.id.startsWith("bot_stdio")) {
          let [cmd, stdio, target] = e.id.split(".");
          window["stdio"+stdio] = e.ok.id;
        }

        else if( e.id.startsWith("bot_name") ) {
          let [cmd, id] = e.id.split(".")
          window.bot_list[id]["name"] = e.ok;
          document.getElementById("button_"+window.bot_list[id].target).innerHTML = e.ok;
        }

        else if( e.id.startsWith("bot_connect") ) {
          let [cmd, target] = e.id.split(".");
          window["bot_selected"] = target;
          let btn = document.getElementById("button_"+target);
          btn.onclick = function() {
              send.service("mc", {id: "bot_disconnect."+target,
                         action: "call",
                         params: {procedure: "com.robulab.target.disconnect", args: [target] }});
          };

          send.service("mc", {id: "bot_stream."+target,
                     action: "call",
                     params: {procedure: "com.robulab.target."+target+".get_stream", args: [target] }});

          update();

          window["cam"] = window.decoder(document.getElementById("bot_show"), ([w, h]) => {
              let canvas = document.getElementById("bot_show");

              canvas.height = h;
              canvas.width = w;
          });
        }
        else if( e.id.startsWith("bot_disconnect") ) {
          let [cmd, target] = e.id.split(".");
          delete window["bot_selected"];

          let btn = document.getElementById("button_"+target)
          btn.onclick = function() {
              send.service("mc", {id: "bot_connect."+target,
                         action: "call",
                         params: {procedure: "com.robulab.target.connect", args: [target] }});
          };

          send.service("mc", {id: "cam_disconnect",
                             action: "unsubscribe",
                             params: {id: window["streaming_id"], args: [] }});

          update();
        }
        else if ( e.id.startsWith("bot_stream") ) {
          let [cmd, target] = e.id.split(".");
          send.service("mc",
                       {id: "streaming",
                        action: "subscribe",
                        params: {topic: e.ok.url, args: [] }});
        }
        else if (e.id == "streaming") {
          window["streaming_id"] = e.ok.id;
        }

        else if ( e.id == "cam_disconnect") {
          delete window.cam;
        }

        else {
          console.log("not handled", e);
      }
      });

      send.service("mc", {id: "bot_list",
                            action: "call",
                            params: {procedure: "com.robulab.target.get-online", args: [] }});

      send.service("mc", {id: "bot_changes",
                            action: "subscribe",
                            params: {topic: "com.robulab.target.changed", args: [] }})

      send.service("mc", {id: "bot_changes",
                            action: "subscribe",
                            params: {topic: "com.robulab.target.reregister", args: [] }})


      window["mc_subscribed"] = true;

};

function login(silent=true) {
    let cmdi = document.getElementById("mcInterface");

    send.service("mc", {start: "MissionControl", settings: null})
      .receive("ok", (e) => {
          console.log("user-connected:", e);
          cmdi.hidden = false;
          window["mc_logged_in"] = true;
          subscriptions();
      })
      .receive("error", (e) => {
          cmdi.hidden = true;
          console.log("Error user-connected:", e);
          alert("Fail: Please check your login!");
      });
};

if (!window.mc_logged_in) {
  setTimeout((e) => { login() }, 300);
}
else {
  document.getElementById("mcInterface").hidden = false;
  update();
}

window.addEventListener("beforeunload", function (event) {

    if ( window.stdio1 ) {
          send.service("mc", {id: "stdio1",
                       action: "unsubscribe",
                       params: {id: window["stdio1"], args: [] }});
    }

    if ( window.stdio0 ) {
          send.service("mc", {id: "stdio0",
                       action: "unsubscribe",
                       params: {id: window["stdio0"], args: [] }});
    }

    if(window.bot_selected) {
        send.service("mc", {id: "cam_disconnect",
                           action: "unsubscribe",
                           params: {id: window["streaming_id"], args: [] }});

        send.service("mc",  {id: "bot_disconnect."+window["bot_selected"],
                   action: "call",
                   params: {procedure: "com.robulab.target.disconnect", args: [window["bot_selected"]] }});
    }

});

</script>


<div id="mcInterface" hidden="true">
  <span style="border-style: solid; width: 49.5%; float: left; min-width: 480px;">
    <span id="bot_list" ></span>
    <br>
    <iframe id="arduinoviewer" style="margin-left: 3px; width: 99%; max-height: 432px;" src="https://elab.ovgu.robulab.com/arduinoview"></iframe>
  </span>
  <span style="border-style: solid; width: 49.5%; height: 480px; float: right; min-width: 480px; overflow: auto">
    <canvas id="bot_show" style="width: calc(16 * 10vw); height: calc(9 * 10vw);"></canvas>
  </span>
</div>
@end


@init_clear
<script>
  let cmdi = document.getElementById("mcInterface");
  if(cmdi)
    cmdi.hidden = true;

  let viewer = document.getElementById("arduinoviewer");
  if(viewer)
    try {
    //  viewer.contentWindow.document.body.innerHTML = ""
    } catch(e) {}

  if ( window.stdio1 ) {
          send.service("mc", {id: "stdio1",
                       action: "unsubscribe",
                       params: {id: window["stdio1"], args: [] }});
  }

  if ( window.stdio0 ) {
          send.service("mc", {id: "stdio0",
                       action: "unsubscribe",
                       params: {id: window["stdio0"], args: [] }});
  }

  if(window.bot_selected) {
        send.service("mc", {id: "cam_disconnect",
                           action: "unsubscribe",
                           params: {id: window["streaming_id"], args: [] }});

        send.service("mc",  {id: "bot_disconnect."+window["bot_selected"],
                   action: "call",
                   params: {procedure: "com.robulab.target.disconnect", args: [window["bot_selected"]] }});
  }
</script>


@end


-->

# Einführung

Willkommen zurück im eLearning-System *eLab*.

    --{{0}}--
Die Grundlagen der Treiberentwicklung für eingebettete Systeme, die in der
letzten Aufgabe thematisiert wurden, sollen in dieser Aufgabe genutzt werden, um
einen weiteren Aktortyp auf unserer Plattform anzusteuern: die Motoren.



    --{{1}}--
Das Ziel dieser Aufgabe wird es sein, die Ansteuerung beider Motoren zu
implementieren. Wenn ihr die Aufgabe abgeschlossen habt, sollte es möglich sein,
beide Motoren unabhängig voneinander vorwärts und rückwärts, sowie in
unterschiedlichen Geschwindigkeiten, bewegen zu lassen.


## Themen und Ziele

    --{{0}}--
Die Ansteuerung von Motoren umfasst zwei Eigenschaften: die Drehrichtung sowie
die Drehgeschwindigkeit. Vor allem die Drehgeschwindigkeit wird durch die
Generierung eines entsprechenden PWM-signals
[(PWM = Pulsweitenmodulation)](https://en.wikipedia.org/wiki/Pulse-width_modulation)
beeinflusst. Daher wird dieses ein Kernthema dieser Aufgabe sein.

    --{{1}}--
PWM-Signale werden oftmals durch
[Timer/Counter](https://www.tutorialspoint.com/embedded_systems/es_timer_counter.htm)
generiert. Die Konfiguration und Programmierung dieser bildet daher ein weiteres
Thema dieser Aufgabe.

    --{{2}}--
Letztlich kann ein Motor aufgrund der auftretenden Ströme nicht direkt an die
Ausgänge eines Mikrocontrollers angeschlossen werden. Stattdessen wird ein
Motortreiber benötigt. Über diesen kann sowohl eine Geschwindigkeits-, sowie
eine Richtungssteuerung der Motoren stattfinden.

    --{{3}}--
Das Ziel der Aufgabe ist es, die zuvor angesprochenen Komponenten zu nutzen, um
eine Motoransteuerung für die beiden Motoren unserer Roboterplattform zu
implementieren. Dazu müsst ihr entsprechend einer vorgegebenen Geschwindigkeit
und Bewegungsrichtung PWM-Signale für beide Motoren generieren und sie an die
Motoren weiterleiten.

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

    --{{0}}--
Da sich diese Aufgabe mit der Ansteuerung von Elektromotoren beschäftigt, kann
es hilfreich sein, sich zunächst über deren Aufbau und ihre verschiedenen
Varianten zu informieren.

    --{{1}}--
Direkt relevant für die Ansteuerung sind vor allem H-Brücken, bzw.
Vierquadrantensteller. Diese elektrischen Schaltungen erlauben die
Drehrichtungsänderung und die Geschwindigkeitssteuerung der Motoren.

    --{{2}}--
Neben den Hardware-Komponenten, ist auch die Theorie zur Ansteuerung der Motoren
von Bedeutung. Zentral ist hier die Pulsweitenmodulation, sowie ihre Generierung
mit Hilfe von Timern/Countern.

    --{{3}}--
Wie immer findet ihr hier aber auch die spezifischen Datenblätter für die
verbauten Komponenten. Dazu kommt in dieser Aufgabe das
[Datenblatt des Motortreibers](http://www.st.com/content/ccc/resource/technical/document/datasheet/59/d0/ce/41/56/bb/4b/10/DM00034699.pdf/files/DM00034699.pdf/jcr:content/translations/en.DM00034699.pdf).

**Elektromotoren:**

* [Elektromotoren](https://en.wikipedia.org/wiki/Electric_motor)
* [Gleichstrommotoren](https://en.wikipedia.org/wiki/DC_motor)
* [H-Brücke/Vierquadrantensteller](https://de.wikipedia.org/wiki/Vierquadrantensteller)
* Einführung in Gleichstrommotoren:

  !?[Einführung in Gleichstrommotoren](https://www.youtube.com/embed/LAtPHANEfQo)<!--
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

    --{{0}}--
In der *zweiten* praktischen Aufgabe sollt ihr die Ansteuerung der Motoren
unserer Roboter-Plattform implementieren. Da es sich auch hier um eine einfache
Treiberentwicklung handelt, geben wir euch, wie auch in der letzten Aufgabe, die
Header-Datei `Motor.h` vor. Darin sind die zu implementierenden Funktionen
deklariert.

    --{{1}}--
Es bietet sich an die Funktionen in zwei verschiedenen Teilaufgaben zu
implementieren.

    --{{2}}--
Da ihr in dieser Aufgabe mit den elektrischen Motoren Kräfte auf die Umgebung
des Roboters ausüben könnt, muss zunächst die Funktionalität zum
**Deaktivieren der Motoren** implementiert werden. Erst danach solltet ihr die
Aktivierung der Motoren implementieren. So soll es euch möglich sein die Motoren
bei potenziell unsicherem Verhalten auszuschalten.

    --{{3}}--
In der zweiten Teilaufgabe wird es dann darum gehen, die eigentliche Bewegung zu
generieren.

> **Hinweis:**
>
> Verwendet bei der Bearbeitung der Aufgabe keine Funktionen aus der
> Arduino-Bibliothek. Lediglich die Funktionen der `Serial`-Klasse zur
> Ansteuerung der seriellen Schnittstelle, sowie der Funktion `millis()` zur
> einfachen Zeitmessung können genutzt werden.


**Teilaufgaben**

* *2.1* Implementiert die Aktivierung/Deaktivierung der Motoren und die
  zugehörigen Callback-Funktionen des Arduinoview-Interfaces.
* *2.2* Implementiert die Geschwindigkeits- und Drehrichtungssteuerung für beide
  Motoren.

## Aufgabe 2.1

    --{{0}}--
Wie bereits angedeutet, muss zur Bearbeitung dieser Aufgabe zunächst
sichergestellt werden, dass ihr die Motoren jederzeit stoppen könnt. Aus diesem
Grund ist eure Code-Vorlage bei dieser Aufgabe nicht kompilierfähig - ihr müsst
zunächst die Funktion `deactivateMotors()` implementieren.

    --{{1}}--
Nachdem ihr die Deaktivierung der Motoren mit der entsprechenden
Callback-Funktion verknüpft habt, gilt es im zweiten Teilschritt das Gegenstück
zu implementieren. Fügt dazu eurem Arduinoview-Interface einen *Start*-Button
hinzu und verknüpft die Callback-Funktion mit der API-Funktion
`activateMotors()`.

**Ziel:**

Implementiert die Funktionen `deactivateMotors()` und `activateMotors()` und
verknüpft sie mit den entsprechenden Callback-Funktionen im
Arduinoview-Interface.

**Teilschritte:**

1. Legt eine `Motor.cpp`-Datei an und implementiert die Funktion
   `deactivateMotors()`. Danach sollte eure Code-Vorlage kompilierfähig sein.
2. Implementiert die Funktion `activateMotors()`.
3. Fügt dem Arduinoview-Interface einen *Start*-Button hinzu und verknüpft ihn
   mit der Funktion `activateMotors()`.


## Aufgabe 2.2

    --{{0}}--
Nachdem wir nun die Motoren aktivieren und vor allem **deaktivieren** können,
besteht das Ziel der zweiten Teilaufgabe darin, die eigentliche Bewegung zu
generieren.

    --{{1}}--
Im ersten Teilschritt solltet ihr, vorbereitend auf die Generierung der
PWM-Signale, die Funktion `initMotors()` implementieren. Da zur Generierung
eines PWM-Signals Timer genutzt werden sollen, muss auch deren Initialisierungen
in dieser Funktion stattfinden. Achtet dabei darauf, dass die Frequenz der
PWM-Signale außerhalb des hörbaren Bereichs (500 Hz - 15 kHz) liegt.

    --{{2}}--
Im zweiten Teilschritt geht es darum, die Motoren zu bewegen. Dazu haben wir
euch die Funktion `setMotors(int8_t left, int8_t right)` vorgegeben. Nutzt die
Funktion um die generierten PWM-Signale entsprechend den Werten für den linken
und rechten Motor anzupassen. Achtet dabei auch auf die Drehrichtung der
Motoren.

    --{{3}}--
Zur Abnahme der Aufgabe sollen positive Werte von *left* und *right* zu einer
Vorwärtsbewegung, negative Werte zu einer Rückwärtsbewegung des jeweiligen
Motors/Rads führen (zur Orientierung: die 7-Segment-Anzeigen sind links, an der
Vorderseite des Roboters sind die Infrarot-Sensoren angebracht).

    --{{4}}--
Zur Vorgabe von Geschwindigkeits- und Richtungswerten für beide Räder, haben wir
eurem Arduinoview-Interface ein *Joystick* hinzugefügt. Durch Klicken und Ziehen
des Cursors im blauen Quadrat könnt ihr die Vorgaben kontinuierlich ändern.


**Ziel:**
Ansteuerung beider Motoren mittels PWM-Signale und Drehrichtungsvorgabe.

**Hinweise zur PWM-Generierung:**

1. Vermeidet Frequenzen im hörbaren Bereich, d.h. zwischen 500 Hz und 15 kHz.
2. Wie viele Timer sind zur PWM-Generierung für **beide** Motoren notwendig?
   Wie können wir die Zahl der genutzten Timer möglichst gering halten?

**Teilschritte:**

1. Implementiert die Funktion `initMotors()`. Achtet darauf, dass die sich für
   das PWM-Signal ergebende Frequenz außerhalb des hörbaren Bereichs
   (500 Hz - 15 kHz) liegt.
2. Implementiert die Funktion `setMotors(int8_t left, int8_t right)`.

**Hinweis Browserunterschiede:**

Der Mozilla Firefox und Chrome verhalten sich leider bei der Auswertung der
Clickevents unterschiedlich des Steuerungs Sticks. Der große HTML Block sollte
daher um die folgende Zeile vor dem SVG-Element eränzt werden.

`"<style>svg * { pointer-events: none; }</style>\n"`

## Bonusaufgabe B.1

    --{{0}}--
Da unsere Funktionalität zur Ansteuerung der Motoren lediglich auf
Timern/Counter basiert und nur das Arduinoview-Interface auf den regulären
Programmablauf zurückgreift, ist die CPU nur bei Änderungen der Geschwindigkeit
oder der Drehrichtung beteiligt. Können wir diesen Umstand nutzen, um den
Energieverbrauch unseres Systems weiter zu minimieren?

    --{{1}}--
Als Bonusaufgabe könnt ihr die verschiedenen Schlafmodi des AVR ATmega32U4
studieren. In welchem Modus wird Energie gespart ohne die zuvor implementierte
Funktionalität zu limitieren?

**Ziel:**
Versetzt die CPU in einen Ruhezustand ohne die zuvor implementierte
Funktionalität zu limitieren.

**Vorgehen:**
Modifiziert die `loop()`-Funktion um Energie zu sparen.

# **Programmierung**

```cpp  sketch.ino
// -----------------------------------------------------------------
// Exercise 2
// -----------------------------------------------------------------

#include <FrameStream.h>
#include <Frameiterator.h>
#include <avr/io.h>

#define OUTPUT__BAUD_RATE 9600
FrameStream frm(Serial1);

// Forward declarations
void InitGUI();

// hierarchical runnerlist, that connects the gui elements with
// callback methods
declarerunnerlist(GUI);

// First level will be called by frm.run (if a frame is recived)
beginrunnerlist();
fwdrunner(!g, GUIrunnerlist); //forward !g to the second level (GUI)
callrunner(!!, InitGUI);
fwdrunner(ST, stickdata);
endrunnerlist();

// GUI level
beginrunnerlist(GUI);
callrunner(es, CallbackSTOP);
endrunnerlist();

void stickdata(char* str, size_t length)
{
    // str contains a string of two numbers ranging from -127 to +127
    // indicating left and right motor values
    //TODO: interprete str, adapt the power for the left and right motor accordingly
}

void CallbackSTOP()
{
    // call deactivateMotors();
    deactivateMotors();
}

/*
 * @brief initialises the GUI of ArduinoView
 *
 * In this function, the GUI, is configured. For this, Arduinoview shorthand,
 * HTML as well as embedded JavaScript can be used.
 */
void InitGUI()
{
    frm.print(F("!h<h1>PKeS Exercise 2</h1>"));
    frm.end();

    frm.print(F("!SbesvSTOP"));
    frm.end();

    //TODO: add a START button

    //this implements a joystick field using HTML SVG and JS
    frm.print(F("!H"
                "<style>svg * { pointer-events: none; }</style>\n"
                "<div id='state'> </div>\n"
                "<svg id='stick' width='300' height='300' viewBox='-150 -150 300 300' style='background:rgb(200,200,255)' >\n"
                "<line id='pxy' x1='0' y1='0' x2='100' y2='100' style='stroke:rgb(255,0,0);stroke-width:3' />\n"
                "<circle id='cc' cx='0' cy='0' r='3'  style='stroke:rgb(0,0,0);stroke-width:3' />\n"
                "</svg>\n"
                "<script>\n"
                "var getEl=function(x){return document.getElementById(x)};\n"
                "var setEl=function(el,attr,val){(el).setAttributeNS(null,attr,val)};\n"
                "var stick=getEl('stick');\n"
                "function sticktransform(x,y){\n"
                "x = x-150;\n"
                "y = -(y-150);\n"
                "setstick(x,y);\n"
                "}\n"
                "function setstick(x,y){\n"
                "setpointer(x,y);\n"
                "l = Math.floor(Math.min(127,Math.max(-127,y + x/2)));\n"
                "r = Math.floor(Math.min(127,Math.max(-127,y - x/2)));\n"
                "setStateDisplay(x,y,l,r);\n"
                "sendframe(\"ST\"+l +\",\"+r);\n"
                "}\n"
                "function setStateDisplay(x,y,l,r){\n"
                "msg=getEl('state');\n"
                "msg.innerHTML= 'x= '+ x +' y= '+ y +' l= '+ l +' r= '+ r ;\n"
                "}\n"
                "function setpointer(x,y){\n"
                "pxy=getEl('pxy');\n"
                "setEl(pxy,'x2',x);\n"
                "setEl(pxy,'y2',-y);\n"
                "}\n"
                "stick.onmousemove=function(e){\n"
                "if( e.buttons == 1 ){\n"
                "sticktransform(e.offsetX,e.offsetY);\n"
                "}};\n"
                "stick.onmousedown=function(e){\n"
                "if( e.buttons == 1 ){\n"
                "sticktransform(e.offsetX,e.offsetY);\n"
                "}};\n"
                "stick.onmouseup=function(e){\n"
                "setstick(0,0);\n"
                "};\n"
                "stick.onmouseleave=function(e){\n"
                "setstick(0,0);\n"
                "};\n"
                "function sendframe(msg){\n"
                "ArduinoView.sendMessage(ArduinoView._createSerialFrame(msg))\n"
                "}\n"
                "setstick(0,0);\n"
                "</script>\n"));
    frm.end();
}

/*
 * @brief Initialisation
 *
 * Implement basic initialisation of the program.
 */
void setup()
{
    delay(1000);

    //prepare Serial interfaces
    Serial.begin(OUTPUT__BAUD_RATE);
    Serial1.begin(OUTPUT__BAUD_RATE);

    Serial.println(F("Willkommen zur PKeS Übung"));

    //request reset of GUI
    frm.print("!!");
    frm.end();

    delay(500);

    //call InitMotors();
}

/*
 *  @brief Main loop
 *
 *  This function will be called repeatedly and shall therefore implement the
 *  main behavior of the program.
 */
void loop()
{
    // read & run ArduinoView Frames
    while(frm.run());

}
```
``` h Motor.h
pragma once

#include <inttypes.h>

// initialise timer(s) here
void initMotors();
// control the motor-power, pay attention to the direction of rotation
void setMotors(int8_t left, int8_t right);

// you may use this to set Motor-DDRs
void deactivateMotors();
void activateMotors();
```
``` cpp Motor.cpp

```
@sketch

# Quizze

    --{{0}}--
Wie auch in der letzten Aufgabe haben wir noch ein paar kurze Fragen an euch.

## Pulsweitenmodulation

    --{{0}}--
Welche Modi der genannten Pulsweitenmodulation unterstützt der verwendete
Mikrocontroller?

    {{0-1}}
    [[X]] Fast PWM
    [[ ]] Detailed PWM
    [[X]] Phase Correct PWM
    [[ ]] Backward PWM
    [[ ]] Forward PWM


    --{{1}}--
Was wird durch *Duty Cycle* angegeben?

    {{1-2}}
    [(X)] Der Anteil eines Intervalls, in dem das PWM-Signal einen High-Pegel hat
    [( )] Der Anteil eines Intervalls, in dem das PWM-Signal einen Low-Pegel hat
    [( )] Der Anteil eines Intervalls, in dem das PWM-Signal keiner Vorgabe folgt
********************************************************************************

Weitere Informationen zum *Duty Cycle* könnt ihr [hier](https://en.wikipedia.org/wiki/Duty_cycle) finden.

********************************************************************************


    --{{2}}--
Welche Einheit hat *Duty Cycle*?

    {{2-3}}
    [( )] ms
    [( )] ns
    [(X)] %
    [( )] Hz
********************************************************************************

Weitere Informationen zum *Duty Cycle* könnt ihr [hier](https://en.wikipedia.org/wiki/Duty_cycle) finden.

********************************************************************************


    --{{3}}--
Mit welchen *Stellschrauben* kann die Frequenz des PWM-Signals angepasst werden?

    {{3}}
    [[X]] Taktquelle für den Timer
    [[X]] Prescalerkonfiguration
    [[ ]] aktueller Wert des Vergleichsregisters
    [[ ]] Konfiguration der COMnAm-Flags
    [[X]] gewählter Timer
********************************************************************************

Selbstverständlich beeinflusst die Duty-Cycle Konfiguration und die Orientierung
der Ausgabe (invertiert/nicht-invertiert) die Frequenz des PWM Signales NICHT!

********************************************************************************

## Timer

    --{{0}}--
Welchen maximalen Wert kann der Timer/Counter0 des AVR ATmega32U4 annehmen?

    {{0-1}}
    [( )] 512
    [( )] 65535
    [( )] 512
    [(X)] 255
********************************************************************************

Bei dem Timer/Counter0 handelt es sich um einen 8-Bit Timer. Daher ist der
maximale Wert durch $2^8-1$ gegeben.

********************************************************************************


    --{{1}}--
Welchen maximalen Wert kann der Timer/Counter1 des AVR ATmega32U4 annehmen?

    {{1-2}}
    [( )] 1023
    [(X)] 65535
    [( )] 512
    [( )] 255
********************************************************************************

Bei dem Timer/Counter1 handelt es sich um einen 16-Bit Timer. Daher ist der
maximale Wert durch $2^16-1$ gegeben.

********************************************************************************


    --{{2}}--
Welches Verhalten zeigt ein Timerbaustein im CTC-Modus?

    {{2}}
    [(X)] Rücksprung des Counters beim Erreichen des Vergleichswertes auf 0
    [( )] Vergleichswertunabhängige Frequenz bei den Ausgaben auf den zugerhörigen PINs
    [( )] Auf- und Abzählen des Counters
********************************************************************************

Clear-to-Compare bezeichnet einen Timermode, bei dem der Counter jeweils nach
dem Erreichen des Vergleichswertes resetet wird. Verallgemeinert kann gesagt
werden, dass der *Wecker* neu aufgezogen wird.

********************************************************************************


## Motortreiber

    --{{0}}--
Wie wird eine H-Brücke noch genannt?

    {{0-1}}
    [[ ]] Transistor Brücke
    [[X]] Vierquadrantensteller
    [[X]] Vollbrücke

    --{{1}}--
Welche Arten des Bremsens können über eine H-Brücke realisiert werden?

    {{1-2}}
    [[ ]] Stillstand, die Magnetfelder im Motor werden nicht gewechselt, die Spulen richten sich entlang dem Magnetfeld aus.
    [[X]] Leerlauf, der Motor wird durch sonstige Energieverluste langsamer.
    [[X]] Gegenschub, die Drehrichtung des Motors wird gewechselt.
    [[X]] Generatorbetrieb, der Motor wandelt die Bewegungsenergie in elektrische Energie um.

    --{{2}}--
In der konkreten Anwendung wurde ein
[STM L6206 Motortreiber](http://www.st.com/content/ccc/resource/technical/document/datasheet/59/d0/ce/41/56/bb/4b/10/DM00034699.pdf/files/DM00034699.pdf/jcr:content/translations/en.DM00034699.pdf)
integriert. Welche Aufgabe haben die mit "Sense" bezeichneten Eingänge?

    {{2-3}}
    [[ ]] Messung der internen Temperatur im Treiberbaustein
    [[ ]] Messung der Spannung an den Motoren
    [[X]] Messung des Stromflusses durch die Motoren
    [[ ]] Messung der Drehgeschwindigkeit des Motors
********************************************************************************

Die Sense-Eingänge können mit einem niederohmigen Shunt-Widerstand benutzt
werden, um den Stromfluß durch den Motor zu erfassen.

********************************************************************************

    --{{3}}--
Mit den `PROGCLA` und `PROGCLB` sind jeweils mit einem 10 kOhm Widerstand gegen
GND verbunden. Damit kann die maximale Stromstärke definiert werden, die der
Treiber liefert, bevor er sich abschaltet. Ermitteln Sie aus dem Datenblatt den
mit dem Widerstand spezifizierten Wert.

    {{3-4}}
    [[X]] 2 bis 2.5 A
    [[ ]] 3 und 3.5 A
    [[ ]] 4 und 4.5 A
********************************************************************************

Das Kapitel 4.3 beschreibt unter dem Stichwort "Non-dissipative overcurrent
detection and protection" die Wirkung und den Hintergrund des Überstromschutzes
des Treibers. Die Formel I charakterisiert das Verhalten für R = 0 und Formel II
für eine beliebigen Widerstandswert. Figure 11 fasst diese Aussage grafisch und
erlaubt es für 10kOhm den zugehörigen Wert zu ermitteln.

********************************************************************************
