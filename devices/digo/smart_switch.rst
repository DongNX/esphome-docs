Digo Smart Switch
================

.. seo::
    :description: Instructions for setting up a Digo Smart Switch.
    :image: digo.svg

The ``digo_smart_switch`` overview introduction. 
`........ Coming soon .........`
Use this component to integrate Digo Smart Switch into ESPHome / Home Assistant ecosystem.

.. figure:: images/coming_soon.png
    :align: center
    :width: 100.0%

    Smart Switch front and back view. Image by `DIGO <https://digotech.net/solution>`__.

The ``digo_smart_switch`` hardware introduction. Touch, Relay, Led7seg, ntc, ...

.. figure:: images/coming_soon.png
    :align: center
    :width: 100.0%

    Photo of something, images by `DIGO <https://digotech.net/solution>`__.
.. figure:: images/coming_soon.png
    :align: center
    :width: 100.0%

    Photo of serial port pins, images by `DIGO <https://digotech.net/solution>`__.

Before using this components make sure:

- board is configured to ``nodemcu-32s``
- :ref:`UART bus <uart>` is configured with default RX / TX pins and 115200 baud rate
- :doc:`logger </components/logger>` to the serial port is disabled by setting ``baud_rate`` to ``0``

.. code-block:: yaml

    # Example configuration entry
    esphome:
    name: smartswitch

    # Need to include dogo custom components
    <<: !include digo_components.yaml


    # Comming soon

Configuration variables:
------------------------
`........ Coming soon .........`


See Also
--------
`........ Coming soon .........`