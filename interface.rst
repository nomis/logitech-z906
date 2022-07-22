Interface
=========

The control console and the amplifier are connected by a cable that uses a
`DE-15 <https://en.wikipedia.org/wiki/D-subminiature>`_ connector. This can be
extended with a `VGA <https://en.wikipedia.org/wiki/VGA_connector>`_ extension
cable (but the screws to hold the connectors together will be mismatched).

Pin out
-------

+-----+-----------------------------------+-----------------------------------+
| Pin | Console                           | Amplifier                         |
+=====+===================================+===================================+
|   1 |                                   | Headphones Right                  |
+-----+-----------------------------------+-----------------------------------+
|   2 |                                   | Headphones Left                   |
+-----+-----------------------------------+-----------------------------------+
|   3 | Audio Ground                      | Audio Ground                      |
+-----+-----------------------------------+-----------------------------------+
|   4 | Aux Right                         |                                   |
+-----+-----------------------------------+-----------------------------------+
|   5 | Aux Left                          |                                   |
+-----+-----------------------------------+-----------------------------------+
|   6 | Ground                            | Ground                            |
+-----+-----------------------------------+-----------------------------------+
|   7 | Unused                            | Unused                            |
+-----+-----------------------------------+-----------------------------------+
|   8 | | Pull-down (0V) resistor         | Output 3.3V for 500ms until power |
|     | | Amplifier presence              | up complete                       |
+-----+-----------------------------------+-----------------------------------+
|   9 | Unused                            | Unused                            |
+-----+-----------------------------------+-----------------------------------+
|  10 | Unused                            | Unused                            |
+-----+-----------------------------------+-----------------------------------+
|  11 | Power supply                      | 3.3V                              |
+-----+-----------------------------------+-----------------------------------+
|  12 | Rx                                | Tx                                |
+-----+-----------------------------------+-----------------------------------+
|  13 | Tx                                | Rx                                |
+-----+-----------------------------------+-----------------------------------+
|  14 | Unused                            | Unused                            |
+-----+-----------------------------------+-----------------------------------+
|  15 | | Output 0V if comms active       | | Pull-up (3.3V) resistor         |
|     | |                                 | | Console presence                |
+-----+-----------------------------------+-----------------------------------+

Serial communication
--------------------

The Rx/Tx pins are 3.3V TTL serial at a baud rate of 57600 bps with 8 bits per
byte, odd parity and 1 stop bit. All communication is initiated by the console.

The amplifier will not turn on its power until the console is present. It will
turn off its power (speakers and serial communication) immediately if there is
no console present. The power supply for the console is always active.

The console will get upset if the amplifier is not present (this is the cause of
the 2, 3 or 4 error LEDs displayed).

The `protocol <protocol.rst>`_ is a simple binary interface.
