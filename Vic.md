# VIC (Video Interface Controller)

The chip whose registers appear in the $D000-$D030 area of the I/O block
is referred to in Commodore literature as the VIC, even
though it is not exactly the same as the chip with that designation in the 
Commodore 64. However, the chip does provide the same 40-column video display 
features as its Commodore 64 predecessor.

The differences are in the chip's
less familiar, but equally vital function of providing all the
basic riming signals required by the system. The 128's version
of the VIC chip also supports the scanning of the additional 24
keys on the 128's keyboard. For these new features, the 128
version of the VIC has two more registers than its Commodore
64 counterpart (49 instead of 47).

There are actually two different versions of the 128 VIC chip, depending
on the video system required in the country where the computer is sold.
For NTSC (North American) video, the version is officially
designated the 8564 chip, while the PAL (European) model is
designated the 8566. All the registers described below operate
the same on both chips; only the video signal format is different.

## Video Fundamentals
The output signals from the VIC chip tell the video device
(monitor or television) how to draw the screen display.
To understand the operation of the VIC chip, you need to understand
a few of the fundamentals of video displays.

The display is drawn on the monitor or television picture tube by a "gun"
that shoots a beam of electrons at the screen. Where the beam
strikes the face of the screen, a spot on the screen's phosphorescent
coating glows briefly. The electron gun doesn't just
spray electrons at random; the beam is moved in a precisely
controlled pattern. Beginning in the upper left corner of the
screen, the beam is scanned (moved) horizontally across to the
right edge of the screen, drawing a very thin line of dots. It is
then blanked while it is moved back to the left edge, but just
below the top line.

The beam is then scanned horizontally
across the screen again, and the process is repeated until the
stack of thin lines fills the screen display.

Actually, for a color display there are three separate
beams working in conjunction to draw each line—one each
for the colors red, green, and blue. Each dot in the thin screen
line consists of red, green, and blue points. When the relative
intensities of the red, green, and blue points are varied, the
dot can take on a variety of hues. The VIC chip can produce
16 different colors. Whenever a memory location or VIC register
calls for a color value, the color is specified by a value in
the range 0-15. Next table lists the standard designations for
the VIC chip colors.

| Value    | VIC color                |
| -------- | -----------              |
|  0/$00   | black                    |
|  1/$01   | white                    |
|  2/$02   | red                      |
|  3/$03   | cyan                     |
|  4/$04   | purple                   |
|  5/$05   | green                    |
|  6/$06   | blue                     |
|  7/$07   | yellow                   |
|  8/$08   | orange                   |
|  9/$09   | brown                    |
| 10/$0A   | light red                |
| 11/$0B   | dark gray                |
| 12/$0C   | medium gray              |
| 13/$0D   | light green              |
| 14/$0E   | light blue               |
| 15/$0F   | light gray               |

The stack of horizontal video lines is called the raster
(from the Latin word for rake—the pattern of evenly spaced
parallel lines is similar to that produced by pulling a rake
through soil). The individual lines are called raster scan lines.

The number of lines required for a full screen depends on the
video system in use. The North American (NTSC) version of
the VIC chip draws a raster of 264 scan lines,
while the European (PAL) version draws 313. Since the screen phosphor
glows only briefly when struck by the raster beam, the screen
must be constantly redrawn.

The rate of redrawing also depends on the video system: 60 times per second
for NTSC systems or 50 times per second for PAL systems. Not all of these
raster lines are used for the active video display. Most televisions and
monitors overscan. That is, some raster lines at the
top, bottom, or both are actually drawn off the screen.

The VIC compensates by restricting the active portion of the display,
the area where characters and graphics can be displayed,
to 200 lines in the middle of the raster for both NTSC and
PAL systems (this can be reduced to 192 lines. For details,
see the entry for [53265/$D011](D011)). The inactive lines form the
top and bottom portions of the border, a solid-color frame
around the active screen.

The horizontal dots that make up each scan line are called
pixels (short for picture elements). The number of pixels in a
scan line depends on the screen mode, and is limited by the
speed at which the VIC chip can read data from memory.

In the standard two-color modes, where a single bit determines
the pixel color, the VIC chip draws 320 active pixels per scan
line. In the four-color (multicolor) modes, two bits are required
to specify the color of each pixel. Since the VIC must read
twice as much data per pixel, only half as many pixels can be
drawn in the time allotted for a single scan line. As a result,
the multicolor modes have only 160 active pixels per scan line.

