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
16K range reserved for a video bank. The 16K video bank must encompass IK of screen
memory, and either 4K of character ROM or an 8K block of memory for the bit map
data. All these components must be present in order for graphics to operate.

In essence, the bank concept can be thought of in this way: the C128 has a 16K
(VIC) video bank within a selected memory configuration within a 64K RAM bank.

## See also

* [Mmu (Memory management unit)](Mmu)
