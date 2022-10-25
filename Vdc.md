# VDC (video display controller)

The VDC (video display controller), officially designated the
8563, is a custom chip designed by Commodore's engineers
especially to provide the 80-column display for the 128.

The VDC provides a digital RGBI signal requiring a special monitor, 
but other than that the fundamentals of its video display
are similar to those described for the VIC chip.

The other thing that distinguishes the VDC from the VIC is the VDC's
high degree of programmability. Many of the features that are
fixed in the silicon of the VIC can be customized on the VDC
simply by storing a value in one of its registers.
For example, the VDC gives you complete control over the number of rows
and columns in the screen display, and even over the number
of pixels and scan lines in each character position.

Although the VDC does have a graphic mode (see the entry for
bit 7 of register 25/$19), it is primarily used for text
displays. In this mode, it uses the same basic elements as the
VIC: screen memory, character memory, and attribute memory
(which corresponds to the VIC's color memory).

However, all these elements for the VDC appear in a 16K block of memory
that is totally separate from the 128's system address space.
This separate block is reserved solely for the VDC. Next figure
illustrates the configuration of VDC memory.

![Vdc memory configuration](./resources/008-16-f-vdc-memory-configuration.png "Vdc memory configuration")

VDC screen memory holds screen code values identical to
the screen codes for the VIC display. For each screen position,
the screen code serves as an index into character memory to
select the pattern to be displayed in the position. All character
patterns are in RAM. The VDC has no character ROM of its
own, so the contents of the VIC's character ROM is copied
into VDC RAM during system initialization. Redefining characters for the VDC is as simple as storing the new pattern in
the proper area of character memory.

Each character definition
is 16 bytes long, but because the default character height is
eight scan lines, only the first eight bytes are used to hold
character pattern information. The remaining eight bytes are
generally padded with zeros.
To determine the starting address for the definition for any character, use the appropriate
formula from the following:
* for uppercase/graphics character set:
  address = (screen code * 16) + 8192
* for lowercase/uppercase character set:
  address = (screen code * 16) + 12288

The remaining memory area is attribute memory, which
determines the display characteristics for the character specified in screen memory.
It is called attribute memory to distinguish it from simple color
memory because each attribute
memory location specifies more than just the color. Next figure
shows the use of each bit of attribute memory

![Vdc character attributes](./resources/008-17-f-vdc-character-attributes.png "Vdc character attributes")

| Value    | VDC color                |
| -------- | -----------              |
|  0/$00   | black                    |
|  1/$01   | dark gray (light black)  |
|  2/$02   | dark blue                |
|  3/$03   | light blue               |
|  4/$04   | dark green               |
|  5/$05   | light green              |
|  6/$06   | dark cyan                |
|  7/$07   | light cyan               |
|  8/$08   | dark red                 |
|  9/$09   | light red                |
| 10/$0A   | dark purple              |
| 11/$0B   | light purple             |
| 12/$0C   | dark yellow              |
| 13/$0D   | light yellow             |
| 14/$0E   | light gray (dark white)  |
| 15/$0F   | white                    |

**Bit 4** selects the flash attribute, causing the character in
the corresponding character position to blink at the rate specified
in bit 5 of internal register 24/$18.

**Bit 5** selects the underline attribute, but the line can be moved to any
scan line of the character position.

**Bit 6** controls the reverse attribute,
which reverses the foreground and background pixels of the
character pattern. However, this attribute isn't used by the 128,
which instead has reversed character patterns as part of the standard character sets.

Finally, **bit 7** selects which of the two character sets will be used. The VIC allows only one of the two
character sets to be used at any one time, but the VDC allows
ju to select the character set independently for each character
position.

The register structure of the VDC chip is rather unusual. It
has only two registers visible in the normal system address
space. You must go through these registers to access any of the
internal functions of the VDC. Next table lists the VDC communications registers visible to the processor.
A detailed description of
both locations follows.

| Address                   | Description                  |
| ------------------------- | ---------------------------- |
| [54784/$D600](D600)       | VDC address/status register  |
| [54785/$D601](D600#D601)  | VDC data register            |

## VDC Internal Registers

| | |
|----|----------|
|0/$00|Total number of horizontal character positions|
|1/$01|Number of visible horizontal character positions|
|2/$02|Horizontal sync position|
|3/$03|Horizontal and vertical sync width|
|4/$04|Total number of screen rows|
|5/$05|Vertical fine adjustment|
|6/$06|Number of visible screen rows|
|7/$07|Vertical sync position|
|8/$08|Interlace mode control register|
|9/$09|Number of scan lines per character|
|10/$0A|Cursor mode control|
|11/$0B|Ending scan line foT cursor|
|12/$0C|Screen memory starting address (high byte)|
|13/$0D|Screen memory starting address (low byte)|
|14/$0E|Cursor position address (high byte)|
|15/$0F|Cursor position address (low byte)|
|16/$10|Light pen vertical position|
|17/$11|Light pen horizontal position|
|18/$12|Current memory address (high byte)|
|19/$13|Current memory address (low byte)|
|20/$14|Attribute memory starting address (high byte)|
|21/$15|Attribute memory starting address (low byte)|
|22/$16|Character horizontal size control register|
|23/$17|Character vertical size control register|
|24/$18|Vertical smooth scrolling and control register|
|25/$19|Horizontal smooth scrolling and control register|
|26/$1A|Foreground/background color register|
|27/$1B|Address increment per row|
|28/$1C|Character set address and memory type register|
|29/$1D|Underline scan-line-position register|
|30/$1E|Number of bytes for block write or copy|
|31/$1F|Memory read/write register|
|32/$20|Block copy source address (high byte)|
|33/$21|Block copy source address (low byte)|
|34/$22|Beginning position for horizontal blanking|
|35/$23|Ending position for horizontal blanking|
|36/$24|Number of memory refresh cycles per scan line|

### <a name="#00"></a> 0/$00 Total number of horizontal character positions 
The value in this register determines the total width (in character
positions) of each horizontal line of the display. The
value stored here should be one less than the desired number
of horizontal character positions. This total includes the active
portion of the display (where characters can be displayed), the
left and right borders, and the horizontal sync width.

The total number of horizontal pixels is given by multiplying the value
here (plus 1) by the total number of pixels per character position
(the value in bits 4-7 of register [22/$16](#16) plus 1).
The default value for this register, established during the
IO1NIT routine [$E109], is 126/$7E, This provides 127
horizontal character positions. You'll need to reduce this by half if
you enable the pixel double feature (see the entry for bit 4 of
register 25/$19). You may need to increase the value here
slightly if you use one of the interlaced modes.

### <a name="#01"></a> 1/$01 Number of active horizontal character positions
The value in this register determines how many of the horizontal character
positions specified in register [0/$00](#00) can actually be used to display
characters. The value stored here
should be the desired number of columns for the display. The
value here must be less than the value in register 0/$00. The
default value for this register is 80/$50, since the default VDC
display is an 80-column text screen. The value here also determines
the width of the bitmap when the VDC is set for graphic
mode. The bitmap width is given by multiplying the number
of character positions by the character-position width specified
in bits 0-3 of register [22/$16](#16).

The screen editor routines that support printing to the 80-
column screen assume that each screen line occupies 80 screen
memory locations. If you want the screen printing routines to
continue to function properly after you reduce the number of
active character positions in this register, you should increase
the value in register [27/$1B](#1B) so that the sum of the value in
that register plus the value in this register remains equal to 80.
Reducing the value here removes characters from the right of
the display area. To center the active display area after reducing
the number of character positions, you must reduce the
value in register [2/$02](#02). The screen editor routines will not
support a display wider than 80 columns, so you'll have to
write your own text handling routines if you want to use a
wider display.

### <a name="#02"></a> 2/$02 Horizontal sync position
The value in this register determines the character position at
which the vertical sync pulse begins. The value here also
determines the horizontal position of the active portion of the
screen within the total display.

The default value here is
102/$66. Increasing this value moves the active screen area to
the left; decreasing it moves the active area to the right.

### <a name="#03"></a> 3/$03 Horizontal and vertical sync width
**Bits 0-3**: These bits specify the width of the horizontal sync
pulse. The value here should be one greater than the desired
number of character positions for the pulse. The default value
for these bits is 9/$9, for a pulse eight character positions wide.

**Bits 4-7**: These bits specify the width of the vertical sync
pulse. The bit value here should be equal to the desired number
of scan lines for the pulse, unless the interlaced sync and
video mode is being used (in that case, use a value that is
twice the desired number of scan lines). The default value for
these bits is 4/$4, for a pulse four scan lines wide.

### <a name="#04"></a> 4/$04 Total number of screen rows
This register specifies the total height (in character positions)
of the VDC display. The value stored here should be one less
than the desired number of vertical character positions. The
total includes the rows for the active display, the top and
bottom portions of the border, and the vertical sync width.

To determine the height of the raster in scan lines, multiply the
value in this register (plus 1) by the number of scan lines per
character position (the value in register [9/$09](#09) plus 1) and add
any additional scan lines specified in register [5/$05](#05).
The proper number of scan lines for the display is a function
of the video system being used; it's different for NTSC
(North American) and PAL (European) systems. During the
IOINIT routine [$E109], the 128 checks the VIC chip to determine
which system is being used (since the VIC isn't programmable
like the VDC, there is a different version of the
VIC for each of the two video systems).

This register is then
initialized accordingly: to 32/$20 for NTSC systems or 39/$27
for PAL systems, selecting 33 or 40 rows, respectively. Since
the default character-position height is eight scan lines,
the respective total heights are 33 * 8, or 264 lines, for NTSC, and
40 * 8, or 320 scan lines, for PAL. These scan-line totals
should remain constant, so if you increase the character height
you must decrease the total number of rows, and vice versa.

### <a name="#05"></a> 5/$05 Vertical fine adjustment 
**Bits 0-4**: the total number of scan lines in the VDC's video
display should be 264 for an NTSC (North American) system
or 320 for a PAL (European) system. The number of scan lines
used in the VDC display is given by the total number of vertical
positions (specified in register [4/$04](#04)) multiplied by the
number of scan lines per character position (specified in register
[9/$09](#09)). If the result doesn't come out exactly equal to the
required 264 or 320, the VDC can add a few extra scan lines at
the end to achieve the proper result.
The value in this register
specifies the number of extra scan lines to add. The available
five bits allow up to % 11111 = 31/$1F additional scan lines.
The default character height of eight scan lines is an exact
multiple of both 264 and 320 (33 * 8 = 264 and 40 * 8 =
320). Thus, no extra scan lines are required, so this register is
initialized to 0/$00 by the Kernal IOINIT routine [$E109]. As
an example of the use of this register, assume that you increased
the character height to nine scan lines. For an NTSC
system, 264 / 9 = 29 with a remainder of 3. Thus, for this
case you should specify 29 for the total number of vertical
character positions and store a 3 in this register to provide the
required additional scan lines.

**Bits 5-7**: These bits are unused; writing to them has no effect,
and they always return %1 when read. Thus, the value you
read from this register will always be at least 224/$E0. To
mask off these bits and see only the valid bits of the register,
use AND 31 in BASIC or AND #$1F in machine language.

### <a name="#06"></a> 6/$06 Number of visible screen rows
The value in this register determines how many of the vertical
character positions specified in register [4/$04](#04) can actually be
used to display characters. The value here must be less than
the total number specified in register [4/$04](#04). The default value
established for this register by the Kernal IOINIT routine
[$E109] is 25/$19, which sets up the standard 25-row display.
One obstacle to selecting other numbers of rows is that the
screen editor ROM routines will, by default, assume a 25-line
screen.

When decreasing the number of rows, you can make
the screen editor use the reduced number by storing a value
equal to the new number of rows minus 1 in location 237/$ED,
then resetting the output window to full screen size (by
printing two cursor-home characters, for example). The screen
editor routines will not support a display with more than 25
rows, so you'll have to provide your own character manipulation
routines to use such a screen.

### <a name="#07"></a> 7/$07 Vertical sync position
The value in this register determines the vertical character
position at which the vertical sync signal will be generated. This
register can be used to adjust the vertical location of the active
display area within the screen. The default value for this register,
established by the IOINIT routine [$E109], is 29/$1D for
NTSC (North American) systems or 32/$20 for PAL (European) systems.
Decreasing the value here will move the active
display area down the screen, while increasing the value will
move the active display area upwards. However, you should
not increase the value here above the maximum number of
rows specified in register [4/$04](#04).
