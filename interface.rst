Interface
=========

The control console and the amplifier are connected by a cable that uses a
`DE-15 <https://en.wikipedia.org/wiki/D-subminiature>`_ connector. This can be
extended with a `VGA <https://en.wikipedia.org/wiki/VGA_connector>`_ extension
cable (but the screws to hold the connectors together will be mismatched).

When a VGA cable is used, pin 9 may not be present.

If not using the original console, avoid connecting anything to the first 5
pins unless those audio connections are required because it's possible to
introduce noise with them.

Pin out
-------

+-----+-----------------------------+-------------------------------------------------+
| Pin | Console                     | Amplifier                                       |
+=====+=============================+=================================================+
|   1 |                             | Headphones Right                                |
+-----+-----------------------------+-------------------------------------------------+
|   2 |                             | Headphones Left                                 |
+-----+-----------------------------+-------------------------------------------------+
|   3 | Audio Ground                | Audio Ground                                    |
+-----+-----------------------------+-------------------------------------------------+
|   4 | Aux Right                   |                                                 |
+-----+-----------------------------+-------------------------------------------------+
|   5 | Aux Left                    |                                                 |
+-----+-----------------------------+-------------------------------------------------+
|   6 | Ground                      | Ground                                          |
+-----+-----------------------------+-------------------------------------------------+
|   7 | Unknown (not required)      | Unknown (not required)                          |
+-----+-----------------------------+-------------------------------------------------+
|   8 | | Pull-down (0V) resistor   | | Output 3.3V for 500ms at power up/comms start |
|     | | Amplifier presence        | | Output 3.3V for 100ms after comms stop        |
+-----+-----------------------------+-------------------------------------------------+
|   9 | Output 3.3V when powered    |                                                 |
|     | (not required)              | Unused                                          |
+-----+-----------------------------+-------------------------------------------------+
|  10 | Unknown (not required)      | Unused                                          |
+-----+-----------------------------+-------------------------------------------------+
|  11 | Power supply                | 3.3V                                            |
+-----+-----------------------------+-------------------------------------------------+
|  12 | Rx                          | Tx                                              |
+-----+-----------------------------+-------------------------------------------------+
|  13 | Tx                          | Rx                                              |
+-----+-----------------------------+-------------------------------------------------+
|  14 | Unused                      | Unused                                          |
+-----+-----------------------------+-------------------------------------------------+
|  15 | | Output 0V if comms active | | Pull-up (3.3V) resistor                       |
|     | | to turn on amplifier      | | Console presence                              |
+-----+-----------------------------+-------------------------------------------------+

Pin 7 has some bidirectional signalling with intervals of no less than 100ms
related to changes in the values of pins 8 and 15. If it's connected then the
duration of amplifier pin 8 changes to 1s at power up/comms start and 4s after
comms stop.

Serial communication
--------------------

The Rx/Tx pins are 3.3V TTL serial at a baud rate of 57600 bps with 8 bits per
byte, odd parity and 1 stop bit. All communication is initiated by the console.

The amplifier will not turn on its power to the speakers until the console is
present (pin 15). It will turn off its power immediately if there is no console
present. The power supply for the console is always active. The amplifier should
be powered off using pin 15 when not actively in use otherwise it will waste
power and heat up.

The console will get upset if the amplifier doesn't respond to communication
(this is the cause of the 2, 3 or 4 error indication displayed on the console
input selection LEDs).

* 2 LEDs: amplifier presence remains 3.3V high
* 3 LEDs: no response to serial communication

The `protocol <protocol.rst>`_ is a simple binary interface.
