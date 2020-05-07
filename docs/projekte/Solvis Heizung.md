# Solvis Heizung via Modbus auslesen und steuern
Eine [Solvis Heizung](https://www.solvis.de) lässt sich mit Hilfe der Solvis Remote fernsteuern. Die Solvis Remote bildet im Prinzip das Touchdisplay der Heizung 1:1 via Browser nach. Zusätzlich gibt es Graphen über den Temperaturverlauf der einzelnen Sensoren des Schichtenspeichers. Aber zusätzlich lässt sich darüber die Heizung via Modbus TCP auslesen und, falls gewünscht, auch steuern. 

Vorbildlich ist die Bereitstellung aller Dokumentationen inkl. Modbus Register von Solvis auf deren Webseite. 


## Home Assistant Übersicht
In Home Assistant wurde die Ansicht der Anlage auf der Solvis Remote nachgebildet
![Heizung Übersicht](../img/hass-heizung-uebersicht.png)

Zusätzlich gibt es eine Detail-Ansicht der einzelnen Sensoren und Stati.
![Heizung Sensoren](../img/hass-heizung-detail.png)

Klick auf die einzelnen Sensoren bringen die Historie des Sensors bzw. die Auswahlmöglichkeiten zur Steuerung, z.b. den HKR1-Betriebsmodus zu ändern

![Heizung Detail1](../img/hass-heizung-wwpuffer-history.png) 
![Heizung Detail2](../img/hass-heizung-hkr1-betriebsart.png)

## Langzeitauswertung mit Grafana/InfluxDB
Für Langzeitstatistiken werden die Daten von Home Assistant in einer InfluxDB Datenbank gespeichert und mittels Grafana visualisiert.
![Heizung Grafana](../img/heizung-grafana.png)

Für die monatliche Auswertung der Brennerstarts und Laufzeit des Brenners werden mit folgenden Queries die Daten ermittelt:
```
SELECT spread("value") FROM "Starts" WHERE ("entity_id" = 'brennerstarts') AND time > now()  - 365d GROUP BY time(30d) fill(null)

SELECT spread("value") FROM "h" WHERE ("entity_id" = 'laufzeit_brenner') AND time > now() - 365d  GROUP BY time(30d) fill(null)
```

## benötigte Hardware
* Solvis Max Heizung
* Solvis Remote zur Anbindung ans Netzwerk und Bereitstellung des Modbus-TCP Protokolls

## Aktivierung Modbus in Solvis Heizung
Zur Aktivierung von Modbus TCP muss in das Installateurmenü gewechselt werden. Den zugehörigen Code werde ich nicht veröffentlichen. Bitte mit dem Installateur der Heizung Kontakt aufnehmen. 
Im Menüpunkt "Sonstiges" --> "Remote" auswählen und dort dann den Modus für "Modbus TCP" auf "lesen" oder auch "schreiben" einstellen.
Sollten die Einstellungen korrekt sein sollte unter der IP-Addresse der Solvis-Remote auf Port TCP/502 ein Connect möglich sein.

![Solvis Remote1](../img/solvis-remote-1.png)

![Solvis Remote2](../img/solvis-remote-2.png)

## Auszug configuration.yaml
```
modbus:
  name: SolvisRemote
  type: tcp
  host: <ip-addresse-solvis-remote>
  port: 502

 - platform: modbus
    scan_interval: 30
    registers:
    - name: Warmwasserpuffer
      hub: SolvisRemote
      unit_of_measurement: °C
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33024

    - name: Warmwassertemperatur
      hub: SolvisRemote
      unit_of_measurement: °C
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33025

    - name: Speicherreferenztemperatur
      hub: SolvisRemote
      unit_of_measurement: °C
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33026

    - name: Heizungspuffertemperatur oben
      hub: SolvisRemote
      unit_of_measurement: °C
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33027

    - name: Aussentemperatur
      hub: SolvisRemote
      unit_of_measurement: °C
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33033

    - name: Heizungspuffertemperatur unten
      hub: SolvisRemote
      unit_of_measurement: °C
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33032

    - name: Zirkulationsdurchfluss
      hub: SolvisRemote
      unit_of_measurement: °C
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33034

    - name: Vorlauftemperatur
      hub: SolvisRemote
      unit_of_measurement: °C
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33035

#    kein Sensor angeschlossen, zeigt immer 10°C  
#    - name: Kaltwassertemperatur
#      hub: SolvisRemote
#      unit_of_measurement: °C
#      scale: 0.1
#      slave: 1
#      precision: 2
#      register_type: input
#      register: 33038
      
    - name: Durchfluss Warmwasserzirkualation
      hub: SolvisRemote
      unit_of_measurement: l/min
      slave: 1
      precision: 2
      scale: 0.1
      register_type: input
      register: 33041

    - name: Laufzeit Brenner
      hub: SolvisRemote
      unit_of_measurement: h
      slave: 1
      precision: 0
      register_type: input
      register: 33536

    - name: Brennerstarts
      hub: SolvisRemote
      unit_of_measurement: Starts 
      slave: 1
      precision: 0
      register_type: input
      register: 33537

    - name: Brennerleistung
      hub: SolvisRemote
      unit_of_measurement: kW
      scale: 0.1
      slave: 1
      precision: 2
      register_type: input
      register: 33539

    - name: Ionisationsstrom
      hub: SolvisRemote
      unit_of_measurement: mA 
      slave: 1
      scale: 0.1
      precision: 1
      register_type: input
      register: 33540

    - name: VersionSC2
      hub: SolvisRemote 
      slave: 1
      scale: 0.01
      precision: 2
      register_type: input
      register: 32770

    - name: VersionNBG
      hub: SolvisRemote 
      slave: 1
      scale: 0.01
      register_type: input
      register: 32771

    - name: ZirkulationBetriebsart
      hub: SolvisRemote 
      slave: 1
      register_type: input
      register: 2049

    - name: A01.Pumpe Zirkulation
      hub: SolvisRemote 
      slave: 1
      unit_of_measurement: V
      scale: 0.01
      precision: 0
      register_type: input
      register: 33280
 
    - name: A02.Pumpe Warmwasser
      hub: SolvisRemote 
      slave: 1
      unit_of_measurement: V
      scale: 0.01
      precision: 0
      register_type: input
      register: 33281

    - name: A03.Pumpe HK1
      hub: SolvisRemote 
      slave: 1
      unit_of_measurement: V
      scale: 0.01
      precision: 0
      register_type: input
      register: 33282

    - name: A12.Brennerstatus
      hub: SolvisRemote 
      slave: 1
      unit_of_measurement: V
      scale: 0.01
      precision: 0
      register_type: input
      register: 33291

    - name: DigIn Stoerungen
      hub: SolvisRemote 
      slave: 1
      register_type: input
      register: 33045
     
    - name: HKR1 Betriebsart
      hub: SolvisRemote 
      slave: 1
      register_type: holding
      register: 2818
      
    - name: WW Solltemperatur
      hub: SolvisRemote 
      unit_of_measurement: °C
      slave: 1
      register_type: holding
      register: 2305
     
    - name: HKR1 Solltemperatur Tag
      hub: SolvisRemote 
      unit_of_measurement: °C
      slave: 1
      register_type: holding
      register: 2820
      
    - name: HKR1 Absenktemperatur Nacht
      hub: SolvisRemote 
      unit_of_measurement: °C
      slave: 1
      register_type: holding
      register: 2821
      
      
      
  - platform: template
    sensors:
      ww_zirkulationsart:
       friendly_name: "WW Zirkulation Betriebsart"
       entity_id: sensor.zirkulationbetriebsart
       value_template: >-
          {% if states('sensor.zirkulationbetriebsart') == '1' %}
            Aus
          {% elif states('sensor.zirkulationbetriebsart') == '2' %}
            Puls
          {% elif states('sensor.zirkulationbetriebsart') == '3' %}
            Temp
          {% elif states('sensor.zirkulationbetriebsart') == '4' %}
            Warten
          {% else %}
            unbekannt
          {% endif %}
            
        
      hkr1betriebsart:
        friendly_name: "Heizkreislauf Betriebsart"
        entity_id: sensor.hkr1_betriebsart
        value_template: "{%if states.sensor.hkr1_betriebsart.state == '1' %}Aus{% elif states.sensor.hkr1_betriebsart.state == '2' %}Automatik{% elif states.sensor.hkr1_betriebsart.state == '3' %}Tagbetrieb{% elif states.sensor.hkr1_betriebsart.state == '4' %}Absenkbetrieb{% elif states.sensor.hkr1_betriebsart.state == '5' %}Standby{% elif states.sensor.hkr1_betriebsart.state == '6' %}Eco{% elif states.sensor.hkr1_betriebsart.state == '7' %}Urlaub{% elif states.sensor.hkr1_betriebsart.state == '8' %}WW Vorang{% elif states.sensor.zirkulationbetriebsart.state == '9' %}Frostschutz{% elif states.sensor.zirkulationbetriebsart.state == '10' %}Pumpenschutz{% elif states.sensor.hkr1_betriebsart.state == '11' %}Estrich{% endif %}"
         
   
```

## Automatisierungen

### Stundenweises umschalten von Automatik auf Standby-Betrieb

![stundenweises Standby HKR1](../img/hass-heizung-automate-standby.png)

Gerade in der Übergangszeit kann es vorkommen, dass an sonnigen Tagen trotz noch kühler Außentemperatur es in den Zimmern durch die Sonneneinstrahlung bereits warm ist. Die Heizung aber noch wegen der kühlen Außentemperatur weiterhin läuft bis auch die Außentemperatur entsprechend später am Tag steigt.

Hier wäre ein Nutzungsszenario für folgende Automatisierung:
Schalte den Heizkreis auf Standby-Modus für x-Stunden (z.B. bis Abends).

Realisiert ist dies über einen `timer` in Home-Assistant, dessen Zeitwert (Stunden) per `input_number` gesetzt wird (Schieberegler im Frontend). Über ein `input_boolean` (Ein-/Ausschalter im Frontend) wird dann eine Automatisierung ausgeführt, die den Timer startet (`service: timer.start`) und das Modbus Register 2818 auf den Wert `5` (HKR1 Standbymodus) schreibt.

Wenn der Timer abgelaufen ist, startet eine zweite Automatisierung (getriggert durch `event_type: timer.finished`) die dann den Ein-/Ausschalter wieder auf "Aus" setzt.

Durch das setzen des Ein-Ausschalters auf "Aus" durch diese Automatisierung wird wiederum eine weitere Automatisierung getriggert die dann zum Einen das Modbus-Register 2818 wieder auf `2` (Automatik) setzt, also den Heizkreis wieder in den normalen Automatikmodus versetzt. Der zweite Anwendungsfall ist das manuelle Umschalten des Ein-/Ausschalters in der Weboberfläche auf "Aus" um wieder manuell auf "Automatik" umzuschalten. Auch hier triggert diese Automatisierung. 
#### Konfiguration
```
# Eingabefelder 
input_number:
  delayheizungstandby:
    name: Stunden auf Standby
    initial: 5
    min: 0
    max: 8
    step: 1
    icon: mdi:timer
    
input_boolean:
  heizungstandbyon:
    name: Heizung stundenweise auf Standby
    initial: off


# Timer for setting heating on standby for x hours
timer:
  heizungaufstandbyfor:
    name: Noch auf Standby für


automation:

  - alias: Heizung auf Standby Start
    trigger:
      platform: state
      entity_id: input_boolean.heizungstandbyon
      to: 'on'
    action:
    -  service: timer.start
       entity_id: timer.heizungaufstandbyfor
       data_template:
         duration: "{{ states('input_number.delayheizungstandby') | int * 60 * 60 }}"
    - service: modbus.write_register
      data:
        address: 2818
        hub: SolvisRemote
        unit: 1
        value: 5    
        
        
  - alias: Heizung Standby To Automatic
    trigger:
      platform: state
      entity_id: input_boolean.heizungstandbyon
      to: 'off'
    action:
    - service: timer.cancel
      entity_id: timer.heizungaufstandbyfor
    - service: modbus.write_register
      data:
        address: 2818
        hub: SolvisRemote
        unit: 1
        value: 2


  - alias: Heizung Standby Timer Finished
    trigger:
      platform: event
      event_type: timer.finished
      event_data:
        entity_id: timer.heizungaufstandbyfor
    action:
    - service: input_boolean.turn_off
      data:
        entity_id: input_boolean.heizungstandbyon
```


## Links zu Solvis Doku
[Doku: Solvis Modbus Register](https://solvis-files.s3.eu-central-1.amazonaws.com/seiten/produkte/solvisremote/Download/SolvisRemote+Modbus+Spezifikationen+201906.pdf)

