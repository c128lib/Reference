---
layout: default
title: Instruction set
---

# Opcodes list

<a name=ADC></a>

## ADC (ADd with Carry)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|ADC #$BE|$69|2|2|
|Zero Page|ADC $BE|$65|2|3|
|Zero Page,X|ADC $BE,X|$75|2|4|
|Absolute|ADC $BEEF|$6D|3|4|
|Absolute,X|ADC $BEEF,X|$7D|3|4+|
|Absolute,Y|ADC $BEEF,Y|$79|3|4+|
|Indirect,X|ADC ($BE,X)|$61|2|6|
|Indirect,Y|ADC ($BE),Y|$71|2|5+|

\+ add 1 cycle if page boundary crossed

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|✓|-|-|-|✓|✓|

* N set if bit 7 of the result is set
* V set if sign bit is incorrect
* Z set if .A is 0
* C set if overflow in bit 7

ADC results are dependant on the setting of the <a href="#DFLAG">decimal flag</a>.
In decimal mode, addition is carried out on the assumption that the values
involved are packed BCD (Binary Coded Decimal).

<a name=AND></a>

## AND (bitwise AND with accumulator)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|AND #$BE|$29|2|2|
|Zero Page|AND $BE|$25|2|3|
|Zero Page,X|AND $BE,X|$35|2|4|
|Absolute|AND $BEEF|$2D|3|4|
|Absolute,X|AND $BEEF,X|$3D|3|4+|
|Absolute,Y|AND $BEEF,Y|$39|3|4+|
|Indirect,X|AND ($BE,X)|$21|2|6|
|Indirect,Y|AND ($BE),Y|$31|2|5+|

\+ add 1 cycle if page boundary crossed

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|-|

* N set if bit 7 set
* Z set if .A is 0

A logical AND is performed, bit by bit, on the accumulator contents
using the contents of a byte of memory.

<a name=ASL></a>

## ASL (Arithmetic Shift Left)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Accumulator|ASL A|$0A|1|2|
|Zero Page|ASL $BE|$06|2|5|
|Zero Page,X|ASL $BE,X|$16|2|6|
|Absolute|ASL $BEEF|$0E|3|6|
|Absolute,X|ASL $BEEF,X|$1E|3|7|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|✓|

* N set if bit 7 of the result is set
* Z set if .A is 0
* C set to contents of old bit 7

