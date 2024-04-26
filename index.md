---
layout: index
title: Index
---
# Commodore 128 Memory map and Reference

## Index

### Memory map
* [Startup sequence](StartupSequence) (wip - need more information)
* [Memory map and layout](MemoryMap)

### Instruction set
* [Instruction set details](InstructionSet)
<!--* [Opcode layout](OpcodeLayout)-->

### Components
* [CIA (Complex Interface Adapter)](Cia)
* [MMU (Memory Management Unit)](Mmu)
* [SID (Sound Interface Device)](Sid)
* [Screen editor ROM](ScreenEditorRom)
* [VDC (Video Display Controller)](Vdc)
* [VIC (Video Interface Controller)](Vic)

### Memory reference
* [$0000-$00FF - Zero Page](0000)
* [$0200-$02FF - Input Buffer, Common indirect routines, Indirect Vectors](0200)
* [$0300-$03FF - Basic/Kernal indirect vectors, Screen editor tables, Kernal file tables, Basic working storage](0300)
* [$0A00-$0AFF - Kernal and Screen Editor Working Storage, Monitor Working Storage Area, Kernal Working Storage](0A00)
* [$0B00-$0BFF - Cassette Buffer and Disk Boot Buffer](0B00)
* [$0E00-$0FFF - Sprite Pattern Storage Area](0E00)
* [$1000-$10FF - Programmable Key Definition String Area](1000)
* [$1100-$11FF - BASIC Working Storage](1100)
* [$1200-$12FF - BASIC General-Purpose Working Storage](1200)
* [$4000-$AEFF - BASIC Entry Vectors](4000)
* [$AF00-$AFFF - BASIC Jump Table](AF00)
* [$B000-$BFFF - Monitor Jump Table](B000)
* [$C000-$CFFF - Screen editor ROM](C000)
* [$D000-$D030 - VIC Chip Registers](D000)
* [$D400-$D41C - SID Chip Registers](D400)
* [$D500-$D50B - Mmu Chip Registers](D500)
* [$D600-$D601 - VDC Chip Registers](D600)
* [$DC00-$DC0F - CIA 1 (Complex Interface Adapter)](DC00)
* [$DD00-$DD0F - CIA 2 (Complex Interface Adapter)](DD00)
* [$DF00-$DFFF - RAM Expansion Controller Chip Registers](DF00) [#5](https://github.com/c128lib/Reference/issues/5)
* [$E000-$FFFF - Kernal Rom, Standard Commodore Jump Table](E000)

### Commodore related resources
This is a list of resources that you use to create the reference.
They are not limited to the C128 but could also deal with other computers,
in particular the C64 with which a good part of the architecture is shared.

#### Books & magazines

* [Mapping the Commodore 128](https://www.cubic.org/~doj/c64/mapping128.pdf)

#### Books & magazines known

* [Books](Books)

#### Links

* [Bart's place](https://www.bartsplace.net/topics/cbm.shtml)
Tons of informations, projects and resources for C128

* [Sven's Techsite](http://tech.guitarsite.de/c64_scope.html)
Interesting information about C64 chips timings

* [The memory accesses of the MOS 6569 VIC-II and MOS 8566 VIC-IIe](https://ist.uwaterloo.ca/~schepers/MJK/ascii/vic2-pal.txt)
An article from Marko Mäkelä about Vic2 (and Vic2e) memory access

* [The MOS 6567/6569 video controller (VIC-II) and its application in the Commodore 64](https://ist.uwaterloo.ca/~schepers/MJK/ascii/VIC-Article.txt)
An article from Christian Bauer about Vic2

