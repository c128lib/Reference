---
layout: default
title: CIA (Complex Interface Adapter)
---
# CIA (Complex Interface Adapter)

The CIA (complex interface adapter) chips perform the majority of the
128's input and output functions. Between them, the
CIAs are responsible for handling communications with the
keyboard, joysticks, the serial bus (where disk drives and
printers are connected), the RS-232 port (where modems are
connected), and the user port. In fact-with the exception of
video output provided by the [Vic](Vic) and [Vdc](Vdc) chips and the audio output
provided by the [Sid](Sid) chip-the list of I/O functions
performed by devices other than the CIAs is quite short: the
[Vic](Vic) and [Vdc](Vdc) chips handle light pen input for their respective
displays, the [Sid](Sid) chip reads paddle controllers (although a
CIA reads paddle buttons and selects which pair of paddles is
to be read), the processor's on-chip I/O port is used to control
some aspects of tape data storage and to read the CAPS LOCK
key, and an MMU register line is used to read the 40/80 DISPLAY key.

All CIA registers are set to zero when the system RESET
line is pulled low, as when the reset button is pushed. Most of
the CIA registers are initialized during the IOINIT routine
[$E109](E000#E109).

## CIA 1 Registers
This CIA is used to read the keyboard, joysticks, and other devices
connected to the control ports, such as the 128 mouse. It
also selects which pair of paddles will be read. The timers and
FLAG input line are used in reading from and writing to tape.
The chip's serial data communications hardware is used for
fast serial bus I/O.

## CIA 2 Registers
This CIA is used to support the serial bus, and to provide the
RS-232 interface. It also provides programmable I/O lines at
the user port for custom interfacing projects. Another vital
function of this unit is to select which area of memory is used
aS the current VIC video bank.

## See also

* [$DC00-$DC0F - CIA 1 (Complex Interface Adapter)](DC00)
* [$DD00-$DD0F - CIA 2 (Complex Interface Adapter)](DD00)