The display is still the same size; the pixels are twice as wide
(the screen width can also be reduced to 304 standard pixels
or 152 multicolor pixels. For details, see the entry for
[53270/$D016](D016)). Just as the VIC draws extra lines above and below the
active ones, it also draws extra pixels to the left and right of
the active ones. The inactive pixels form the sides of the
solidcolor border.

The VIC chip supports two major classes of display
modes—character and bitmapped. These are also referred to
as low resolution and high resolution, but that's somewhat
misleading, since both provide the same active screen areas—
320 pixels X 200 lines for standard modes or 160 pixels X 200
lines for multicolor modes.

The difference between the classes is in the degree of control
over individual pixels.

The bitmapped (high-resolution) modes allow you to determine the color of
each pixel individually, while the character (low-resolution)
modes only allow control of groups of pixels. The tradeoff is
that the character display modes use much less memory.

## Video Banks
Before you read a discussion of the display modes, it's
important to understand how the VIC chip sees memory.

The VIC uses the same RAM as the 8502 microprocessor, but it views
the memory very differently. The VIC chip has only 14 address lines
compared to the processor's 16. This means that
the VIC can, at any given time, address only 16K (16384
bytes) out of the 64K of RAM in a block. The 16K area seen
by the VIC chip is referred to as a video bank, not to be confused
with one of the processor's bank configurations—there
is no relationship.

All of the information for the VIC screen
display must be visible within the same video bank. There are
four possible video banks per 64K block, or a total of eight
possible video banks in the two RAM blocks in the 128. Bits
0-1 of the CIA #2 register at [56576/$DD00](DD00) select one of the
four banks, and bit 6 of the MMU register at [54534/$D506](D506) selects
which 64K-RAM block the video bank will be seen in.
Refer to the entries for those locations later in this chapter for
more details. The base (starting) addresses for the banks are as
follows:

| Bank | Base address   |
| ---- | -------------- |
|  0   |     0/$0000    |
|  1   | 16384/S4000    |
|  2   | 32768/$8000    |
|  3   | 49152/$C000    |

## Character Display Modes
The VIC provides three character display modes: standard,
multicolor, and extended background color.

