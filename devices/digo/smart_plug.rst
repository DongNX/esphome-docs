Using With Digo Smart Plug
============================

The Digo Smart Switch is based on the ``ESP32C3`` platform and is a subtype of the ``ESP32C3`` module.
You can just take the below configuration file and modify it to your needs.

.. figure:: ../images/sonoff_s20_header.jpg
    :align: center
    :width: 75.0%

    Digo Smart Plug.

Creating Firmware
-----------------

.. code-block:: yaml
    esphome:
        name: ct2wf
        platform: ESP32
        board: nodemcu-32s

    wifi:

    logger:

    ota:

    switch:
        - platform: gpio
            pin: GPIO2
            id: relay

    binary_sensor:
        - platform: gpio
            pin: GPIO3
            id: button
            on_press:
            then:
                - switch.toggle: relay

    external_components:
        - source:
            type: local
            path: /path/to/digo_component

    digo:
        model: "PLWF"
        hardware: 101
        firmware: 102