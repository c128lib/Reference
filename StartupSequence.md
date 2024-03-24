---
layout: default
title: Startup sequence
---

When you first turn on power, the Z80 microprocessor has control before the 8502 is
allowed to take over. 
The initialization steps performed by the Z80 include copying two routines into block
0 RAM.
One, at 65488-65503/$FFD0-$FFDF, is an 8502 machine language routine to surrender
control to the Z80; the other, at 65504-65519/$FFE0-$FFEF, is a Z80 machine language
routine to surrender control to the 8502.

If the Z80 reset routine does
not find a CP/M boot disk in the drive (or a Commodore 64
cartridge or the Commodore key held down), it jumps to a
routine it has copied into 65504/$FFE0 in block 0 RAM. That
routine ends by setting bit 0 of $D505 to %1 to return control to the
8502 for 128 mode. 

TODO: where is code for this and other operations before 8502 wakes up

When the 8502 finally starts after a reset, the Kernal initialization routine
START always receives control and immediately performs the following actions:
1. Brings the system map into context.
2. Disables IRQ's.
3. Resets the processor stack pointer.
4. Clears decimal mode.
5. Initializes the MMU.
6. INSTALLS the Kernal RAM code.
7. SECURE: Check and initialize the SYSTEM vector.
8. [POLL](#POLL): Check for a ROM cartridge. 

## POLL <a name="POLL"></a>
POLL first scans for any installed C64 cartridges. They are recognized by
either the GAME or EXROM signal being pulled low. If so, GO64 is executed
(see the Kernal jump entry for details). Polling for C64 cartridges is actually
redundant at this point since the Z80 processor, which powers up initially, did this
already. POLL then scans for installed C128 cartridges and function ROM's. They
are recognized by the existence of the C= key in any of the four function ROM
slots (two internal, two external) and are polled in the order external low (16 or
32KB), external high (16KB), internal low (16 or 32KB), internal high (16KB).
The entire format is:

<pre>
$x000 -> cold start entry
$x003 -> warm start entry (unused)
$x006 -> ID, ($01-$FF)
$x007 -> "CBM" key string
where x = $8--- or $C---.
</pre>

The ID of any C128 cartridge found is entered into the Physical Address Table
(PAT) located at $0AC1-$0AC4. ID's must be non-zero. Cartridges may recognize
each other by examining the PAT for particular ID's. An ID of 1 indicates an
auto-start cartridge, and its cold start entry will be called immediately. All others
will be called later (see PHOENIX jump), as will any auto-starters that RTS is to
POLL. A cartridge can determine where it is installed by examining CURBNK,
located at $0AC0. Because it is possible for a cartridge to be called with interrupts
enabled, the following diversion from the above format is recommended (the warm
start entry is never called by the system):

<pre>
$x000 SEI
$x001 JMP STARTUP
$x004 NOP
$x005 NOP
</pre>

The balance of the C128 initialization is:

9. [IOINIT](#IOINIT): Initialize I/O devices.
10. Check for STOP and C= keys.
11. [RAMTAS](#RAMTAS): Initialize system RAM.
12. [RESTOR](#RESTOR): Initialize system indirects.
13. [CINT](#CINT): Initialize video displays.
14. Enable IRQ's (except foreign systems).
15. Dispatch. 

### IOINIT
IOINIT is perhaps the major function of the Reset handler. It initializes both CIA's
(timers, keyboard, serial port, user port, cassette) and the 8502 port (keyboard,
cassette, VIC bank). It distinguishes a PAL system from an NTSC one and sets
PALCNT ($A03) if PAL. The VIC, SID and 8563 devices are initialized, including
the downloading of character definitions to 8563 display RAM (if necessary). The
system 60Hz IRQ source (the VIC raster) is started. IOINIT is callable by the user
via the jump table.

During initialization, the user may press certain keys as a means of selecting
an operating mode. One key checked is the Commodore key C=, indicating C64
mode is desired. While this key was scanned much earlier by the Z80 to speed
the switchover to C64 mode, there is a redundant check for it here. The only other
key scanned at this time is the STOP key, which signals a request by the user
to power up into the Monitor utility. Note that control does not pass from the
initialization process at this point; the Kernal needs to know if RAMTAS should be
skipped. Only if the STOP key is depressed and this was a "warm" reset
(vs. "cold" power-up) can RAMTAS be skipped. 

### RAMTAS
RAMTAS clears all page-zero RAM, allocates the cassette and RS-232
buffers, sets pointers to the top and bottom of system RAM (RAM-0), and
installs the [SYSTEM_VECTOR](0A00#0A00) that points to BASIC cold
start ($4000). Lastly it sets a flag, [DEJAVU](0A00#0A02) ($A02), to indicate
to other routines
that system RAM has been initialized. This is the difference between a "cold"
and a "warm" system. If DEJAVU contains $A5, the system is "warm" and
[SYSTEM_VECTOR](0A00#0A00) is valid. Many programmers debugging code need to recover
from a system hang or crash via RESET but do not want RAM cleared. This
is why the STOP key is scanned, RAMTAS is skipped, and the Monitor
(rather than BASIC) is selected. RAMTAS is callable by the user via the jump
table.

### RESTOR
RESTOR initializes the Kernal indirect vectors. This must be done before
many system routines will function. Applications that complement the operating
system via "wedges" must install them after they are initialized. RESTOR is user
callable from the jump table (see also the VECTOR call).

### CINT
CINT is the editor's initialization routine. Both 40- and 80-column display
modes are prepared, editor indirect vectors installed, programmable key definitions
assigned, and the 40/80 key scnaned for default display determination. CINT is also
a jump table entry.

Finally, the IRQ's are enabled and control is passed to either BASIC
initialization, GO64 code, or the ML Monitor. BASIC will call the Kernal
PHOENIX routine upon the conclusion of its initialization, which will call any
installed C128 cartridges (any ID) and attempt to auto-boot an application from
disk. 

## Kernal debugging

TODO: what happens before this?

<pre>
E263: A9 F7     LDA #$F7
E265: 8D 05 D5  STA $D505
E268: 6C FC FF  JMP ($FFFC) // FFFC contains FF3D

FF3D: A9 00     LDA #$00
FF3F: 8D 00 FF  STA $FF00   // Set bank 15
FF42: 4C 00 E0  JMP $E000   // Jump to RESET

E000: A2 FF     LDX #$FF
E002: 78        SEI         // Disable interrupt
E003: 9A        TXS         // Set stack pointer to $ff (stack cleared)
E004: D8        CLD         // Disable decimal mode
E005: A9 00     LDA #$00
E007: 8D 00 FF  STA $FF00   // Set bank 15
E00A: A2 0A     LDX #$0A    
E00C: BD 4B E0  LDA $E04B,X 
E00F: 9D 00 D5  STA $D500,X // Reset MMU status
E012: CA        DEX
E013: 10 F7     BPL $E00C
E015: 8D 04 0A  STA $0A04   // Set INIT_STATUS flag to 0
E018: 20 CD E0  JSR $E0CD   // Move Code For High RAM Banks
E01B: 20 F0 E1  JSR $E1F0   // Check Special Reset
E01E: 20 42 E2  JSR $E242   // Detect C64 cartridge inserted, if so switch to C64
E021: 20 09 E1  JSR $E109   // IOINIT
E024: 20 3D F6  JSR $F63D   // Watch For RUN or Shift
E027: 48        PHA         // Push .A for further use
E028: 30 07     BMI $E031   // RUN/STOP not pressed, jump to RAMTAS
E02A: A9 A5     LDA #$A5
E02C: CD 02 0A  CMP $0A02   // Check if RAMTAS should be omitted
E02F: F0 03     BEQ $E034

E031: 20 93 E0  JSR $E093   // RAMTAS
E034: 20 56 E0  JSR $E056   // RESTOR
E037: 20 00 C0  JSR $C000   // CINT
E03A: 68        PLA
E03B: 58        CLI         // Enable interrupt
E03C: 30 03     BMI $E041   // If RUN/STOP is pressed, skip to non
E03E: 4C 00 B0  JMP $B000   // JMONINIT

E041: C9 DF     CMP #$DF    // Check if Commodore key is pressed
E043: F0 03     BEQ $E048
E045: 6C 00 0A  JMP ($0A00) // Jump to SYSTEM_VECTOR for a Basic Restart

E048: 4C 4B E2  JMP $E24B   // Switch to C64 mode

4000: 4C 23 40  JMP $4023   

4023: 20 7A 41  JSR $417A   // Set Preconfig Registers
4026: 20 51 42  JSR $4251   // Set Basic Links
4029: 20 45 40  JSR $4045   // Set-Up Basic Constants
402C: 20 9B 41  JSR $419B   // Print Startup Message
402F: AD 04 0A  LDA $0A04 
4032: 09 01     ORA #$01
4034: 8D 04 0A  STA $0A04   // Set bit 0 of INIT_STATUS flag to 1
4037: A2 03     LDX #$03
4039: 8E 00 0A  STX $0A00   // Restart System (BASIC Warm) [4000]
403C: A2 FB     LDX #$FB
403E: 9A        TXS
403F: 20 56 FF  JSR $FF56   // PHOENIX 
4042: 4C 1C 40  JMP $401C

401C: 58        CLI
401D: 4C 37 4D	JMP $4D37	  // Error or Ready

4D37: A2 80     LDX #$80
4D39: 2C        .BYTE $2C   // Means BIT $10A2
4D3A: A2 10     LDX #$10
4D3C: 6C 00 03  JMP ($0300) // 0300 contains $4D3F

4D3F: 8D 03 FF  STA $FF03
4D42: 8A        TXA         // .X contains $80
4D43: 30 72     BMI $4DB7

4DB7: 20 2A 4D  JSR $4D2A   // Print 'ready'
4DBA: A9 80     LDA #$80
4DBC: 20 90 FF	JSR $FF90   // Enables Control messages, disables Error messages
4DBF: A9 00     LDA #$00
4DC1: 85 7F     STA $7F     // Set BASIC Immediate mode

4DC3: 6C 02 03	JMP ($0302) // BASIC Warm Start Link (0302 contains $4DC6)

</pre>
