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

| Address       | Description                  |
| ------------- | ---------------------------- |
| [54784/$D600](D600)   | VDC address/status register  |
| [54785/$D601](D601)  | VDC data register            |
