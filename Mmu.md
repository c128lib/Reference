---
layout: default
title: SID (Sound Interface Device)
---
# MMU (Memory Management Unit)

The MMU memory management chip, officially designated the
8722, is the cornerstone of the 128 system. In fact, the MMU
is what makes most of the 128's special features possible. The
MMU was designed by Commodore's engineers specifically to
support the 128's multiple operating modes and elaborate
memory banking scheme. It is the MMU which determines
which microprocessor, 8502 or Z80, has control of the computer. When the 8502 is in control, the MMU determines
whether the computer operates in 128 mode or 64 mode. The 128 hardware includes many times
more elements than can simultaneously fit in the 64K address
space of the 8502 or Z80 microprocessors. The MMU chip also
determines which memory resources are visible to the
processsor at any given time. 

Most BASIC programming can be done without understanding the inner
workings of the MMU, simply by using the
available standard bank configurations. However, a thorough
knowledge of the MMU is essential for taking full advantage
of all the 128's features. 

The 17 registers of the MMU are unusual in that they are
divided into two separate groups at different memory locations. The first 12 appear in the I/O block at 54528-54539/$D500-$D50B, while the other 5 appear at 65280-65284/$FF00-$FF04.
This second set of MMU registers is special in
that it appears in all memory configurations. There is no way
to make the processor see anything other than the MMU registers at those locations while the 128 is in 128 mode. Next table
lists the MMU registers. A detailed description of each register
follows.

|Address|Function|
|-|-|
|54528/$D500|Configuration register|
|54529/$D501|Preconfiguration register A|
|54530/$D502|Preconfiguration register B|
|54531/$D503|Preconfiguration register C|
|54532/$D504|Preconfiguration register D|
|54533/$D505|Mode configuration register|
|54534/$D506|RAM configuration register|
|54535/$D507|Page 0 page pointer|
|54536/$D508|Page 0 block pointer|
|54537/$D509|Page 1 page pointer|
|54538/$D50A|Page 1 block pointer|
|54539/$D50B|MMU version register|
|65280/$FF00|Configuration register|
|65281/$FF01|Load configuration register A|
|65282/$FF02|Load configuration register B|
|65283/$FF03|Load configuration register C|
|65284/$FF04|Load configuration register D|

## See also

* [$D500-$D50B - Mmu Chip Registers](D500)
* [$FF00-$FF04 - Mmu Chip Registers](E000#FF00)
