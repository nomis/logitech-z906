Protocol
========

All commands are initiated by the console. All state is maintained primarily by
the amplifier so the console requires the echoed responses to update its state.

There is no way to do automatic power on without modifying the
`firmware <firmware.rst>`_. It is not otherwise possible to make the console
light up and respond to input unless it is powered on through its own power
button or remote control.

Message formats
---------------

All messages are one byte except for ``AA`` that starts a longer message.

The longer messages have the following format:

.. code-block:: none

   AA <U8 type> <U8 length of data> <variable length data> <U8 checksum>

The checksum is verified by calculating the U8 sum of all bytes from the type
(inclusive) to the checksum (inclusive) so that the final result is 0.

Communication is represented as follows:

.. code-block:: none

   > Message from console to amplifier
   < Message from amplifier to console

Status commands
---------------

Read status/configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 34
   < AA
   < 0A = type
   < 14 = length of remaining data (excluding checksum)
   < 0A = main volume (0 to 43)
   < 15 = rear level (0 to 43)
   < 15 = centre level (0 to 43)
   < 15 = subwoofer level (0 to 43)
   < 00 = input (0 to 5 => 1 to 6)
   < 00
   < 00
   < 00
   < 00 = input 2 effect (0 = 3D, 1 = 2.1, 2 = 4.1, 3 = disabled)
   < 00 = input 6 effect (0 = 3D, 1 = 2.1, 2 = 4.1, 3 = disabled)
   < 03 = input 1 effect (0 = 3D, 1 = 2.1, 2 = 4.1, 3 = disabled)
   < 00
   < 00 = unknown (00 or 0E)
   < 00 = speaker test? (00 or 33)
   < 01
   < 05
   < 04
   < 00 = standby (0 = on, 1 = standby)
   < 00
   < 00 = auto standby (0 = enabled, 1 = disabled)
   < XX = checksum

Write status/configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

Send the ``AA 0A`` message from the section above
(`Read status/configuration`_).

There's an amplifier bug in that standby status is never restored to 1, but it
can be set. Changing it from 1 to 0 will power on, changing it from 0 to 1 will
not go into standby. Try to avoid changing the power state with this command.

Reset idle time
~~~~~~~~~~~~~~~

Sent liberally by the console whenever user input is provided. Resets the idle
time maintained by the amplifier.

.. code-block:: none

   > 30
   < 30

Read idle time
~~~~~~~~~~~~~~

The idle time is not just reset by the console, it is also reset by the
amplifier itself when non-silent audio is being output. The console will send
this command every 60 seconds after the last user input.

If automatic standby is enabled and the time is reached the console will turn
the power off.

.. code-block:: none

   > 31
   < 31
   < AA
   < 0F = type
   < 03 = length of remaining data (excluding checksum)
   < 00 06 1C = idle time (U24BE in seconds)
   < XX = checksum

Read input volume
~~~~~~~~~~~~~~~~~

The console doesn't use this command, but you can find out the current volume of
the input. Silence is 0 and it looks like it goes up to 1,000,000 with some
maximum amplitude square waves. Normal values are around 1,000 to 2,000.

.. code-block:: none

   > 2F
   < 2F
   < AA
   < 08 = type
   < 03 = length of remaining data (excluding checksum)
   < 00 00 00 = volume (U24BE in unknown units)
   < XX = checksum

Power commands
--------------

Power on
~~~~~~~~

The first part of this is identical to `Headphones disconnected`_ so it should
be possible to power on with the `Headphones connected`_ sequence in its place.

The amplifier will power on with the currently configured input active, but the
effect is sent by the console automatically (`Effect selection`_).

.. code-block:: none

   > 11 11
   > XX (effect selection)
   > 39 38 30 39
   < 11 11
   < XX (effect selection)
   > 39 38 30 39

Power off
~~~~~~~~~

The ``36`` command here turns the speakers off and saves settings.

The ``37`` command's purpose is unknown.

Sends `Read status/configuration`_ at the end to update the console state.

.. code-block:: none

   > 30 37 36
   < 30 37 36
   > 34
   < AA 0A ...

Headphones connected
~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 10 10
   > 3F (effect selection)
   < 10 10
   < 3F (effect selection)

