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

    # Add Digo BLE to control switch component via BLE.
    digo_ble:


Configuration variables:
------------------------

None

GATT Services
-------------

Service UUID: ``F61E3BE9-2826-A81B-970A-4D4DECFABBAE``
    
Characteristic: RPC Command
***************************

Characteristic UUID: ``6490FAFE-0734-732C-8705-91B653A081FC``

This characteristic is where the client can write data to control component through the RPC service. Developer feed json data directly into payload.

Example:
``
{"ts":682820481,"tc":"abcdefghiklmn","values":{"switch_1":true,"cur_cur":1242,"cur_vol":220,"cur_pwr":23}}
``


