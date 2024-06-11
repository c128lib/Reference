---
layout: default
title: Startup sequence
---

# Startup sequence

## Operations summary

At startup or reset, the Z80 microprocessor has control and it starts executing
code from address $0000.

## Kernal debugging

Z80 is in control now and starts by putting $0000 in program counter.

``` Assembly
0000: 3e 3e       LD      a,3Eh
0002: 32 00 ff    LD      (0FF00h),a    // Set MMU to 3E
0005: c3 3b 00    JP      003Bh

003B: 01 2f d0    LD      bc,0D02Fh
003E: 11 fc ff    LD      de,0FFFCh     // Load Reset vector to de
0041: ed 51       OUT     (c),d
0043: 03          INC     bc
0044: ed 59       OUT     (c),e         // Value of reset vector is written to port c
0046: 01 05 d5    LD      bc,0D505h
0049: 3e b0       LD      a,0B0h
004B: ed 79       OUT     (c),a         // Set MMUMCR to $B0
004D: ed 78       IN      a,(c)
004F: 2f          CPL
0050: e6 30       AND     30h
0052: 28 05       JR      z,0059h

0054: 3e f1       LD      a,0F1h
0056: ed 79       OUT     (c),a
0058: c7          RST     00h
0059: 01 0f dc    LD      bc,0DC0Fh
005C: 3e 08       LD      a,08h
005E: ed 79       OUT     (c),a
0060: 0d          DEC     c             // Set D1CRA and D1CRB timer to
0061: ed 79       OUT     (c),a         // one-shot mode
0063: 0e 03       LD      c,03h         // Set bc to DC03
0065: af          XOR     a
0066: ed 79       OUT     (c),a
0068: 0d          DEC     c             // Set bc to DC02
0069: 3d          DEC     a
006A: ed 79       OUT     (c),a
006C: 0d          DEC     c             // Set bc to DC01
006D: 0d          DEC     c             // Set bc to DC00
006E: 3e 7f       LD      a,7Fh
0070: ed 79       OUT     (c),a
0072: 03          INC     bc
0073: ed 78       IN      a,(c)
0075: e6 20       AND     20h
0077: 01 05 d5    LD      bc,0D505h     // Check for Commodore key pressed
007A: 28 d8       JR      z,0054h

007C: 21 b4 0f    LD      hl,0FB4h      // Copies from $0FB4 (-11)
007F: 01 0a d5    LD      bc,0D50Ah     // to $D50A (-11)
0082: 16 0b       LD      d,0Bh         // $0B (11) bytes
0084: 7e          LD      a,(hl)        // It's a decrement copy
0085: ed 79       OUT     (c),a         // for Mmu init
0087: 2b          DEC     hl
0088: 0d          DEC     c
0089: 15          DEC     d
008A: 20 f8       JR      nz,0084h

008C: 21 1a 0d    LD      hl,0D1Ah      // Copies from $0D1A
008F: 11 00 11    LD      de,1100h      // to $1100
0092: 01 08 00    LD      bc,0008h      // $08 bytes
0095: ed b0       LDIR
0097: 21 e5 0e    LD      hl,0EE5h      // Copies from $0EE5
009A: 11 d0 ff    LD      de,0FFD0h     // to $FFD0
009D: 01 1f 00    LD      bc,001Fh      // $1F (31) bytes
00A0: ed b0       LDIR                  // These are the copy of z80 and 8502
                                        // surrender routine from kernal to memory

00A2: 21 00 11    LD      hl,1100h
00A5: 22 fa ff    LD      (0FFFAh),hl
00A8: 22 fc ff    LD      (0FFFCh),hl
00AB: 22 fe ff    LD      (0FFFEh),hl
00AE: 22 dd ff    LD      (0FFDDh),hl
00B1: c3 e0 ff    JP      0FFE0h        // Jump to z80 surrender to 8502 cpu

// Data values used in routine at 008C
0D1A: a9 00       LDA #$00
0D1C: 8d 00 ff    STA $FF00
0D1F: 6c fc ff    JMP ($FFFC)

// Data values used in routine at 0097 (8502 and z80 surrender routines)
0EE5: 78          SEI
0EE6: a9 b0       LDA #$B0
0EE8: 8d 05 d5    STA $D505
0EEB: ea          NOP
0EEC: 4c 00 30    JMP $3000
0EEF: ea          NOP
0EF0: f3          DI
0EF1: 3e 3e       LD  a,3Eh
0EF3: 32 00 ff    LD  (0FF00h),a
0EF6: 01 05 d5    LD  bc,0D505h
0EF9: 3e b1       LD  a,0B1h
0EFB: ed 79       OUT (c),a
0EFD: 00          NOP
0EFE: cf          RST 08h

// Data values used in routine at 007c (MMU init values)
0FAA: 3f // $D500
0FAB: 3f // $D501
0FAC: 7f // $D502
0FAD: 3e // $D503
0FAE: 7e // $D504
0FAF: b0 // $D505
0FB0: 0b // $D506
0FB1: 00 // $D507
0FB2: 00 // $D508
0FB3: 01 // $D509
0FB4: 00 // $D50A
```

When execution reach $0EFB, 8502 gains control and starts for
the first time. Its program counter is set to
<pre>
.PC = $FFFD * 256 + $FFFC
</pre>

When running the Z80 at address $00a2, the $fffc vector is assigned the value $1100.
Since the 8502 processor begins its execution by inserting the value contained in $fffc into the program counter, execution continues from $1100, set bank 15
(activating Kernal rom) and then proceed to RESET routine.

``` Assembly
1100: A9 F7     LDA #$00
1102: 8D 05 D5  STA $FF00   // Sets bank 15 so Kernal is visible
1105: 6C FC FF  JMP ($FFFC) // FFFC contains FF3D

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
401D: 4C 37 4D	JMP $4D37   // Error or Ready

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
```

### RAMTAS

RAMTAS is a routine at $E093:
* RAMTAS clears all page-zero RAM
* allocates the cassette and RS-232 buffers
* sets pointers to the top and bottom of system RAM (RAM-0)
* installs the [SYSTEM_VECTOR](0A00#0A00) that points to BASIC cold
start ([$4000](4000#4000))
* sets a flag, [DEJAVU](0A00#0A02) ($A02), to indicate to other routines
that system RAM has been initialized.

The DEJAVU is the difference between a "cold" and a "warm" system.
If DEJAVU contains $A5, the system is "warm" and
[SYSTEM_VECTOR](0A00#0A00) is valid.

### RESTOR

RESTOR is a routine at $E056 that initializes the Kernal indirect vectors.
This must be done before
many system routines will function. Applications that complement the operating
system via "wedges" must install them after they are initialized. RESTOR is user
callable from the jump table.

### CINT

CINT is a routine at $C000. It's the editor's initialization routine.
It prepares 40 and 80 column display, set editor indirect vectors, assign programmable keys.

The IRQ's are enabled and control is passed to either BASIC
initialization, GO64 code, or the ML Monitor. BASIC will call the Kernal
PHOENIX routine upon the conclusion of its initialization, which will call any
installed C128 cartridges (any ID) and attempt to auto-boot an application from
disk.

## References

* Mapping the Commodore 128 - Ottis Cowper
* Commodore 128 ROM Disassembly https://klasek.at/c64/c128-rom-listing.html