The standard
character display mode is the default system for the VIC—the
one which is active when no other mode is selected. The other
two modes are not directly supported by the 128 operating
system (there's no GRAPHIC statement to select these modes),
so you must enable them by directly setting the appropriate
bits in VIC registers. As a result, those modes are a bit more
difficult to use effectively.

For a standard (GRAPHIC 0) character display, the 320
pixel X 200 line active screen area is divided into 1000 8-pixel
X 8-line character positions, arranged as 25 rows with 40
character positions per row. The contents of the character positions are 
determined by values stored in a 1000-byte area of
screen memory (sometimes referred to as the video matrix).
Each location in screen memory corresponds to a single character position on the 
screen. The value in a screen memory location selects one of 256 standard 
character-pattern definitions
to be drawn in the corresponding character position.

The screen memory values are referred to as screen codes, and
they are not the same as character codes. See Appendix C for
a list of screen codes and corresponding character patterns.
The location of screen memory within the current video bank
is controlled by bits 4-7 of the VIC register at [53272/$D018](D018).
See the entry for that register for details.

The pattern definitions come from another area of memory known as
character memory. As mentioned above, each
character position consists of eight scan lines with eight pixels
per line—a total of 64 pixels per position. In standard character mode,
a pixel can be one of two colors, so only one bit
(which can be either %0 or %1) is required per pixel. Thus, a
character-pattern definition requires 64 bits, or eight bytes.
The pixels represented by %0 bits in the pattern definition are
drawn in what is referred to as the background color, which is
common to all screen positions. The pixels represented by %1
bits are drawn in what is referred to as the foreground color,
which can be independently selected for each character position.

The location of character-pattern memory within the current video bank
is controlled by bits 1-3 of the VIC register at
[53272/$D018](D018). See the entry for that register for more details.
Standard character definitions for the 128 come from the character ROM.
This ROM is located beginning at address [53248/$D000](D000)
in the system's address space, but can be made visible
in any video bank. See the section on character ROM later in
this chapter for more information.

The background color for %0 bits in all character positions
is determined by the value in the VIC register at
[53281/$D021](D021). The foreground color for the %1 bits in each
character position is determined by values in another 1000-
byte area of memory known as color memory. As in screen
memory, each location in color memory corresponds to a character
position on the screen.

Unlike the case of screen and
character memory, however, the location of VIC color memory
is fixed and does not have to be within the current video bank.
It always appears to the processor at locations 55296-56319/
$D800-$DBFF in the I/0 block. Refer to the section on color
memory later in this chapter for details.
The procedure for displaying the character A in blue at
the upper left corner of the screen would be something like
this: The VIC looks to the first location of screen memory to
determine which character pattern to display in that position.

The screen code for A is 1, so this value is used as an index to
the eight-byte pattern definition in character memory. The VIC
then looks to the first location of color memory and proceeds
to draw pixels in the color specified there (6 for blue) for all
%1 bits in the pattern. For all %0 bits, pixels are drawn in the
color specified in the background color register. Next figure
illustrates the process.

![Vic standard character display mode](./resources/008-01-f-vic-standard-character-display-mode.png "Vic standard character display mode")

## Custom Characters
You are not limited just to the character patterns provided in
the ROM. It is relatively simple to design your own characters.
However, using custom characters is an all-or-nothing affair.
Once you switch off the standard ROM-based character set,
you must provide definitions for every character you wish to
use. You must begin by selecting the area of RAM where you
will place the new character set. See the entry for the register
at [53272/$D018](D018) for details.

If you only want to use a few custom characters while retaining
the majority of the standard
character set, the next step is to copy the standard character
patterns from ROM to the selected RAM area. If you do this,
you'll only have to provide custom pattern information for
those characters you wish to redefine.
To calculate the proper byte values for a custom character
definition, use an 8 X 8 grid as shown in the next figure. Fill in
the grid squares for those pixels you wish to have displayed in
the foreground color. Unless you are designing patterns that
will connect with adjacent patterns (such as the line segments
in the standard character set), it is customary to leave at least
one row and one column of the pattern blank to provide some
horizontal and vertical separation between characters.

The
ROM patterns for the standard letters and numbers don't fill
in any pixels in the leftmost column, and only lowercase characters
with descenders (g, j, p, q, and y) use the bottom row.
To calculate the binary bit pattern for each row of the pattern,
use a %0 for each blank (background) pixel and a %1 for
each filled (foreground) pixel. Next, convert the binary
bit pattern to a number (use the machine language monitor's
number conversion feature if you're not handy with binary). The final
step is to store the resulting eight values in the character
memory locations for the pattern you are changing.
The simple formula for finding the starting address in
character memory of the pattern for any character is:

pattern address = character base address + (8 * screen code)

The character base address is the starting address of character
memory (see the entry for [53272/$D018](D018)).
For example, to replace the British pound symbol (£, screen code 28/$1C) with
the pattern shown in next figure, you could use statements like
the following:

![Vic custom character design grid](./resources/008-02-f-vic-custom-character-design-grid.png "Vic custom character design grid")

## Multicolor Character Mode
Multicolor character mode is similar in operation to standard
character mode. The difference is that each multicolor character
position consists of 4 pixels X 8 lines (instead of 8 X 8).
The number of positions remains the same (25 rows X 40 columns)
and the positions are the same size, but now each pixel
is twice as wide (there are only 160 pixels per raster line).
However, each pixel can be one of four colors instead of just
one of two colors. Screen memory still holds pointers to pattern
definitions in character memory, but the pattern information is
interpreted differently. It now takes two bits per
pixel to select the color instead of just one, but since there are
only half as many pixels per pattern, the number of bits required for each definition remains the same (2 bits per pixel *
4 pixels * 8 lines = 64 bits).

To select multicolor character mode, you must set bit 4 of
the VIC register at location [53270/$D018](D016). However, there's a
problem here because the screen editor IRQ routine always resets
this bit to %0 when setting up the text screen (see the section
below on the screen editor IRQ routine). To prevent this,
you must turn off the screen-setup portion of the IRQ by storing
the value 255/$FF in location 216/$D8. Setting the VIC
register bit makes it possible to enable multicolor character
mode for any or all screen positions, but it doesn't actually
switch any screen positions to multicolor mode.

Multicolor
mode must be enabled individually for each character position
on the screen. The controlling factor is **bit 3** of the
color memory location for each character position. When that bit is %0
in a color memory position, the corresponding character position
on the screen remains in standard character mode, so it is
possible to intermix standard and multicolor character modes
on the same screen display. However, only **bits 0-2** of the
color memory location are now available to hold color values,
so the foreground color for standard mode positions in a multicolor display is limited to the first eight values in Vic color table black-yellow,
values 1-7.

To select multicolor character mode
for a screen position, you must set bit 3 of the corresponding
color memory location to %1 . That is, you must store a value
of 8 or greater in the location.

When a multicolor character is drawn, all pixels in the
pattern represented by %00 bit pairs will be drawn in the
background color specified in the VIC register at [53281/$D021](D021).
All pixels represented by %01 bit pairs will be drawn
in the color specified by the value in the register at [53282/$D022](D022),
and all pixels for %10 bit pairs will be drawn in the
color specified in the register at [53283/$D023](D023).

All of these
registers can take any of the 16 standard colors listed in Vic color
table, but since the registers are common to all positions, the
color for pixels with %00, %01, and %10 patterns will be the
same for all characters. However, the color for pixels with
%11 bit patterns can be specified individually for each screen
position in the corresponding color memory location. Since bit
3 of the color memory location is used to specify multicolor
mode, only bits 0-2 are available to hold color values.

As a result, only the first eight colors are available. But since bit 3
must be set to %1 , the values you store in color memory to
achieve these colors are different from the standard values. For
multicolor character mode, the values to store in color memory
to select the available colors for %11 bits are as follows:

| Desired %11 bit color | Value to store in color memory |
| ----------------------| ------------------------------ |
| black                 |  8/$08                         |
| white                 |  9/$09                         |
| red                   | 10/$0A                         |
| cyan                  | 11/$0B                         |
| purple                | 12/$0C                         |
| green                 | 13/$0D                         |
| blue                  | 14/$0E                         |
| yellow                | 15/$0F                         |

Because the standard character sets were not designed to
be displayed in multicolor character mode, any text printed to
the screen in this mode will be at best barely legible.
As a result, multicolor mode is practical only when you are
using custom characters designed specifically for this mode. The rules
for designing custom characters for multicolor mode are the
same as for the standard character mode, except that you
design the characters in a 4 X 8 grid, as shown in Figure 8-3.

![Vic custom character design grid](./resources/008-03-f-vic-multicolor-character-design-grid.png "Vic multicolor character design grid")

Each grid position can hold one of four two-bit values
representing the four color choices:

| | |
|-|-|
| %00 | Background color 0 (common to all characters) |
| %01 | Background color 1 (common to all characters) |
| %10 | Background color 2 (common to all characters) |
| %11 | Foreground color {independently selectable for all characters, but only eight colors are available) |

Once the design is completed, the byte values for the
character-pattern definition are calculated just as for
standard character mode.

## Extended Background Color Mode
The third character mode, extended background color mode, is
selected by setting bit 6 of the VIC register at [53265/$D011](D011) to
%1. It also uses the same fundamental elements as standard
character mode: screen memory, character memory, and color
memory. As in standard character mode, the extended background color
mode screen is divided into a 25-row X 40-column
grid of 8-pixel X 8-line character positions.

As in standard
character mode, each position has a corresponding screen
memory location that holds a value indicating which pattern
from character memory is to be drawn in the position. And, as
in standard character mode, each character position on the
screen takes its foreground color (the color for pixels represented
by %1 bits in the character pattern) from the value in a
corresponding color memory location. The difference is that
extended background color mode allows you to select from
among four different background colors for the pixels represented
by %0 bits in the character patterns.

The background color for each position is specified by bits
6-7 of the screen code for the position. These bits select which
of the four background color registers will specify the background
color for the position:

|Bits 7-6 | Background color source                   |
|-------- | ----------------------------------------- |
|   0 0   | background color register 0 (53281/$D021) |
|   0 1   | background color register 1 (53282/$D022) |
|   1 0   | background color register 2 (53283/SD023) |
|   1 1   | background color register 3 (53284/$D024) |

Since the highest two bits of each screen memory location
are used to specify background color, only bits 0-5 are available
to hold screen code data. Thus, there are only 64 different
unique screen code values (0-63), so only the first 64 eight-bit
pattern definitions in character memory are used in this mode.

For example, screen memory values of 1/$01, 65/$41, 129/$81,
and 193/$C1 all produce the same character (screen code 1,
the letter A in the standard character set), but each provides a
different background color for that character.

TBC page 175 https://www.cubic.org/~doj/c64/mapping128.pdf
