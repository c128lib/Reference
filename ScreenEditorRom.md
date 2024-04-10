---
layout: default
title: Screen editor ROM
---
# Screen editor ROM

If the 128 is your first computer, or if you have upgraded to
the 128 from an earlier Commodore model, then you may not
appreciate the power and versatility of the 128's full-screen
editor. The editor is so well integrated into the system that
you may take it for granted. However, if you've ever worked
with an Apple, Radio Shack, TI-99, or other computer with
limited editing features, you'll appreciate the ability to move
the cursor to any position on the screen, insert and delete
characters, and reenter entire lines just by pressing RETURN
anywhere on the line. (Imagine the frustration of having to retype an entire line of input just because a single character was
mistyped.) And, with the addition of ESC key sequences, the
128's screen editor is even more powerful and versatile than
the earlier Commodore editors from which it is descended.

In earlier Commodore models, the screen editing routines
were an integral part of the Kernal ROM, but in the 128 the
routines have been separated and moved to their own block of
ROM, occupying locations [$C000-$CFFF](C000). The
routines in this block handle keyboard input and all aspects of
text screen output for both the 40- and 80-column displays.
Special features such as bitmapped graphics and sprites are
handled in BASIC ROM, but editor routines set up the bitmapped and multicolor bitmapped screens.

The 40- and 80-column text displays constitute the main
output device for the 128, and thereby its primary mechanism
for communicating with the user. Note
that both video sources remain on at all times, so it's possible
to connect both a composite and an RGBI or monochrome
monitor and have simultaneous 40- and 80-column displays.
At any given time, however, one of the screens will be considered active and the other inactive, That is, the cursor will
"live" on only one of the displays, and all printing will be directed to that screen. The active screen is considered device 3,
the default output device for the system.

## Windows
One of the most significant new features of the 128's screen
output is that it's window-oriented. That is, printing and
cursor movement are confined to the boundaries of a window-a defined area of the screen-rather than to the absolute height and width of the screen. The window can be, and
most often is, the full size of the screen, but it can be as small
as a single character. Once you change the window boundaries, you must think in terms of windows rather than screens.
For example, you should think of the SHIFT-CLR/HOME key
as "clear window" rather than "clear screen." Only the currently defined window area will be cleared when that key
combination is pressed. You can also limit printing to certain
areas of the screen by modifying the window margins. The
128's windowing capabilities are no match for those of more
advanced computers like the Macintosh and Amiga, but they
do offer exceptional control over display formatting.

In order to understand how characters in the window are
manipulated, you must first understand the distinction between
physical lines and logical lines. A physical line is just one horizontal row of the window-the characters between the left
and right margins. However, when you print characters on the
screen and the printing overflows from one row into the next,
the editor will consider the physical lines to be linked together
into a single logical line. Lines will continue to be linked until
either the RETURN or SHIFT-RETURN characters are encountered. A logical line can be as short as a single physical line or
as long as the entire window, depending on how many characters are printed before the RETURN or SHIFT-RETURN. (By
contrast, the Commodore 64 allows logical lines to span a
maximum of two physical lines.) Some screen editor routines
operate on physical lines, others on logical lines. The entries
in this chapter indicate when an operation affects an entire
logical line.

## Line Links
The system for linking lines on the 128 is considerably different from that of earlier Commodore models. For example, the
Commodore 64 maintains a table at 217-242/$D9-$F2 which
contains the high bytes of the first address on each screen line,
with the high bit of each address byte used to indicate whether
the line is linked. The 128 has no such table; instead, it contains a four-byte line link bitmap at 862-865/$035E-$0361.
Each screen line has a corresponding bit in the map. A bit set
to %1 indicates that the corresponding line is linked to the
one above. See the entries for location 862/$035E and for the
routines at 52084-52144/$CB74-$CBB0 for more information.

The 128 also implements a tab feature using a similar bitmapping scheme. The ten bytes at 852-861/$0354-$035D
provide a bit corresponding to each screen column, A bit in
the map set to %1 corresponds to a tab stop. See the entry for
location 852/$0354 and for the routines at 51535-51597/$C94F-$C98D for more information.

The other major function of the screen editor is to handle
input from the keyboard. Although you tend to think of the
keyboard as an integral part of the computer, the system sees
it as just another peripheral, device 0. The keyboard is the default input device for the 128. It is normally scanned during
the jiffy IRQ sequence. See the entry for the SCNKEY routine
[$C55D](C000#C55D) for details. The system's two major Kernal input
routines, GETIN [$EEEB](E000#EEEB) and BASIN [$EF06](E000#EF06), both call screen
editor routines to retrieve input from the keyboard.

An enhanced feature of the 128's keyboard is programmable keys. Definition strings can be assigned to the eight function keys, F1-F8, and to the SHIFT-RUN/STOP combination
and the HELP key. The definitions can be of variable lengths,
with the restriction that the combined lengths of all ten definitions cannot exceed 246 characters.

The screen editor has another feature that makes redefining the keyboard much easier than on previous Commodore
models. The tables for decoding keyscan codes into character
codes are still in ROM, but the pointers to the tables are now
maintained in RAM. Thus, redefining the keyboard is as simple as redirecting the pointer to a new decoding table set up in
RAM. See the entry for locations 830-841/$033E-$0349 for
details.

## See also

* [$C000-$CFFF - Screen editor ROM](C000#C000)