Headphones disconnected
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 11 11
   > XX (effect selection)
   < 11 11
   < XX (effect selection)

Volume/level commands
---------------------

The console implements the mute operation by setting the volume to 0 using lots
of `Main volume down`_ commands and then back up using lots of `Main volume up`_
commands. Going into standby while muted may result in a volume of 0 being
saved.

Main volume up
~~~~~~~~~~~~~~

Turning the volume up above level 43 is ignored and no command is sent.

.. code-block:: none

   > 08
   < 08

Main volume down
~~~~~~~~~~~~~~~~

Turning the volume down below level 0 is ignored and no command is sent.

.. code-block:: none

   > 09
   < 09

Mute
~~~~

Turn the volume down to level 0.

.. code-block:: none

   > 09
   > 09
   > 09 ...
   < 09
   < 09
   < 09 ...

The amplifier can also report a muted status (without changing the volume).
When the volume is next adjusted the console will unmute immediately:

.. code-block:: none

   < 20
   ...
   > 38
   > 08 or 09
   > 39

The console user interface will only report this muted status correctly (by
flashing the volume) if the amplifier has previously sent its own unmute status
while the console has entered the mute state itself at least once since the last
power on.

Unmute
~~~~~~

Turn the volume back up.

.. code-block:: none

   > 38
   > 08
   > 08
   > 08 ...
   > 39
   < 38
   < 08
   < 08
   < 08 ...
   < 39

The amplifier can also report an unmuted status (without changing the volume).
If the console had muted then the volume level will still be 0:

.. code-block:: none

   < 21

Subwoofer level up
~~~~~~~~~~~~~~~~~~

Turning the subwoofer level up above 43 is ignored and no command is sent.

.. code-block:: none

   > 0A
   < 0A

Subwoofer level down
~~~~~~~~~~~~~~~~~~~~

Turning the subwoofer level down below 0 is ignored and no command is sent.

.. code-block:: none

   > 0B
   < 0B

Centre level up
~~~~~~~~~~~~~~~

Turning the centre level up above 43 is ignored and no command is sent.

.. code-block:: none

   > 0C
   < 0C

Centre level down
~~~~~~~~~~~~~~~~~

Turning the centre level down below 0 is ignored and no command is sent.

.. code-block:: none

   > 0D
   < 0D

Rear level up
~~~~~~~~~~~~~

Turning the rear level up above 43 is ignored and no command is sent.

.. code-block:: none

   > 0E
   < 0E

Rear level down
~~~~~~~~~~~~~~~

Turning the rear level down below 0 is ignored and no command is sent.

.. code-block:: none

   > 0F
   < 0F

Input selection
---------------

Mute the volume before changing inputs (`Volume/level commands`_) and switch to
the configured effect for the input (`Effect selection`_) before unmuting.

The "decode" light turns on if there is a digital signal from inputs 3, 4 or 5.

Input 1 (3.5mm TRS, 6 channels)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 09 ... (mute)
   > 02
   > XX (effect selection)
   > 08 ... (unmute)
   < 09 ... (mute)
   < 02
   < XX (effect selection)
   < 08 ... (unmute)
   < 39

Input 2 (RCA, 2 channels)
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 09 ... (mute)
   > 05
   > XX (effect selection)
   > 08 ... (unmute)
   < 09 ... (mute)
   < 05
   < XX (effect selection)
   < 08 ... (unmute)
   < 39

Input 3 (TOSLINK, S/PDIF optical fibre)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 09 ... (mute)
   > 03
   > XX (effect selection)
   > 08 ... (unmute)
   < 09 ... (mute)
   < 03
   < XX = `Decode state`_
   < XX (effect selection)
   < 08 ... (unmute)
   < 39

Input 4 (TOSLINK, S/PDIF optical fibre)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 09 ... (mute)
   > 04
   > XX (effect selection)
   > 08 ... (unmute)
   < 09 ... (mute)
   < 04
   < XX = `Decode state`_
   < XX (effect selection)
   < 08 ... (unmute)
   < 39

