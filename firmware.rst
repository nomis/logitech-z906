Firmware
========

Control Console
---------------

The microcontroller in the control console is an STM8S103K3. The programming
header is at the bottom of the PCB to the right of the microcontroller.

.. figure:: images/console-pcb.jpg
   :height: 275px
   :alt: Console PCB with "PROGRAMMING" header at the bottom (2 rows of 3 pins)

   Programming header pins

   +---------------------------+--------------------+--------------------+
   | ○ No connection           | ○ SWIM (pin 26)    | ○ NRST (pin 1)     |
   +---------------------------+--------------------+--------------------+
   | ■ |Vdd| (pin 6) 3.3V      | ○ GND (pin 4)      | ○ GND (pin 4)      |
   +---------------------------+--------------------+--------------------+
.. |Vdd| replace:: V\ :sub:`DD`

The firmware can be accessed using an ST-LINK implementation,
e.g. `esp-stlink <https://github.com/rumpeltux/esp-stlink>`_
and `stm8flash <https://github.com/vdudouyt/stm8flash>`_.

The flash consists of 64 byte pages. Writes must be to whole pages.

The EEPROM consists of 640 bytes that are all zero.

Attempts were made to use an official ST-LINK/V2 but it was unable to
communicate.

.. list-table::
   :header-rows: 1

   * - Version
     - Size
     - SHA-3-256 hash
   * - S-00103 (2020)
     - 8KB
     - ``a16ecffacc67c9005814028080c90d098f10708b88b285fdbeef480358c37cb8``

Useful tools
~~~~~~~~~~~~

* `Disassembler (naken_asm) <https://github.com/mikeakohn/naken_asm>`_
* `Decompiler (Ghidra) <https://ghidra-sre.org/>`_
  with `STM8 plugin <https://github.com/esaulenka/ghidra_STM8>`_

Modifications
~~~~~~~~~~~~~

Idle time for automatic standby
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Read 64 bytes of flash at ``0x80C0`` and modify the seconds time value
(U32BE, default 7198) at ``0x80DE``:

.. code-block:: none

   80C0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80D0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx 00 00
   80E0  1C 1E xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80F0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx

Write 64 bytes of flash at ``0x80C0`` (2 hours):

.. code-block:: none

   80C0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80D0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx 00 00
   80E0  1C 20 xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80F0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx

Write 64 bytes of flash at ``0x80C0`` (24 hours):

.. code-block:: none

   80C0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80D0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx 00 01
   80E0  51 80 xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80F0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx

Write 64 bytes of flash at ``0x80C0`` (7 days):

.. code-block:: none

   80C0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80D0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx 00 09
   80E0  3A 80 xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80F0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx

Write 64 bytes of flash at ``0x80C0`` (no auto off):

.. code-block:: none

   80C0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80D0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx 01 00
   80E0  00 00 xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   80F0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx

This value is limited to 24 bits by the protocol, so setting any bits in the top
byte will cause the idle time to never be exceeded.

Automatic power on
^^^^^^^^^^^^^^^^^^
This code is part of the reset interrupt for the microcontroller so it will not
be executed after a short power cycle while the speakers are turned off.

When turning the speakers on/off by switching the power supply on/off, be aware
that the settings (current volume, etc.) are only saved when going into standby.

There is a `protocol <protocol.rst>`_ side effect of enabling this in that the
console will send ``F8`` or ``FC`` instead of ``34`` the first time it goes into
standby. This appears to have no impact to the operation of the speakers.


Read 64 bytes of flash at ``0x87C0`` and modify the instruction at ``0x87F2``:

.. code-block:: none

   87C0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   87D0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   87E0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx CD 89
   87F0  BD 9A 4F CD 85 D1 xx xx  xx xx xx xx xx xx xx xx
               ^^

Write 64 bytes of flash at ``0x87C0`` (auto on):

.. code-block:: none

   87C0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   87D0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   87E0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx CD 89
   87F0  BD 9A 9D CD 85 D1 xx xx  xx xx xx xx xx xx xx xx
               ^^

Write 64 bytes of flash at ``0x87C0`` (power up in standby):

.. code-block:: none

   87C0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   87D0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx xx xx
   87E0  xx xx xx xx xx xx xx xx  xx xx xx xx xx xx CD 89
   87F0  BD 9A 4F CD 85 D1 xx xx  xx xx xx xx xx xx xx xx
               ^^

Explanation:
  The original instruction clears register A which is the parameter to the
  function at ``0x85D1``. By not clearing it, the previous function that was
  called (``0x89BD``) has left A with a non-zero value.

  Function ``0x85D1`` stores the value at memory ``0x01BB``, which is then used
  in the first status request to decide whether to run the power on or power off
  function.
