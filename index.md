---
layout: default
title: Index
---
# Commodore 128 Memory map and Reference

## Index

### Memory map
* [Memory map and layout](MemoryMap)

### Components
* [CIA (Complex Interface Adapter)](Cia)
* [MMU (Memory Management Unit)](Mmu)
* [SID (Sound Interface Device)](Sid)
* [Screen editor ROM](ScreenEditorRom)
* [VDC (Video Display Controller)](Vdc)
* [VIC (Video Interface Controller)](Vic)

### Memory reference
* [$0000-$00FF - Zero Page](0000)
* [$117E-$11D5 - Sprite movement control data](117E)
* [$11D6-$11EA - Shadow registers](11D6)
* [$C000-$CFFF - Screen editor ROM](C000)
* [$D000-$D030 - VIC Chip Registers](D000)
* [$D400-$D41C - SID Chip Registers](D400)
* [$D500-$D50B - Mmu Chip Registers](D500)
* [$D600-$D601 - VDC Chip Registers](D600)
* [$DC00-$DC0F - CIA 1 (Complex Interface Adapter)](DC00)
* [$DD00-$DD0F - CIA 2 (Complex Interface Adapter)](DD00)
* [$FF00-$FF04 - Mmu Chip Registers](FF00)
* [$FF47-$FFF3 - Kernal](FF47)

### Commodore related resources
This is a list of resources that you use to create the reference.
They are not limited to the C128 but could also deal with other computers,
in particular the C64 with which a good part of the architecture is shared.

#### Books & magazines

* [Mapping the Commodore 128](https://www.cubic.org/~doj/c64/mapping128.pdf)

#### Links

* [Bart's place](https://www.bartsplace.net/topics/cbm.shtml)
Tons of informations, projects and resources for C128

* [Sven's Techsite](http://tech.guitarsite.de/c64_scope.html)
Interesting information about C64 chips timings

* [The memory accesses of the MOS 6569 VIC-II and MOS 8566 VIC-IIe](https://ist.uwaterloo.ca/~schepers/MJK/ascii/vic2-pal.txt)
An article from Marko Mäkelä about Vic2 (and Vic2e) memory access

* [The MOS 6567/6569 video controller (VIC-II) and its application in the Commodore 64](https://ist.uwaterloo.ca/~schepers/MJK/ascii/VIC-Article.txt)
An article from Christian Bauer about Vic2

