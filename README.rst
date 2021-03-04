Logitech® Z906 Surround Sound Speakers
======================================

Overview
--------

5.1 surround sound speaker system with the amplifier built into the subwoofer
and managed by an external control console attached using a
`DE-15 <https://en.wikipedia.org/wiki/D-subminiature>`_ connector.

Purpose
-------

Provide unofficial documentation of the serial protocol and possible
firmware/configuration modifications.

The console `firmware <firmware.rst>`_ can be modified to:

* Automatically turn the speakers on when power is supplied.
* Change the automatic standby idle timeout from the default 2 hours.

Automatic standby
~~~~~~~~~~~~~~~~~

Hold the "level" button for 5 seconds (until the level change light goes out).
There's no obvious indication that anything has changed but the current effect
will have been changed to the default for all inputs (check by turning the
console off and on).

Use the reset procedure to get this setting into a known state (automatic
standby enabled).

Contents
--------

* `Firmware <firmware.rst>`_
* `Interface <interface.rst>`_
* `Protocol <protocol.rst>`_

"Logitech" is a trademark of `Logitech International SA <https://www.logitech.com/>`_.