Input 5 (RCA, S/PDIF 75Ω coaxial)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 09 ... (mute)
   > 06
   > XX (effect selection)
   > 08 ... (unmute)
   < 09 ... (mute)
   < 06
   < XX = `Decode state`_
   < XX (effect selection)
   < 08 ... (unmute)
   < 39


Input 6 (3.5mm TRS on the console, 2 channels)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 09 ... (mute)
   > 07
   > XX (effect selection)
   > 08 ... (unmute)
   < 09 ... (mute)
   < 07
   < XX (effect selection)
   < 08 ... (unmute)
   < 39

Decode state
~~~~~~~~~~~~

* ``17`` = 5.1 signal detected (5.1 levels, no effects)
* ``18`` = no signal
* ``19`` = 4.1 signal detected (4.1 levels, no effects)
* ``1A`` = 3.1 signal detected (5.1 levels, no effects)
* ``1B`` = 2.1 signal detected (2.1 levels, no effects)
* ``1C`` = 2.0 signal detected (2.1 levels, no effects)
* ``1D`` = signal detected (0.1 levels, no effects)
* ``1E`` = signal detected (5.1 levels, effects mandatory)
* ``1F`` = unknown signal (decode/effect LEDs cycle continuously)

Effect selection
----------------

Using effects that are not compatible with the selected input has not been
tested.

3D effect
~~~~~~~~~

.. code-block:: none

   > 14
   < 14

4.1 effect
~~~~~~~~~~

.. code-block:: none

   > 15
   < 15

2.1 effect
~~~~~~~~~~

.. code-block:: none

   > 16
   < 16

Effect disabled
~~~~~~~~~~~~~~~

.. code-block:: none

   > 35
   < 35

No effect, headphones
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > 3F
   < 3F

Speaker test
------------

While powered on hold down the "input" button for 5 seconds.

Tests speakers in this order:

* Front left
* Centre
* Front right
* Rear right
* Rear left
* Subwoofer

Start
~~~~~

Enter speaker test mode and `Select test speaker`_ "front left".

.. code-block:: none

   > 36
   > 22
   < 22
   > AA 07 ... (Select test speaker: front left)
   < 36

Select test speaker
~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   > AA
   > 07 = type
   > 08 = length of remaining data (excluding checksum)
   > 01
   > 01 = speaker (01 front left, 10 centre, 02 front right,
                   08 rear right, 04 rear left, 20 sub, 00 none)
   > 00 00 00 00 09 2B
   > XX = checksum

Stop
~~~~

Exit speaker test mode, `Select test speaker`_ "none" and restore the previously
selected input (`Input selection`_).

.. code-block:: none

   > 33
   > AA 07 ... (Select test speaker: none)
   > 00
   < 33
   < 00

Configuration reset
-------------------

While in standby hold down the "input" button for 8 seconds.

Sends `Read status/configuration`_ at the end to update the console state.

.. code-block:: none

   > AA
   > 0E = type
   > 03 = length of remaining data (excluding checksum)
   > 20 00 00
   > CF = checksum
   > AA
   > 0A = type
   > 14 = length of remaining data (excluding checksum)
   > 0A 15 15 15 00 00 00 00 00 00 03 00 00 00 06 01 03 00 00 00
   > 8C = checksum
   > 36
   < AA
   < FF = type
   < 01 = length of remaining data (excluding checksum)
   < 8A
   < 76 = checksum
   < 36
   > 34
   < AA 0A 14 0A 15 15 15 00 00 00 00 00 00 03 00 00 00 01 05 04 00 00 00 8C

Enable/disable automatic standby
--------------------------------

While powered on hold the "level" button for 5 seconds (until the level change
light goes out). The setting will be toggled.

.. code-block:: none

   > AA
   > 0E = type
   > 03 = length of remaining data (excluding checksum)
   > 20 00 00
   > CF = checksum
   > AA
   > 0A = type
   > 14 = length of remaining data (excluding checksum)
   > 0A 15 15 15 00 00 00 00 00 00 03 00 00 00 01 05 04 00 00
   > XX = auto standby (0 = enabled, 1 = disabled)
   > XX = checksum
   > 36
   < AA
   < FF = type
   < 01 = length of remaining data (excluding checksum)
   < 8A
   < 76 = checksum
   < 36
