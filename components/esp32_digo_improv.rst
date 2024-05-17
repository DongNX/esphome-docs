Digo Improv via BLE
==============

.. seo::
    :description: Instructions for setting up Improv via BLE in ESPHome with Digo additional setting.
    :image: improv-social.png

This component was customized by DigoTech. Basically, this component still base on ``esp32_improv`` component.
Some additional RPC commands were added to make device could setup to connect to DigoTech cloud.

The ``esp32_improv`` component in ESPHome implements the open `Improv standard <https://www.improv-wifi.com/>`__
for configuring Wi-Fi on an ESP32 device by using Bluetooth Low Energy (BLE) to receive the credentials.

The ``esp32_improv`` component will automatically set up the :doc:`BLE Server <esp32_ble>`.

.. warning::

    The BLE software stack on the ESP32 consumes a significant amount of RAM on the device.
    
    **Crashes are likely to occur** if you include too many additional components in your device's
    configuration. Memory-intensive components such as :doc:`/components/voice_assistant` and other
    audio components are most likely to cause issues.

.. code-block:: yaml

    # Example configuration entry
    wifi:
      # ...

    esp32_improv:
      authorizer: binary_sensor_id


Configuration variables:
------------------------

- **authorizer** (**Required**, :ref:`config-id`): A :doc:`binary sensor <binary_sensor/index>` to authorize with.
  Also accepts ``none``/``false`` to skip authorization.
- **authorized_duration** (*Optional*, :ref:`config-time`): The amount of time until authorization times out and needs
  to be re-authorized. Defaults to ``1min``.
- **status_indicator** (*Optional*, :ref:`config-id`): An :doc:`output <output/index>` to display feedback to the user.
- **identify_duration** (*Optional*, :ref:`config-time`): The amount of time to identify for. Defaults to ``10s``.
- **wifi_timeout** (*Optional*, :ref:`config-time`): The amount of time to wait before starting the improv service after Wi-Fi
  is no longer connected. Defaults to ``1min``.

Status Indicator
----------------

The ``status_indicator`` has the following patterns:

- solid: The improv service is active and waiting to be authorized.
- blinking once per second: The improv service is awaiting credentials.
- blinking 3 times per second with a break in between: The identify command has been used by the client.
- blinking 5 times per second: Credentials are being verified and saved to the device.
- off: The improv service is not running.

GATT Services
-------------

Service UUID: ``00467768-6228-2272-4663-277478268000``

Characteristic: Capabilities
****************************

Characteristic UUID: ``00467768-6228-2272-4663-277478268005``

This characteristic has binary encoded byte(s) of the device’s capabilities.

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Bit (LSB)
     - Capability
   * - 0
     - 1 if the device supports the identify command.

Characteristic: Current State
*****************************

Characteristic UUID: ``00467768-6228-2272-4663-277478268001``

This characteristic will hold the current status of the provisioning service and will write and notify any listening clients for instant feedback.

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Value
     - State
     - Purpose
   * - 0x01
     - Authorization Required	
     - Awaiting authorization via physical interaction.
   * - 0x02
     - Authorized
     - Ready to accept credentials.
   * - 0x03
     - Provisioning
     - Credentials received, attempt to connect.
   * - 0x04
     - Provisioned
     - Connection successful.

Characteristic: Error state
***************************

Characteristic UUID: ``00467768-6228-2272-4663-277478268002``

This characteristic will hold the current error of the provisioning service and will write and notify any listening clients for instant feedback.

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Value
     - State
     - Purpose
   * - 0x00
     - No error	
     - This shows there is no current error state.
   * - 0x01
     - Invalid RPC packet
     - RPC packet was malformed/invalid.
   * - 0x02
     - Unknown RPC command
     - The command sent is unknown.
   * - 0x03
     - Unable to connect
     - The credentials have been received and an attempt to connect to the network has failed.
   * - 0x04
     - Not Authorized
     - Credentials were sent via RPC but the Improv service is not authorized.
   * - 0xFF
     - Unknown Error
     - 
    
Characteristic: RPC Command
***************************

Characteristic UUID: ``00467768-6228-2272-4663-277478268003``

This characteristic is where the client can write data to call the RPC service.

Note: if the combined payload is over 20 bytes, it will require multiple BLE packets to transfer the data. Make sure that your code deals with this.

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Byte
     - Description
   * - 1
     - Command (see below)
   * - 2
     - Data length
   * - 3...X
     - Data
   * - X + 1
     - Checksum - A simple sum checksum keeping only the LSB

RPC Command: Send Wi-Fi settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Submit Wi-Fi credentials to the Improv Service to attempt to connect to.

Requires the Improv service to be authorized.

Command ID: ``0x01``

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Byte
     - Description
   * - 1
     - Command
   * - 2
     - Data length
   * - 3
     - SSID length
   * - 4...X
     - SSID bytes
   * - X + 1
     - Password length
   * - X + 2 ... Y
     - Password bytes
   * - Y + 1
     - Checksum - A simple sum checksum keeping only the LSB

Example: SSID = MyWirelessAP, Password = mysecurepassword
``01 1E 0C {MyWirelessAP} 10 {mysecurepassword} CS``

RPC Command: Identify
^^^^^^^^^^^^^^^^^^^^^
What a device actually does when an identify command is received is up to that specific device, but the user should be able to visually or audibly identify the device.

Command ID: ``0x02``

Does not require the Improv service to be authorized.

Should only be sent if the capability characteristic indicates that identify is supported.

Payload: ``02 00 CS``

DIGO RPC command: Set Host settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Command ID: ``0xC8``

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Byte
     - Description
   * - 1
     - Command
   * - 2
     - Data length
   * - 3
     - Host length
   * - 4...X
     - Host bytes
   * - X + 1
     - Port length
   * - X + 2 ... Y
     - Port bytes
   * - Y + 1
     - Host_http length
   * - Y + 2 ... Z
     - Host_http bytes
   * - Z + 1
     - Port_http length
   * - Z + 2 ... G
     - Port_http bytes
   * - G + 1
     - SSL length
   * - H + 2 ... I
     - SSL bytes
   * - I + 1
     - Checksum - A simple sum checksum keeping only the LSB

DIGO RPC command: Set Device settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Command ID: ``0xC9``

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Byte
     - Description
   * - 1
     - Command
   * - 2
     - Data length
   * - 3
     - Device_profile_id length
   * - 4...X
     - Device_profile_id bytes
   * - X + 1
     - Device_id length
   * - X + 2 ... Y
     - Device_id bytes
   * - Y + 1
     - Box_id length
   * - Y + 2 ... Z
     - Box_id bytes
   * - Z + 1
     - Tenant_id length
   * - Z + 2 ... G
     - Tenant_id bytes
   * - G + 1
     - Device_owner length
   * - G + 2 ... I
     - Device_owner bytes
   * - I + 1
     - Checksum - A simple sum checksum keeping only the LSB

DIGO RPC command: Set Token settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Command ID: ``0xCA``

.. list-table:: 
   :widths: auto
   :header-rows: 1

   * - Byte
     - Description
   * - 1
     - Command
   * - 2
     - Data length
   * - 3
     - Token length
   * - 4...X
     - Token bytes
   * - X + 1
     - Checksum - A simple sum checksum keeping only the LSB
