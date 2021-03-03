Interface
=========

The control console and the amplifier are connected by a cable that uses a
`DE-15 <https://en.wikipedia.org/wiki/D-subminiature>`_ connector. This can be
extended with a `VGA <https://en.wikipedia.org/wiki/VGA_connector>`_ extension
cable (but the screws to hold the connectors together will be mismatched).

Pin out
-------

   +-----+-------------------------------------------+-------------------------+
   | Pin | Console                                   | Amplifier               |
   +=====+===========================================+=========================+
   |   1 | 100kΩ to ground when disconnected         | Headphones Right        |
   +-----+-------------------------------------------+-------------------------+
   |   2 | 100kΩ to ground when disconnected         | Headphones Left         |
   +-----+-------------------------------------------+-------------------------+
   |   3 | Unknown (Aux ground?)                     |                         |
   +-----+-------------------------------------------+-------------------------+
   |   4 | Aux Right                                 |                         |
   +-----+-------------------------------------------+-------------------------+
   |   5 | Aux Left                                  |                         |
   +-----+-------------------------------------------+-------------------------+
   |   6 | Ground                                    | Ground                  |
   +-----+-------------------------------------------+-------------------------+
   |   7 | Unused                                    | Unused                  |
   +-----+-------------------------------------------+-------------------------+
   |   8 | Amplifier presence                        | Unknown signalling      |
   +-----+-------------------------------------------+-------------------------+
   |   9 | Unused                                    | Unused                  |
   +-----+-------------------------------------------+-------------------------+
   |  10 | Unused                                    | Unused                  |
   +-----+-------------------------------------------+-------------------------+
   |  11 | Power supply                              | 3.3V                    |
   +-----+-------------------------------------------+-------------------------+
   |  12 | Rx                                        | Tx                      |
   +-----+-------------------------------------------+-------------------------+
   |  13 | Tx                                        | Rx                      |
   +-----+-------------------------------------------+-------------------------+
   |  14 | Unused                                    | Unused                  |
   +-----+-------------------------------------------+-------------------------+
   |  15 | 0V if present                             | Console presence        |
   +-----+-------------------------------------------+-------------------------+

Serial communication
--------------------

The Rx/Tx pins are 3.3V TTL serial at a baud rate of 57600 bps with 8 bits per
byte, odd parity and 1 stop bit. All communication is initiated by the console.

The amplifier will turn off the speakers if there is no console present.

The console will get upset if the amplifier is not present (this is the cause of
the 2, 3 or 4 error LEDs displayed).

The `protocol <protocol.rst>`_ is a simple binary interface.
