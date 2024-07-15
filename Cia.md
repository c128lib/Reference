---
layout: default
title: CIA (Complex Interface Adapter)
---
# CIA (Complex Interface Adapter)

The CIA (complex interface adapter) chips perform the majority of the
128's input and output functions.
The 128 (like the 64) has two CIA chips which have separated responsibilities.
They are responsible for handling communications with the
keyboard, joysticks, the serial bus (where disk drives and
printers are connected), the RS-232 port (where modems are
connected), and the user port.

All CIA registers are set to zero when the system RESET
line is pulled low, as when the reset button is pushed. Most of
the CIA registers are initialized during the IOINIT routine
[$E109](E000#E109).

## Integrated circuits
The CIA has been produced in several integrated chips including:
* 6526, with a frequency of 1 MHz;
* 6526A, identical to the previous one but capable of working at 2 MHz;
* 8520, CIA produced with HMOS-III technology intended for the Commodore Amiga computer. Compared to the 6526 it had an internal clock based on a 24-bit binary counter;
* 5710, a shortened version of the CIA (only 4 internal registers) produced for the 1571CR disk drive.

## CIA 1 Registers ($DC00)
This CIA is used to read the keyboard, joysticks, and other devices
connected to the control ports, such as the 128 mouse. It
also selects which pair of paddles will be read. The timers and
FLAG input line are used in reading from and writing to tape.
The chip's serial data communications hardware is used for
fast serial bus I/O.

## CIA 2 Registers ($DD00)
This CIA is used to support the serial bus, and to provide the
RS-232 interface. It also provides programmable I/O lines at
the user port for custom interfacing projects. This unit also
permits to select which area of memory is used
as the current VIC video bank.

## See also

* [$DC00-$DC0F - CIA 1 (Complex Interface Adapter)](DC00)
* [$DD00-$DD0F - CIA 2 (Complex Interface Adapter)](DD00)
