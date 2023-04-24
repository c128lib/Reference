---
layout: default
title: Memory map
---
# Memory map

Banking is a process in which a section of memory is addressed by the microprocessor.
The memory is said to be banked in when it is available to the microprocessor in the
current memory configuration.

The Commodore 128 is programmable in its memory configuration. BASIC and
the Machine Language Monitor give you 16 pre-programmed default configurations of
memory (referred to in BASIC as banks). For the purposes of this discussion, BASIC
banks are referred to simply as default memory configurations which are combinations
of ROM and RAM in various ranges of memory. The current configuration, whether in
BASIC or machine language, is determined by the value in the configuration register of
the C128 Memory Management Unit (MMU) chip.

The sixteen different configurations in BASIC and the Machine Language Monitor
require different values to be placed in the configuration register so that particular
combinations of ROM and RAM can be banked into memory simultaneously. For
example, the character ROM is only available in memory configuration 14 (Bank 14 in
BASIC; the fifth digit hexadecimal prefix "E" in the Machine Language Monitor),
since this configuration tells the C128 MMU to swap out the I/O registers between
$D000 and $DFFF, and replace them with the character ROM. To swap the I/O
capabilities back in, change to any configuration number that contains I/O. Next table
lists the sixteen default memory configurations available in BASIC and the Machine
Language Monitor. See also [Mmu](Mmu).

|Bank|Configuration|
|-|-|
|0|RAM(0) only|
|1|RAM(1) only|
|2|RAM(2) only (same as 0)|
|3|RAM(3) only (same as 1)|
|4|Internal ROM, RAM(0), I/O|
|5|Internal ROM, RAM(1), I/O|
|6|Internal ROM, RAM(2), I/O (same as 4)|
|7|Internal ROM, RAM(3), I/O (same as 5)|
|8|External ROM, RAM(0), I/O|
|9|External ROM, RAM(1), I/O|
|10|External ROM, RAM(2), I/O (same as 8)|
|11|External ROM, RAM(3), I/O (same as 9)|
|12|Kernal and Internal ROM (LOW), RAM(0), I/O|
|13|Kernal and External ROM (LOW), RAM(0), I/O|
|14|Kernal and BASIC ROM, RAM(0), Character ROM|
|15|Kernal and BASIC ROM, RAM(0), I/O|

## The two 64K ram banks
The Commodore 128 memory is composed of two RAM banks (labeled 0 and 1), each
having 64K of RAM, giving a total of 128K. The 8502 microprocessor address bus is 16
bits wide, allowing it to address 65536 (64K) memory locations at a time.

Although only 64K can be accessed at one time, the MMU has provisions for
sharing up to 16K of common RAM between the two RAM banks.

The 8502 microprocessor and the VIC chip can each access a different 64K RAM
bank. The 8502 RAM bank is selected by the configuration register (bits 6 and 7) and
the VIC RAM bank is selected by the RAM configuration register (bits 6 and 7).

The configuration determined by the configuration register can be composed of
RAM and ROM, where the ROM portion overlays the RAM layer underneath.

A read (PEEK) operation returns a ROM value, and a write (POKE) operation
bypasses the ROM and stores the value in the RAM underneath.
Many different combinations of memory can be constructed to comprise a 64K
configuration of accessible memory. Bits six and seven of the configuration register
specify which RAM bank lies beneath the ROM layers specified by bits zero through
five. The underlying RAM bank can be switched independently of any ROM layers on
top. For instance, you may switch from RAM Bank 0 to RAM Bank 1, while
maintaining the Kernal, BASIC and I/O.

## Memory layout

### $0000-$00FF Zero page Ram
In both the Commodore 64 and 
the Commodore 128, the block of memory that extends from $0000 to 
$00FF is known as page zero. Page zero appears in this segment of 
memory in all 16 of the C-128's memory banks, so you don't need to 
use any bank-switching techniques to access page zero in a C-128 
program. 

Page zero is a very important block of memory because data 
stored there can be accessed using a 1-byte operand- a procedure 
that saves both memory and processing time. In addition, two addressing modes- indirect indexed addressing and indexed indirect 
addressing- will not work unless their operands have zero page addresses. 

Because zero page addresses offer important benefits, page zero 
is a desirable district on the memory map of the Commodore 128. 
The engineers who designed the C-128 claimed most of it for themselves to set up the computer's operating system and BASIC interpreter. Consequently, very little space on page zero is available for 
use by user-written C-128 programs. 

