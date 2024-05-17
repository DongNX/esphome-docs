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

Passkey
-------------
The ``passkey`` is 6-digits number which will be used when Mobile App want to connect to the device. The ``passkey`` is sum of bytes in token.
For example, token is ``abc`` in string where ``a``, ``b``, ``c`` ascii code is 97, 98 and 99 then passkey = 97 + 98 +99 = 000294

GATT Services
-------------

Service UUID: ``F61E3BE9-2826-A81B-970A-4D4DECFABBAE``
    
Characteristic: RPC Command
***************************

Characteristic UUID: ``6490FAFE-0734-732C-8705-91B653A081FC``

This characteristic is where the client can write data to control component through the RPC service. Developer feed json data directly into payload.

Example:

Command:
``
{"method":"command","values":{"cmd_1":3,"position_1":0}}
``

Response:
``
{"method":"status","success":true,"values":{"cmd_1":3,"position_1":0}}
``