ASL shifts all bits left one position. 0 is shifted into bit 0 and the
original bit 7 is shifted into the Carry. The effect of this operation is
to multiply the memory contents by 2 (ignoring 2's complement considerations),
setting the carry if the result will not fit in 8 bits.

<a name=BIT></a>

## BIT (test BITs)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|BIT $BE|$24|2|3|
|Absolute|BIT $BEEF|$2C|3|4|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|✓|-|-|-|✓|-|

* N set to bit 7 of the memory value
* V set to contents of old bit 7
* Z set if .A is 0

BIT sets the Z flag as though the value in the address tested were ANDed
with the accumulator. The N and V flags are set to match bits 7 and 6
respectively in the value stored at the tested address.

BIT is often used to skip one or two following bytes as in:

<PRE>
CLOSE1 LDX #$10     If entered here, we
       .BYTE $2C    effectively perform
CLOSE2 LDX #$20     a BIT test on $20A2,
       .BYTE $2C    another one on $30A2,
CLOSE3 LDX #$30     and end up with the X
CLOSEX LDA #12      register still at $10
       STA ICCOM,X  upon arrival here.
</pre>

Beware: a BIT instruction used in this way as a NOP does have effects: the flags
may be modified, and the read of the absolute address, if it happens to access an
I/O device, may cause an unwanted action.

<a name=BCC></a> <a name=BCS></a> <a name=BEQ></a> <a name=BNE></a>
<a name=BMI></a> <a name=BPL></a> <a name=BVC></a> <a name=BVS></a>
<a name=BRA></a>

## Branch Instructions
*Affects flags: none*

All branches are relative mode and have a length of two bytes. Syntax is "Bxx
Displacement" or (better) "Bxx Label". See the notes on the <A
href="#PC">Program Counter</a> for more on
displacements.

Branches are dependant on the status of the flag bits when the opcode is
encountered. A branch not taken requires two machine cycles. Add one if the
branch is taken and add one more if the branch crosses a page boundary.

|Mnemonic|Hex|
|-|-|
|BPL (Branch on PLus)|$10|
|BMI (Branch on MInus)|$30|
|BVC (Branch on oVerflow Clear)|$50|
|BVS (Branch on oVerflow Set)|$70|
|BCC (Branch on Carry Clear)|$90|
|BCS (Branch on Carry Set)|$B0|
|BNE (Branch on Not Equal)|$D0|
|BEQ (Branch on EQual)|$F0|

There is no BRA (BRanch Always) instruction but it can be easily emulated
by branching on the basis of a known condition. One of the best flags to use for
this purpose is the <a href="#VFLAG">oVerflow</a> which is unchanged
by all but addition and subtraction operations.

A page boundary crossing occurs when the branch destination is on a different
page than the instruction AFTER the branch instruction. For example:
<pre>
  SEC
  BCS LABEL
  NOP
</pre>

A page boundary crossing occurs (i.e. the BCS takes 4 cycles) when (the
address of) LABEL and the NOP are on different pages. This means that
<pre>
        CLV
        BVC LABEL
  LABEL NOP
</pre>

the BVC instruction will take 3 cycles no matter what address it is located
at.

<a name=BRK></a>

## BRK (BReaK)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Implied|BRK|$00|1|7|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|-|-|✓|-|-|-|-|

* B set to 1

BRK causes a non-maskable interrupt and increments the program counter by
one. Therefore an <a href="#RTI">RTI</a> will
go to the address of the BRK +2 so that BRK may be used to replace a
two-byte instruction for debugging and the subsequent RTI will be correct.

<a name=CMP></a>

## CMP (CoMPare accumulator)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|CMP #$BE|$C9|2|2|
|Zero Page|CMP $BE|$C5|2|3|
|Zero Page,X|CMP $BE,X|$D5|2|4|
|Absolute|CMP $BEEF|$CD|3|4|
|Absolute,X|CMP $BEEF,X|$DD|3|4+|
|Absolute,Y|CMP $BEEF,Y|$D9|3|4+|
|Indirect,X|CMP ($BE,X)|$C1|2|6|
|Indirect,Y|CMP ($BE),Y|$D1|2|5+|

\+ add 1 cycle if page boundary crossed

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|✓|

* N set if bit 7 of the result is set
* Z set if .A = memory
* C set if .A >= memory

Compare sets flags as if a subtraction had been carried out. If the value
in the accumulator is equal or greater than the compared value, the Carry will
be set. The equal (Z) and negative (N) flags will be set based on equality or lack
thereof and the sign (i.e. A&gt;=$80) of the accumulator.

<a name=CPX></a>

## CPX (ComPare X register)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|CPX #$BE|$E0|2|2|
|Zero Page|CPX $BE|$E4|2|3|
|Absolute|CPX $BEEF|$EC|3|4|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|✓|

* N set if bit 7 of the result is set
* Z set if .X = memory
* C set if .X >= memory

Operation and flag results are identical to equivalent mode accumulator
<a href="#CMP">CMP</a> ops.

<a name=CPY></a>

## CPY (ComPare Y register)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|CPY #$BE|$C0|2|2|
|Zero Page|CPY $BE|$C4|2|3|
|Absolute|CPY $BEEF|$CC|3|4|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|✓|

* N set if bit 7 of the result is set
* Z set if .Y = memory
* C set if .Y >= memory

Operation and flag results are identical to equivalent mode accumulator
<a href="#CMP">CMP</a> ops.

<a name=DEC></a>

## DEC (DECrement memory)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|DEC $BE|$C6|2|5|
|Zero Page,X|DEC $BE,X|$D6|2|6|
|Absolute|DEC $BEEF|$CE|3|6|
|Absolute,X|DEC $BEEF,X|$DE|3|7|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|-|

* N set if bit 7 of the result is set
* Z set if result is zero

Subtracts one from the value held at a specified memory location
setting the zero and negative flags as appropriate.

<a name=EOR></a>

## EOR (bitwise Exclusive OR)
*Affects flags: N Z*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|EOR #$BE|$49|2|2|
|Zero Page|EOR $BE|$45|2|3|
|Zero Page,X|EOR $BE,X|$55|2|4|
|Absolute|EOR $BEEF|$4D|3|4|
|Absolute,X|EOR $BEEF,X|$5D|3|4+|
|Absolute,Y|EOR $BEEF,Y|$59|3|4+|
|Indirect,X|EOR ($BE,X)|$41|2|6|
|Indirect,Y|EOR ($BE),Y|$51|2|5+|

\+ add 1 cycle if page boundary crossed

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|-|

* N set if bit 7 is set
* Z set if .A is zero

An exclusive OR is performed, bit by bit, on the accumulator
contents using the contents of a byte of memory.

<a name=CLC></a> <a name=SEC></a> <a name=CLD></a> <a name=SED></a>
<a name=CLI></a> <a name=SEI></a> <a name=CLV></a>

## Flag (Processor Status) Instructions
Affect Flags: as noted

These instructions are implied mode, have a length of one byte and require
two machine cycles.

|Mnemonic|Hex|
|-|-|
|CLC (CLear Carry)|$18|
|SEC (SEt Carry)|$38|
|CLI (CLear Interrupt)|$58|
|SEI (SEt Interrupt)|$78|
|CLV (CLear oVerflow)|$B8|
|CLD (CLear Decimal)|$D8|
|SED (SEt Decimal)|$F8|

Notes:
<a name=IFLAG></a> The Interrupt flag is used to prevent (SEI) or
enable (CLI) maskable interrupts (aka IRQ's). It does not signal the presence or
absence of an interrupt condition. The 6502 will set this flag automatically in
response to an interrupt and restore it to its prior status on completion of the
interrupt service routine. If you want your interrupt service routine to permit
other maskable interrupts, you must clear the I flag in your code.

<a name=DFLAG></a> The Decimal flag controls how the 6502 adds and
subtracts. If set, arithmetic is carried out in packed binary coded decimal.
This flag is unchanged by interrupts and is unknown on power-up. The implication
is that a CLD should be included in boot or interrupt coding.

<a name=VFLAG></a> The Overflow flag is generally misunderstood and
therefore under-utilised. After an ADC or SBC instruction, the overflow flag
will be set if the twos complement result is less than -128 or greater than
+127, and it will cleared otherwise. In twos complement, $80 through $FF
represents -128 through -1, and $00 through $7F represents 0 through +127.
Thus, after:

<pre>
  CLC
  LDA #$7F ;   +127
  ADC #$01 ; +   +1
</pre>

the overflow flag is 1 (+127 + +1 = +128), and after:
<pre>
  CLC
  LDA #$81 ;   -127
  ADC #$FF ; +   -1
</pre>

the overflow flag is 0 (-127 + -1 = -128). The overflow flag is not
affected by increments, decrements, shifts and logical operations i.e. only
ADC, BIT, CLV, PLP, RTI and SBC affect it. There is no opcode to set the
overflow but a BIT test on an RTS instruction will do the trick.

<a name=INC></a>

## INC (INCrement memory)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|INC $BE|$E6|2|5|
|Zero Page,X|INC $BE,X|$F6|2|6|
|Absolute|INC $BEEF|$EE|3|6|
|Absolute,X|INC $BEEF,X|$FE|3|7|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|-|

* N set if bit 7 is set
* Z set if result is zero

Adds one to the value held at a specified memory location
setting the zero and negative flags as appropriate.

<a name=JMP></a>

## JMP (JuMP)
|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Absolute|JMP $5597|$4C|3|3|
|Indirect|JMP ($5597)|$6C|3|5|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|-|-|-|-|-|-|-|

JMP transfers program execution to the following address (absolute) or to
the location contained in the following address (indirect). Note that there is
no carry associated with the indirect jump so:

<span class="badge badge-info">An indirect jump must never use a vector beginning on the last byte of a page</span>

For example if address $3000 contains $40, $30FF contains $80, and $3100
contains $50, the result of JMP ($30FF) will be a transfer of control to $4080
rather than $5080 as you intended i.e. the 6502 took the low byte of the address
from $30FF and the high byte from $3000.

<a name=JSR></a>

## JSR (Jump to SubRoutine)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Absolute|JSR $5597|$20|3|6|

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|-|-|-|-|-|-|-|

JSR pushes the address-1 of the next operation on to the stack before
transferring program control to the following address. Subroutines are normally
terminated by a <a href="#RTS">RTS</a> opcode.

<a name=LDA></a>

## LDA (LoaD Accumulator)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|LDA #$BE|$A9|2|2|
|Zero Page|LDA $BE|$A5|2|3|
|Zero Page,X|LDA $BE,X|$B5|2|4|
|Absolute|LDA $BEEF|$AD|3|4|
|Absolute,X|LDA $BEEF,X|$BD|3|4+|
|Absolute,Y|LDA $BEEF,Y|$B9|3|4+|
|Indirect,X|LDA ($BE,X)|$A1|2|6|
|Indirect,Y|LDA ($BE),Y|$B1|2|5+|

\+ add 1 cycle if page boundary crossed

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|-|

* N set if bit 7 of .A is set
* Z set if .A is zero

Loads a byte of memory into the accumulator setting the zero
and negative flags as appropriate.

<a name=LDX></a>

## LDX (LoaD X register)

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|LDX #$BE|$A2|2|2|
|Zero Page|LDX $BE|$A6|2|3|
|Zero Page,Y|LDX $BE,Y|$B6|2|4|
|Absolute|LDX $BEEF|$AE|3|4|
|Absolute,Y|LDX $BEEF,Y|$BE|3|4+|

\+ add 1 cycle if page boundary crossed

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|-|

* N set if bit 7 of .X is set
* Z set if .X is zero

Loads a byte of memory into the X register setting the zero
and negative flags as appropriate.

<a name=LDY></a>

## LDY (LoaD Y register)
*Affects flags: N Z*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|LDY #$BE|$A0|2|2|
|Zero Page|LDY $BE|$A4|2|3|
|Zero Page,X|LDY $BE,X|$B4|2|4|
|Absolute|LDY $BEEF|$AC|3|4|
|Absolute,X|LDY $BEEF,X|$BC|3|4+|

\+ add 1 cycle if page boundary crossed

|N|V|B|D|I|Z|C|
|-|-|-|-|-|-|-|
|✓|-|-|-|-|✓|-|

* N set if bit 7 of .Y is set
* Z set if .Y is zero

Loads a byte of memory into the Y register setting the zero
and negative flags as appropriate.

<a name=LSR></a>

## LSR (Logical Shift Right)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Accumulator|LSR A|$4A|1|2|
|Zero Page|LSR $BE|$46|2|5|
|Zero Page,X|LSR $BE,X|$56|2|6|
|Absolute|LSR $BEEF|$4E|3|6|
|Absolute,X|LSR $BEEF,X|$5E|3|7|

LSR shifts all bits right one position. 0 is shifted into bit 7 and the
original bit 0 is shifted into the Carry.

<a name=WRAP></a>

## Wrap-Around
Use caution with indexed zero page operations as they are subject to
wrap-around. For example, if the X register holds $FF and you execute LDA $80,X
you will not access $017F as you might expect; instead you access $7F i.e.
$80-1. This characteristic can be used to advantage but make sure your code is
well commented.

It is possible, however, to access $017F when X = $FF by using the Absolute,X
addressing mode of LDA $80,X. That is, instead of:
<pre>
  LDA $80,X    // ZeroPage,X
</pre>

the resulting object code is: B5 80
which accesses $007F when X=$FF, use:
<pre>
  LDA $0080,X  // Absolute,X
</pre>

the resulting object code is: BD 80 00
which accesses $017F when X = $FF (a at cost of one additional byte and one
additional cycle). All of the ZeroPage,X and ZeroPage,Y instructions except
STX ZeroPage,Y and STY ZeroPage,X have a corresponding Absolute,X and
|Absolute,Y instruction. Unfortunately, a lot of 6502 assemblers don't have an
easy way to force Absolute addressing, i.e. most will assemble a LDA $0080,X
as B5 80. One way to overcome this is to insert the bytes using the .BYTE
pseudo-op (on some 6502 assemblers this pseudo-op is called DB or DFB,
consult the assembler documentation) as follows:
<pre>
  .BYTE $BD,$80,$00  // LDA $0080,X (absolute,X addressing mode)
</pre>

The comment is optional, but highly recommended for clarity.

In cases where you are writing code that will be relocated you must consider
wrap-around when assigning dummy values for addresses that will be adjusted.
Both zero and the semi-standard $FFFF should be avoided for dummy labels. The
use of zero or zero page values will result in assembled code with zero page
opcodes when you wanted absolute codes. With $FFFF, the problem is in
addresses+1 as you wrap around to page 0.

<a name=PC></a>

## Program Counter
When the 6502 is ready for the next instruction it increments the program
counter before fetching the instruction. Once it has the opcode, it increments
the program counter by the length of the operand, if any. This must be accounted
for when calculating branches or when pushing bytes to create a false return
address (i.e. jump table addresses are made up of addresses-1 when it is
intended to use an RTS rather than a JMP).

The program counter is loaded least signifigant byte first. Therefore the
most signifigant byte must be pushed first when creating a false return address.

When calculating branches a forward branch of 6 skips the following 6 bytes
so, effectively the program counter points to the address that is 8 bytes beyond
the address of the branch opcode; and a backward branch of $FA (256-6) goes to
an address 4 bytes before the branch instruction.

<a name=TIMES></a>

## Execution Times
Opcode execution times are measured in machine cycles; one machine cycle
equals one clock cycle. Many instructions require one extra cycle for
execution if a page boundary is crossed; these are indicated by a + following
the time values shown.

<a name=NOP></a>

## NOP (No OPeration)
*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Implied|NOP|$EA|1|2|

NOP is used to reserve space for future modifications or effectively REM
out existing code.

<a name=ORA></a>

## ORA (bitwise OR with Accumulator)
*Affects flags: N Z*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|ORA #$BE|$09|2|2|
|Zero Page|ORA $BE|$05|2|3|
|Zero Page,X|ORA $BE,X|$15|2|4|
|Absolute|ORA $BEEF|$0D|3|4|
|Absolute,X|ORA $BEEF,X|$1D|3|4+|
|Absolute,Y|ORA $BEEF,Y|$19|3|4+|
|Indirect,X|ORA ($BE,X)|$01|2|6|
|Indirect,Y|ORA ($BE),Y|$11|2|5+|

\+ add 1 cycle if page boundary crossed

<a name=TAX></a> <a name=TXA></a> <a name=TAY></a> <a name=TYA></a>
<a name=INX></a> <a name=DEX></a> <a name=INY></a> <a name=DEY></a>

## Register Instructions
*Affects flags: N Z*

These instructions are implied mode, have a length of one byte and require
two machine cycles.

|Mnemonic|Hex|
|-|-|
|TAX (Transfer A to X)|$AA|
|TXA (Transfer X to A)|$8A|
|DEX (DEcrement X)|$CA|
|INX (INcrement X)|$E8|
|TAY (Transfer A to Y)|$A8|
|TYA (Transfer Y to A)|$98|
|DEY (DEcrement Y)|$88|
|INY (INcrement Y)|$C8|

<a name=ROL></a>

## ROL (ROtate Left)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Accumulator|ROL A|$2A|1|2|
|Zero Page|ROL $BE|$26|2|5|
|Zero Page,X|ROL $BE,X|$36|2|6|
|Absolute|ROL $BEEF|$2E|3|6|
|Absolute,X|ROL $BEEF,X|$3E|3|7|

ROL shifts all bits left one position. The Carry is shifted into bit 0 and
the original bit 7 is shifted into the Carry.

<a name=ROR></a>

## ROR (ROtate Right)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Accumulator|ROR A|$6A|1|2|
|Zero Page|ROR $BE|$66|2|5|
|Zero Page,X|ROR $BE,X|$76|2|6|
|Absolute|ROR $BEEF|$6E|3|6|
|Absolute,X|ROR $BEEF,X|$7E|3|7|

ROR shifts all bits right one position. The Carry is shifted into bit 7
and the original bit 0 is shifted into the Carry.

<a name=RTI></a>

## RTI (ReTurn from Interrupt)
*Affects flags: all*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Implied|RTI|$40|1|6|

RTI retrieves the Processor Status Word (flags) and the Program Counter
from the stack in that order (interrupts push the PC first and then the PSW).
Note that unlike RTS, the return address on the stack is the actual address
rather than the address-1.

<a name=RTS></a>

## RTS (ReTurn from Subroutine)
*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Implied|RTS|$60|1|6|

RTS pulls the top two bytes off the stack (low byte first) and transfers
program control to that address+1. It is used, as expected, to exit a subroutine
invoked via <a href="#JSR">JSR</a> which
pushed the address-1.

RTS is frequently used to implement a jump table where addresses-1 are pushed
onto the stack and accessed via RTS eg. to access the second of four routines:
<PRE>
 LDX #1
 JSR EXEC
 JMP SOMEWHERE

LOBYTE
 .BYTE &lt;ROUTINE0-1,&lt;ROUTINE1-1
 .BYTE &lt;ROUTINE2-1,&lt;ROUTINE3-1

HIBYTE
 .BYTE &gt;ROUTINE0-1,&gt;ROUTINE1-1
 .BYTE &gt;ROUTINE2-1,&gt;ROUTINE3-1

EXEC
 LDA HIBYTE,X
 PHA
 LDA LOBYTE,X
 PHA
 RTS
</pre>

<a name=SBC></a>

## SBC (SuBtract with Carry)
*Affects flags: N V Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|SBC #$BE|$E9|2|2|
|Zero Page|SBC $BE|$E5|2|3|
|Zero Page,X|SBC $BE,X|$F5|2|4|
|Absolute|SBC $BEEF|$ED|3|4|
|Absolute,X|SBC $BEEF,X|$FD|3|4+|
|Absolute,Y|SBC $BEEF,Y|$F9|3|4+|
|Indirect,X|SBC ($BE,X)|$E1|2|6|
|Indirect,Y|SBC ($BE),Y|$F1|2|5+|

\+ add 1 cycle if page boundary crossed

SBC results are dependant on the setting of the decimal flag. In decimal
mode, subtraction is carried out on the assumption that the values involved are
packed BCD (Binary Coded Decimal).

There is no way to subtract without the carry which works as an inverse
borrow. i.e, to subtract you set the carry before the operation. If the carry is
cleared by the operation, it indicates a borrow occurred.

<a name=STA></a>

## STA (STore Accumulator)
*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|STA $BE|$85|2|3|
|Zero Page,X|STA $BE,X|$95|2|4|
|Absolute|STA $BEEF|$8D|3|4|
|Absolute,X|STA $BEEF,X|$9D|3|5|
|Absolute,Y|STA $BEEF,Y|$99|3|5|
|Indirect,X|STA ($BE,X)|$81|2|6|
|Indirect,Y|STA ($BE),Y|$91|2|6|

<a name=TXS></a> <a name=TSX></a> <a name=PHA></a> <a name=PLA></a>
<a name=PHP></a> <a name=PLP></a> <a name=STACK></a>

## Stack Instructions

These instructions are implied mode, have a length of one byte and require
machine cycles as indicated. The "PuLl" operations are known as "POP" on most
other microprocessors. With the 6502, the stack is always on page one
($100-$1FF) and works top down.

|Mnemonic|Hex|Cycle|
|-|-|-|
|TXS (Transfer X to Stack ptr)|$9A|2|
|TSX (Transfer Stack ptr to X)|$BA|2|
|PHA (PusH Accumulator)|$48|3|
|PLA (PuLl Accumulator)|$68|4|
|PHP (PusH Processor status)|$08|3|
|PLP (PuLl Processor status)|$28|4|

<a name=STX></a>

## STX (STore X register)
*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|STX $BE|$86|2|3|
|Zero Page,Y|STX $BE,Y|$96|2|4|
|Absolute|STX $BEEF|$8E|3|4|

<a name=STY></a>

## STY (STore Y register)
*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|STY $BE|$84|2|3|
|Zero Page,X|STY $BE,X|$94|2|4|
|Absolute|STY $BEEF|$8C|3|4|

# Illegal opcodes

<a name=ALR></a>

## ALR, ASR (AND + LSR)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|ALR #$BE|$4B|2|2|

This opcode ANDs the contents of the A register with an immediate value and
then LSRs the result.

<a name=ANC></a>

## ANC (AND + ROL)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|ANC #$BE|$0B|2|2|
|Immediate|ANC #$BE|$2B|2|2|

ANC ANDs the contents of the A register with an immediate value and then
moves bit 7 of A into the Carry flag. This opcode works basically
identically to AND #immediate except that the Carry flag is set to the same
state that the Negative flag is set to.

<a name=ANE></a><a name=XAA></a>

## ANE, XAA
<span class="badge badge-error">Instable</span>

*Affects flags: N Z*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|ANE #$BE|$8B|2|2|

This opcode ORs the A register with CONST, ANDs the result with X. ANDs the result
with an immediate value, and then stores the result in A.

Instability:
CONST is chip- and/or temperature
dependent (common values may be $ee, $00, $ff …). Some dependency on the RDY line.
Bit 0 and Bit 4 are “weaker” than the other bits, and may drop to 0 in the
first cycle of DMA when RDY goes low.

Do not use ANE with any immediate value other than 0, or when the accumulator
value is $ff (both take the magic constant out of the equation)! (Or, more
accurately, these are safe if all bits that could be 0 in A are 0 in either
the immediate value or X or both.)

<a name=ARR></a>

## ARR (AND + ROR)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|ARR #$BE|$6B|2|2|

This opcode ANDs the contents of the A register with an immediate value and
then RORs the result.

<a name=DCP></a><a name=DCM></a>

## DCP, DCM (DEC + CMP)
*Affects flags: N Z C

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|DCP $BE|$C7|2|5|
|Zero Page,X|DCP $BE,X|$D7|2|6|
|Absolute|DCP $BEEF|$CF|3|6|
|Absolute,X|DCP $BEEF,X|$DF|3|7|
|Absolute,Y|DCP $BEEF,Y|$DB|3|7|
|Indirect,X|DCP ($BE,X)|$C3|2|8|
|Indirect,Y|DCP ($BE),Y|$D3|2|8|

This opcode DECs the contents of a memory location and then CMPs the result
with the A register.

<a name=ISC></a><a name=ISB></a><a name=INS></a>

## ISC, ISB, INS (SBC + INC)
*Affects flags: N V Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|ISC $BE|$E7|2|5|
|Zero Page,X|ISC $BE,X|$F7|2|6|
|Absolute|ISC $BEEF|$EF|3|6|
|Absolute,X|ISC $BEEF,X|$FF|3|7|
|Absolute,Y|ISC $BEEF,Y|$FB|3|7|
|Indirect,X|ISC ($BE,X)|$E3|2|8|
|Indirect,Y|ISC ($BE),Y|$F3|2|8|

This opcode INCs the contents of a memory location and then SBCs the result
from the A register.

<a name=LAS></a><a name=LAR></a>

## LAS, LAR (TSX + TXA + AND + TAX + TXS)
<span class="badge badge-warning">Unreliable</span>

*Affects flags: N Z*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Absolute,Y|LAS $BEEF,Y|$BB|3|4+|

This opcode ANDs the contents of a memory location with the contents of the
stack pointer register and stores the result in the accumulator, the X
register, and the stack pointer.

Unreliability: LAS is called as 'probably unreliable'

<a name=LAX></a>

## LAX (LDA + LDX)
*Affects flags: N Z*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page|LAX $BE|$A7|2|3|
|Zero Page,Y|LAX $BE,X|$B7|2|4|
|Absolute|LAX $BEEF|$AF|3|4|
|Absolute,Y|LAX $BEEF,Y|$BF|3|4+|
|Indirect,X|LAX ($BE,X)|$A3|2|6|
|Indirect,Y|LAX ($BE),Y|$B3|2|5+|

This opcode loads both the accumulator and the X register with the contents
of a memory location.

<a name=LAX_Imm></a><a name=ATX></a><a name=LXA></a><a name=OAL></a>
<a name=ANX></a>

## LAX immediate, ATX, LXA, OAL, ANX
<span class="badge badge-error">Instable</span>

*Affects flags: N Z*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate|LAX #$BE|$AB|2|2|

This opcode ORs the A register with CONST, ANDs the result with an immediate
value, and then stores the result in both A and X.

Instability: CONST is chip and/or temperature
dependent (common values may be $ee, $00, $ff,…). Some dependency on the RDY line.
Bit 0 and Bit 4 are “weaker” than the other bits, and may
drop to 0 in the first cycle of DMA when RDY goes low.

Do not use LAX #imm with any immediate value other than 0, or when the
accumulator value is $ff (both take the magic constant out of the equation)!
(Or, more accurately, these are safe if all bits that could be 0 in A are 0
in the immediate value.)

<a name=RLA></a>

## RLA (ROL + AND)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page  |RLA $BE    |$27|2|5|
|Zero Page,X|RLA $BE,X  |$37|2|6|
|Absolute   |RLA $BEEF  |$2F|3|6|
|Absolute,X |RLA $BEEF,Y|$3F|3|7|
|Absolute,Y |RLA $BEEF,Y|$3B|3|7|
|Indirect,X |RLA ($BE,X)|$23|2|8|
|Indirect,Y |RLA ($BE),Y|$33|2|8|

Rotate one bit left in memory, then AND accumulator with memory.
* Carry is shifted in as LSB and bit 7 is shifted into Carry
* N and Z are set according to the AND

<a name=RRA></a>

## RRA (ROR + ADC)
*Affects flags: N V Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page  |RRA $BE    |$67|2|5|
|Zero Page,X|RRA $BE,X  |$77|2|6|
|Absolute   |RRA $BEEF  |$6F|3|6|
|Absolute,X |RRA $BEEF,X|$7F|3|7|
|Absolute,Y |RRA $BEEF,Y|$7B|3|7|
|Indirect,X |RRA ($BE,X)|$63|2|8|
|Indirect,Y |RRA ($BE),Y|$73|2|8|

Rotate one bit right in memory, then add memory to accumulator (with carry)
* Bit 0 is shifted out into the carry flag and carry flag is shifted into
bit 7 by the ROR
* then all flags are set according to the ADC

<a name=SAX></a><a name=AXS></a><a name=AAX></a>

## SAX, AXS, AAX (STA + STX)
*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page  |SAX $BE    |$87|2|3|
|Zero Page,Y|SAX $BE,X  |$97|2|4|
|Absolute   |SAX $BEEF  |$8F|3|4|
|Indirect,X |SAX ($BE,X)|$83|2|6|

AND the contents of the A and X registers (without changing the contents of either
register) and stores the result in memory.

Note that SAX does not affect any flags in the processor status register,
and does not modify A/X. It would also not use the stack.

<a name=SBX></a><a name=AXS></a><a name=SAX></a>

## SBX, AXS, SAX (CMP + DEX)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate  |SBX #$BE   |$CB|2|2|

SBX ANDs the contents of the A and X registers (leaving the contents of A intact),
subtracts an immediate value, and then stores the result in X.
It actually works just like the CMP instruction, except
that CMP does not store the result of the subtraction it performs in any register.

* This subtract operation is not affected by the state of the Carry flag,
though it does affect the Carry flag. It does not affect the Overflow flag.
(Flags are set like with CMP, not SBC)
* N and Z are set according to the value ending up in X

Another property of this opcode is that it doesn't respect the decimal mode,
since it is derived from CMP rather than SBC. So if you need to perform
table lookups and arithmetic in a tight interrupt routine there's no need
to clear the decimal flag in case you've got some code running that operates
in decimal mode.

<a name=SHA></a><a name=AHX></a><a name=AXA></a>

## SHA, AHX, AXA (STA + STX + STY)
<span class="badge badge-warning">Instable</span>

*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Indirect,Y |SHA ($BE),Y|$93|2|6|
|Absolute,Y |SHA $BEEF,Y|$9F|3|5|

This opcode stores the result of A AND X AND the high byte of the target address of
the operand +1 in memory.

Instability:
* The value written is ANDed with &{H+1}, except when the RDY line goes low
 in the 4th (opcode $9f) or 5th (opcode $93) cycle.
* When adding Y to the target address causes a page boundary crossing, the highbyte
of the target address is incremented by one (as expected), and then ANDed with
(A & X).

<a name=SHX></a><a name=SXA></a><a name=XAS></a>

## SHX, SXA, XAS (STA + STX + STY)
<span class="badge badge-warning">Instable</span>

*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Absolute,Y |SHX $BEEF,Y|$9E|3|5|

AND X register with the high byte of the target address of the argument + 1. Store the
result in memory.

Instability:
* The value written is ANDed with &{H+1}, except when the RDY line goes low
 in the 4th cycle.
* When adding Y to the target address causes a page boundary crossing, the
highbyte of the target address is incremented by one (as expected), and then
ANDed with X.

<a name=SHY></a><a name=SYA></a><a name=SAY></a>

## SHY, SYA, SAY (STA + STX + STY)
<span class="badge badge-warning">Instable</span>

*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Absolute,X |SHY $BEEF,Y|$9C|3|5|

AND Y register with the high byte of the target address of the argument + 1.
Store the result in memory.

Instability:
* The value written is ANDed with &{H+1}, except when the RDY line goes low in
the 4th cycle.
* When adding X to the target address causes a page boundary crossing, the
highbyte of the target address is incremented by one (as expected), and then
ANDed with Y.

<a name=SLO></a><a name=ASO></a>

## SLO, ASO (ASL + ORA)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page  |SLO $BE    |$87|2|5|
|Zero Page,X|SLO $BE,X  |$17|2|6|
|Absolute   |SLO $BEEF  |$0F|3|6|
|Absolute,X |SLO $BEEF,Y|$1F|3|7|
|Absolute,Y |SLO $BEEF,Y|$1B|3|7|
|Indirect,X |SLO ($BE,X)|$03|2|8|
|Indirect,Y |SLO ($BE),Y|$13|2|8|

Shift left one bit in memory, then OR accumulator with memory.
* The leftmost bit is shifted into the carry flag
* N and Z are set after the ORA

<a name=SRE></a><a name=LSE></a>

## SRE, LSE (LSR + EOR)
*Affects flags: N Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Zero Page  |SRE $BE    |$47|2|5|
|Zero Page,X|SRE $BE,X  |$57|2|6|
|Absolute   |SRE $BEEF  |$4F|3|6|
|Absolute,X |SRE $BEEF,Y|$5F|3|7|
|Absolute,Y |SRE $BEEF,Y|$5B|3|7|
|Indirect,X |SRE ($BE,X)|$43|2|8|
|Indirect,Y |SRE ($BE),Y|$53|2|8|

Shift right one bit in memory, then EOR accumulator with memory.
* LSB is shifted into the carry flag
* N and Z are set after the EOR

<a name=TAS></a><a name=XAS></a><a name=SHS></a>

## TAS, XAS, SHS (STA + TXS)
<span class="badge badge-warning">Instable</span>

*Affects flags: none*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Absolute,Y |TAS $BEEF,Y|$9B|3|5|

This opcode ANDs the contents of the A and X registers (without changing the contents
of either register) and transfers the result to the stack pointer. It then ANDs
that result with the contents of the high byte of the target address of the
operand +1 and stores that final result in memory.

Instabilities:
* The value written is ANDed with &{H+1}, except when the RDY line goes low
in the 4th cycle.
* When adding Y to the target address causes a page boundary crossing, the
highbyte of the target address is incremented by one (as expected), and then
ANDed with (A & X).

<a name=USBC></a><a name=USB></a>

## USBC, USB (SBC + NOP)
*Affects flags: N V Z C*

|Mode|Syntax|Hex|Length|Cycle|
|-|-|-|-|-|
|Immediate  |USBC #$BE  |$EB|2|2|

Subtract immediate value from accumulator with carry. Same as the regular
<a href="#SBC">SBC</a>.

## References

* 6502 Instruction Set - https://www.masswerk.at/6502/6502_instruction_set.html
* NMOS 6510 Unintended Opcodes - https://codebase64.org/lib/exe/fetch.php?media=base:nomoresecrets-nmos6510unintendedopcodes-20202412.pdf