When you're writing a program for the Commodore 128, as 
we'll see a little later in this chapter, there is a way to increase the
amount of space available on page zero by using some fancy bank-switching techniques. But if you don't want to go through the trouble 
of doing that, it's absolutely essential to find at least a few free 
memory locations on page zero. Unless you take special steps to 
increase the number of memory locations, that's how many free 
memory locations you'll find on page zero: a few.

See [Zero page](0000) for details.

There are only five bytes on page zero 
that have been left free for use in user-written programs: $FA, $FB, 
$FC, $FD, and $FE. Despite this scarcity of page zero space, however, a sharp-eyed programmer can usually find other zero page 
addresses that can be safely used in assembly language programs. For 
example, many of the addresses on page zero are reserved for use by 
the C-128's built-in BASIC 7.0 interpreter, and a number of others are 
only used by the floating-point math routines built into the com- 
puter's operating system. Many of the registers in these categories 
can be used by user-written programs, if the programs aren't designed to be called from BASIC and don't require the C-128's floating-point math package. 

### $0100-$01FF: 8502 Stack
The RAM space that extends from 
$0100 through $01FF in the Commodore 128 is reserved for use by 
the 8502 stack. The stack appears in this segment of RAM in all 16 of 
the C-128's memory banks, so you don't have to do any bank-switching
operations to access the stack. However, bank-switching operations can be used to move the 
C-128 stack, thus permitting you to assign individual stacks to individual programs. 

The stack is a section of memory that the Commodore 128 uses to keep track of the return 
addresses of machine language subroutines and interrupts (tempo- 
rary interruptions in normal program processing). The stack is also 
used for temporary storage of the values of memory registers during 
operations that would otherwise change those values and thus destroy them. The stack is frequently used by the C-128 operating 
system, and is also available for use in user-written programs.

### $0200-$03FF: Kernal RAM and Free RAM
The block of memory 
that extends from $0200 through $03FF contains important RAM 
vectors and routines, and is shared by all 16 of the computer's mem- 
ory banks. So this block of memory is always accessible to user- 
written programs, no matter which memory bank is active. 

### $0400-$07FF: Video Memory RAM (Text Mode) and Color RAM (Bit-Mapped Mode)
In all 16 memory banks of the Commodore 128, the block of memory that extends from $0400 through 
$07FF is ordinarily used as a screen map for 40-column text- that is, 
for the storage of data that generates 40-column text displays. Other 
blocks of memory can be used for the same purpose, but the block of 
memory at addresses $0400 through $07FF is the C-64/C-128's de- 
fault screen map; it is the RAM block used as a screen map when you 
first turn on the C-64 or the C-128. 

The $0400 through $07FF memory block is not used as a screen 
map, though, when the C-128 is in its high-resolution, or bit-mapped, 
graphics mode. When you use high-resolution graphics, this segment 
of memory is too small to hold a complete screen map. A 40-colunm 
screen map uses only 1,000 bytes of memory, but a high-resolution 
screen map requires 8,000 bytes. So, when you operate the C-128 in 
high-resolution graphics mode, a larger block of memory must be 
designated as a screen map, and the $0400 through $07FF memory 
block can then be used to control the colors that appear on the 
screen. 

### $0800-$1BFF (Bank 0): BASIC and Kernal Working Storage
The memory space that extends from $0800 to $1BFF in bank is 
reserved for use by BASIC and kernel routines. However, several 
blocks of RAM in this area are often available for use in user-written 
programs. They are: 

