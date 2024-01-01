Digo Control via BLE
====================

.. seo::
    :description: Instructions for setting up Improv via BLE in ESPHome with Digo additional setting.
    :image: improv-social.png

This component was customized by DigoTech to control some other components via BLE. 
User could control device directly by bluetooth on mobilephone.

The ``esp32_digo_ble`` component need set up the :doc:`BLE Server <esp32_ble>`.

.. warning::

    The BLE software stack on the ESP32 consumes a significant amount of RAM on the device.
    
    **Crashes are likely to occur** if you include too many additional components in your device's
    configuration. Memory-intensive components such as :doc:`/components/voice_assistant` and other
    audio components are most likely to cause issues.

    **Currently, we only support control switch component. Other component will be supported soon.**

.. code-block:: yaml

    # Example configuration entry
    # Make sure enable BLE server
    esp32_ble:
      enable_on_boot: true 

    # Add automation disable BLE when connect Wifi success and vice versa.
    wifi:
      on_connect:
        - delay: 2s
        - logger.log: "Wifi connect then disable BLE."
        - ble.disable:
      on_disconnect:
        - delay: 5s
        - logger.log: "Wifi disconnect then enable BLE."
        - ble.enable:

    # Declare switch components
    switch:
      - platform: gpio
        pin: 25
        id: relay1
      - platform: gpio
        pin: 26
        id: relay2

    # Add Digo BLE to control switch component via BLE.
    digo_ble:
      switch_id: 
        - relay1
        - relay2


Configuration variables:
------------------------

- **switch_id** (*Optional*, list): List of switch to control.

GATT Services
-------------

Service UUID: ``00467768-6228-2272-4663-277478266000``
    
Characteristic: RPC Command
***************************

Characteristic UUID: ``00467768-6228-2272-4663-277478266001``

This characteristic is where the client can write data to control component through the RPC service.

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Byte
     - Description
   * - 1
     - Switch index
   * - 2
     - Switch value

Example: Set Switch index 0 to 1:
``00 01``

Example: Set Switch index 1 to 0:
``01 00``