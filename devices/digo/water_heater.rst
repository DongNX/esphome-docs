Digo Water Heater
=================

.. seo::
    :description: Instructions for setting up a Digo Water Heater.
    :image: water_heater.jpg

The ``digo_water_heater`` device is developed by DIGO with some kind of functions as below:

- Connect and control by DIGO app (available on `IOS <https://apps.apple.com/in/app/digo-smart/id1577695678>`__ and `Android <https://play.google.com/store/apps/datasafety?id=com.dolphin.digosmart&hl=vi&gl=US&pli=1>`__).
- Turn on / turn off and synchronize water temperature with DIGO app.
- Heat water follow set temperatrue.
- Heat water follow set schedule.

Use this component to integrate Digo Water Heater into ESPHome / Home Assistant ecosystem.

.. figure:: images/water_heater.jpg
    :align: center
    :width: 100.0%

    Water heater front and back view. Image by `DIGO <https://digotech.net/solution>`__.

The ``digo_water_heater`` hardware introduction. Touch, Relay, Led7seg, ntc, ...

.. figure:: images/water_heater.jpg
    :align: center
    :width: 100.0%

    Photo of something, images by `DIGO <https://digotech.net/solution>`__.
.. figure:: images/water_heater.jpg
    :align: center
    :width: 100.0%

    Photo of serial port pins, images by `DIGO <https://digotech.net/solution>`__.

Before using this components make sure:

- board is configured to ``nodemcu-32s``
- framework type is configured to ``esp-idf``
- framework version is configured to ``5.1.1``
- framework platform_version is configured to ``6.4.0``
- make sure you include ``digo_component`` to your yaml file.

This is example source for waterheaterwifi.yaml file:

.. code-block:: yaml

    substitutions:
    devicename: waterheaterwifi
    hardware_ver: '101'
    firmware_ver: '100'

    <<: !include digo_component.yaml

    esphome:
    name: waterheaterwifi
    on_boot:
        priority: 600
        then:
        - switch.turn_on: relay1
        - rtttl.play: 'Looney:d=4,o=5,b=140:32p,c6,8f6,8e6,8d6,8c6,a.,8c6,8f6,8e6,8d6,8d#6,e.6,8e6,8e6,8c6,8d6,8c6,8e6,8c6,8d6,8a,8c6,8g,8a#,8a,8f'

    esp32:
    board: nodemcu-32s
    framework:
        type: esp-idf
        version: 5.1.1
        platform_version: 6.4.0
    
    globals:
    - id: touch_hold_time
    type: int
    initial_value: "0"

    interval:
    - interval: 100ms
    then:
        - lambda: |-
            if (id(touch_pad).state){
            id(touch_hold_time) = (id(touch_hold_time) + 1);
            }
            else {
            id(touch_hold_time) = 0;
            }

    switch:
    - platform: digo_relay
        name: "Relay"
        id: relay1
        relay_pin: 21
        zero_detect_pin: 35
        on_turn_on:
        then:
            - rtttl.play: 'short:d=4,o=5,b=100:16e6'
        on_turn_off:
        then:
            - rtttl.play: 'short:d=4,o=5,b=100:16e6'

    sensor:
    - platform: wifi_signal # Reports the WiFi signal strength/RSSI in dB
        name: "WiFi Signal dB"
        id: wifi_signal_db
        update_interval: 30s
        entity_category: "diagnostic"

    - platform: copy # Reports the WiFi signal strength in %
        source_id: wifi_signal_db
        name: "WiFi Signal Percent"
        filters:
        - lambda: return min(max(2 * (x + 100.0), 0.0), 100.0);
        unit_of_measurement: " %"
        entity_category: "diagnostic"
        device_class: ""

    - platform: debug
        free:
        name: "Heap Free"

    # Declare water temperature sensor
    - platform: ntc
        sensor: water_temperature_sensor
        calibration:
        b_constant: 3950
        reference_temperature: 25°C
        reference_resistance: 10kOhm
        name: Water Temperature
        id: water_temperature
    # Declare board temperature sensor
    - platform: ntc
        sensor: board_temperature_sensor
        calibration:
        b_constant: 3950
        reference_temperature: 25°C
        reference_resistance: 10kOhm
        name: Board Temperature

    # Configuration for water temperature sensor
    - platform: resistance
        internal: true
        id: water_temperature_sensor
        sensor: water_temperature_resistance_sensor
        configuration: DOWNSTREAM
        resistor: 33kOhm
        name: Water Temperature Resistance Sensor
    - platform: adc
        id: water_temperature_resistance_sensor
        pin: A6
        update_interval: 30s
    # Configuration for board temperature sensor
    - platform: resistance
        internal: true
        id: board_temperature_sensor
        sensor: board_temperature_resistance_sensor
        configuration: DOWNSTREAM
        resistor: 33kOhm
        name: Board Temperature Resistance Sensor
    - platform: adc
        id: board_temperature_resistance_sensor
        pin: A5
        update_interval: 30s

    display:
    - platform: digo_led7seg
        hc595_latch_pin: 26
        hc595_clock_pin: 27
        hc595_data_pin: 25
        digit_1_ctrl_pin: 18
        digit_2_ctrl_pin: 19
        lambda: |-
        if (id(touch_hold_time)) {
            it.setNumber((uint8_t)(id(touch_hold_time) / 10));
        }
        else {
            it.setNumber((uint8_t)id(water_temperature).get_state());
        } 

    output:
    - platform: ledc
        pin: GPIO32
        id: rtttl_out

    rtttl:
    output: rtttl_out

    button:
    - platform: restart
        name: "Restart"
        id: restart_bt

    - platform: factory_reset
        name: Restart with Factory Default Settings
        id: factory_reset_bt

    - platform: template
        name: "Play music"
        on_press:
        then:
            - rtttl.play: 'MissionImp:d=16,o=6,b=95:32d,32d#,32d,32d#,32d,32d#,32d,32d#,32d,32d,32d#,32e,32f,32f#,32g,g,8p,g,8p,a#,p,c7,p,g,8p,g,8p,f,p,f#,p,g,8p,g,8p,a#,p,c7,p,g,8p,g,8p,f,p,f#,p,a#,g,2d,32p,a#,g,2c#,32p,a#,g,2c,a#5,8c,2p,32p,a#5,g5,2f#,32p,a#5,g5,2f,32p,a#5,g5,2e,d#,8d'

    binary_sensor:
    - platform: digo_touch
        name: "Touch Pad"
        pin: GPIO4
        id: touch_pad
        
        on_click:
        - min_length: 3000ms
        max_length: 5000ms
        then:
            - button.press: restart_bt
        - min_length: 5000ms
        max_length: 10000ms
        then:
            - if:
                condition: ble.enabled
                then:
                - ble.disable:
                else:
                - ble.enable:
            # - button.press: factory_reset_bt

    climate:
    - platform: bang_bang
        id: climate_1
        name: "Water Heater Controller"
        sensor: water_temperature
        default_target_temperature_low: 70 °C
        default_target_temperature_high: 75 °C

        visual:
        min_temperature: 20
        max_temperature: 100
        temperature_step: 1

        heat_action:
        - switch.turn_on: relay1
        idle_action:
        - switch.turn_off: relay1


Configuration variables:
------------------------

- All other options for switch component come from :ref:`Switch <config-switch>`.
- All other options for button component come from :ref:`Button <config-button>`.
- All other options for binary_sensor component come from :ref:`Binary Sensor <config-binary_sensor>`.
- All other options for sensor component come from :ref:`Sensor <config-sensor>`.
- All other options for climate component come from :ref:`Light <config-climate>`.


See Also
--------