* $0B00 through $0BFF. This 255-byte block of 
RAM acts as a buffer when a data cassette recorder is being 
used. But it can be used as free RAM in programs that do not 
make use of a cassette recorder. 
* $0C00 through $0DFF. This 512-byte block of 
RAM (which immediately follows block $0B00 through 
$0BFF in the C-128's memory) is normally set aside for use 
as a text buffer in programs that use the C-128's RS-232 
serial port. But in programs that do not make use of the 
serial port, it is available as free RAM. 
* $0E00 through $0FFF. This 512-byte memory segment of RAM (which comes after the $0C00 through $0DFF 
block) is used for storing sprite definitions. In programs that 
don't use sprites, it can be used as free RAM. 
* $1300 through $1BFF. This block of memory can 
provide more than 2K of free RAM to programs that do not 
use foreign language keys or the C-128's function keys. This 
segment of memory is used by many of the assembly language programs in this volume. 

### $1C00-$FEFF (Bank 0): BASIC Program Text Storage
TBD

### $1C00-$3FFF (Bank 0): High-Resolution Screen Data and Color Memory (Bit-Mapped Mode)
TBD

### $0800-$FEFF (Bank 1): BASIC Variable Storage
TBD

### $4000-$FF00 (Bank 15): BASIC and Kernal ROM
* $4000-$AFFF BASIC ROM 
* $B000-$BFFF Monitor ROM 
* $C000 [Screen editor ROM](C000)
* $D000-$DFFF I/O or character generator ROM (depending on the contents of memory address $D9)
* $E000-$FFEF Kernal ROM 

### $FF00-$FF04: MMU
The memory management unit 
(MMU) register, which controls the memory resources of the Commodore 128, 
occupies memory addresses $FF00 through $FF04 in all 
16 of the computer's memory banks.

See [Mmu](Mmu) for details.

## 16K Video banks

In C128 graphics programming, a video bank is a 16K block of memory that contains
the essential portions of memory controlling the C128 graphics system: screen and
character memory. These two types of memory, which are discussed in the following
section, must lie within the 16K range of memory referred to as a (VIC) video bank.
The VIC chip is capable of addressing 16K of memory at any one time, so all graphics
memory must be present in that 16K. Since the Commodore 128 microprocessor
addresses 64K at a time, there are four video banks in each 64K RAM bank, or a total
of eight video banks.

Where you place the VIC video bank depends on your application program. The
Commodore 128 ROM operating system expects this bank in default video Bank 0, in
the bottom of RAM Bank 0. Screen and character memory may be located at different
positions within each 16K video bank, though in order to successfully program the VIC
chip, the current 16K bank must contain screen and character memory in their entirety.
You'll understand this after reading the next few pages.

The four video banks in each 64K RAM bank are set up in the memory ranges
specified here:

| Bank | Base address   |Value of bit 1-0 in $DD00|
| ---- | -------------- |-|
|  0   |     0/$0000    |11 (default)|
|  1   | 16384/$4000    |10|
|  2   | 32768/$8000    |01|
|  3   | 49152/$C000    |00|

Each RAM bank (0 and 1) has this memory layout.

Whenever you change video banks, you must add $4000 to the address of your
starting screen memory (video matrix) and character memory (bit map in bit map mode)
for each bank above 0. To change to video Bank 1, add $4000 to your starting screen
and character address; for Bank 2 add $8000; for Bank 3 add $C000. You must always
add an offset of $4000 to the start of your screen and character memory for each video
bank that is greater than zero.

## Summary of banking concept

The major features of the banking concept can be summarized as follows:
1. BASIC and the Machine Language Monitor have sixteen 64K memory configurations that give you sixteen different combinations of memory layouts.
The MMU chip, particularly the value in the configuration register, controls
most of the memory management in the Commodore 128. In order to PEEK
(read) from or POKE (write) to a particular portion of memory, you must
choose a BASIC or monitor configuration that contains the desired section of
memory.
2. The 128K of memory is divided into two 64K RAM banks. Only one bank is
addressable at a time by the microprocessor. RAM bank selection is controlled by the MMU configuration register (bits 6 and 7), which is part of the
C128 I/O memory. The VIC chip and 8502 microprocessor can each access a
different 64K RAM bank.
3. Each 64K RAM bank is divided into four 16K video segments. The screen
and character memory must both lie within the selected 16K video segment in
order to successfully display graphics and characters on the screen. For each
16K video bank higher than zero, remember to add $4000 (16384 decimal) to
the start address of screen and character memory.

Here's how the banks fit together and operate within the Commodore 128. One
64K RAM bank is always mapped into memory. Within BASIC or the Machine
Language Monitor, sixteen different memory configurations are available in a 64K bank.
To change the configuration, issue the BASIC BANK command, or precede the four
digit hexadecimal address in the Machine Language Monitor with an additional hexadecimal digit 0 through F. Outside of BASIC or the monitor, you can select other
configurations, by changing the value in the configuration register at location $FF00
(or $D500). 

Within the selected configuration, and part of the current 64K RAM bank, is a
16K range reserved for a video bank. The 16K video bank must encompass 1K of screen
memory, and either 4K of character ROM or an 8K block of memory for the bit map
data. All these components must be present in order for graphics to operate.

In essence, the bank concept can be thought of in this way: the C128 has a 16K
(VIC) video bank within a selected memory configuration within a 64K RAM bank.

## See also

* [Mmu (Memory management unit)](Mmu)
