# Kernal

## 65409 $FF81 CINT <a name="FF81"></a>
CINT is the Editor's initialization routine. Both 40- and 80-column display modes are prepared, editor indirect vectors installed, programmable key definitions asigned, and the 40/80 key scanned for default display determination. CINT sets the VIC bank and VIC nybble bank, enables the character ROM, resets SID volume, places both 40- and 80-column screens and clears them. The only thing it does not do that pertains to the Editor is I/O initialization, which is needed for IRQ's (keyscan, VIC cursor blink, split screen modes), key lines, screen background colors, etc. (see IOINIT). Because CINT updates Editor indirect vectors that are used during IRQ processing, you should disable IRQ's prior to callint it. CINT utilizes the status byte INIT_STATUS ($A04) as follows:
<pre>
$A04 bit 6 = 0 -> Full initialization. (set INIT_STATUS bit 6)
             1 -> Partial initialization. (not key matrix pointers) and (not program key definitions)
</pre>
CINT is also an Editor jump table entry ($C00).

## 65412 $FF84 IOINIT <a name="FF84"></a>
IOINIT is perhaps the major function of the Reset handler. It initializes both CIA's (timers, keyboard, serial port, user port, cassette) and the 8502 port (keyboard, cassette, VIC bank). It distinguishes a PAL system from and NTSC one and sets PALCNT ($0A03) if PAL. The VIC, SID and 8563 devices are initialized, including the downloading of character definitions to 8563 display RAM (if necessary). The system 60Hz IRQ source, the VIC raster, is started (pending IRQs are cleared). IOINIT utilizes the status byte INIT_STATUS ($0A04) as follows:
<pre>
$A04 bit 7 = 0 -> Full initialization (set INIT_STATUS bit 7)
       1 -> Partial initialization. (not 8563 character definitions)
</pre>
You should be sure IRQ's are disabled before calling IOINIT to avoid interrupts while the various I/O devices are bing initialized.

## 65433 $FF99 MEMTOP <a name="FF99"></a>
MEMTOP is used to read or set the top of system RAM, pointed to by MEMSIZE ($0A07). This call is included in the C128 for completeness, but neither the Kernal nor BASIC utilizes MEMTOP since it has little meaning in the banked memory environment of the C128 (even the RS-232 buffers are permanently allocated). Nonetheless, set the carry status to load MEMSIZE into X and Y, and clear it to update the pointer from X and Y. Note that MEMSIZE references only system RAM (RAM0). The Kernal initially sets MEMSIZE to $FF00 (MMU and Kernal RAM code start here).

## 65436 $FF9C MEMBOT <a name="FF9C"></a>
MEMBOT is used to read or set the bottom of system RAM, pointed to by MEMSTR ($0A05). This call is included in the C128 for completeness, but neither the Kernal nor BASIC utilizes MEMBOT since it has little meaning in the backed memory environment of the C128. Nonetheless, set the carry status to load MEMSTR into .X and .Y, and clear it to update the pointer from X and Y. Note that MEMSTR references only system RAM (RAM0). The Kernal initially sets MEMSTR to $1000 (BASIC text starts here).

## 65445 $FFA5 ACPTR <a name="FFA5"></a>
ACPTR is a low-level serial I/O utility to accept a single byte from the current serial bus TALKer using full handshaking. To prepare for this routine, a device must first have been established as a TALKer (see [TALK](#FFB4)) and passed a secondary address if necessary (see [TKSA](#FF96)). The byte is returned in A. Most applications should use the higher-level I/O routines; see BASIN and GETIN.

## 65448 $FFA8 CIOUT <a name="FFA8"></a>
CIOUT is a low-level serial I/O utility to transmit a single byte to the current serial bus LISTENer using full handshaking. To prepare for this routine, a device must first have been established as a LISTENer (see [LISTEN](##FFB4)) and passed a secondary address if necessary (see [SECOND](#FF93)). The byte is passed in A. Serial output data is buffered by one character, with the last character being transmitted with EOI after a call to [UNLSN](#FFAE). Most applications should use the higher level I/O routines; see BSOUT.

## 65457 $FFB1 LISTEN <a name="FFB1"></a>
LISTEN is a low-level Kernal serial bus routine that sends a LISTEN command to the serial bus device in .A. It commands the device to start reading data. Most applications should use the higher-level I/O routines; see CHKOUT.

## 65472 $FFC0 OPEN <a name="FFC0"></a>
OPEN prepares a logical file for I/O operations. It creates a unique entry in the Kernal logical file tables LAT ($0362), FAT ($36C) and SAT ($0376) using its index LDTND ([152/$98](0000#98)) and data supplied by the user via SETLFS. There can be up to then logical files OPENed simultaneously. OPEN performs device-specific opening tasks for serial, cassette and RS-232 devices, including clearing the previous status and transmitting any given filename or command string supplied by the user via SETNAM and SETBNK. The I/O status is updated appropriately and can be read via READST.

The path to OPEN is through an indirect RAM vector at $031A. Applications may therefor provide their own OPEN procedures or supplement the system's by redirecting this vector to their own routine.

## 65475 $FFC3 CLOSE <a name="FFC3"></a>
CLOSE removes the logical file (LA) passed in .A from the logical file tables and performs device-specific closing tasks. Keyboard, screen and any unOPENed files pass through. Cassette files opened for output are closed by writing the last buffer and (optionally) an EOT mark. RS-232 I/O devices are reset, losing any buffered data. Serial files are closed by transmitting a CLOSE command (if an SA was given when it was opened), sending any buffered character, and UNLSNing the bus.
There is a special provision incorporated into the CLOSE routine of systems featuring a BASIC DOS command. If the following conditions are all true, a full CLOSE is not performed; the table entry is removed but a CLOSE command is not transmitted to the device. This allows the disk command channel to be properly OPENed and CLOSEd without the disk operating system closing all files on its end:
<pre>
.C = 1	->	indicates special CLOSE
FA >= 8	->	device is a disk
SA = 15	->	command channel
</pre>
The path to CLOSE is through an indirect RAM vector at $031C. Applications may therefor provide their own CLOSE procedure or supplement the system's by redirecting this vector to their own routine.

## 65478 $FFC6 CHKIN <a name="FFC6"></a>
CHKIN establishes an input channel to the device associated with the logical address (LA) passed in X, in preparation for at call to BASIN or GETIN. The Kernal variable DFLTN ([153/$99](0000#99)) is updated to indicate the current input device and the variables LA, FA and SA are updated with the file's parameters from its entry in the logical file tables (put there by OPEN). CHKIN performs certain device specific tasks: screen and keyboard channels pass through, cassette files are confirmed for input, and serial channels are sent a TALK command and the SA transmitted (if necessary). Call CLRCHN to restore normal I/O channels.

CHKIN is required for all input except the keyboard. If keyboard input is desired and no other input channel is established, you do not need to call CHKIN or OPEN. The keyboard is the default input device for BASIN and GETIN.

The path to CHKIN is through an indirect RAM vector at $031E. Applications may therefor provide their own CHKIN procedure or supplement the system's by redirecting this vector to their own routine.

## 65481 $FFC9 CHKOUT <a name="FFC6"></a>
CHKOUT establishes an output channel to the device associated with the logical address (LA) passed in X, in preparation for a call to BSOUT. The Kernal variable DFLTO [154/$9A](0000#9A) is updated to indicate the current output device and the variables LA, FA and SA are updated with the file's parameters from its entry in the logical file tables (put there by OPEN). CKOUT performs certain device specific tasks: keyboard channels are illegal, screen channels pass through, cassette files are confirmed for output, and serial channels are sent a LISTN command and the SA transmitted (if necessary). Call CLRCH to restore normal I/O channels.

CKOUT is required for all output except the screen. If screen output is desired and no other output channel is established, you do not need to call CKOUT or OPEN. The screen is the default output device for BSOUT.

The path to CKOUT is through an indirect RAM vector at $0320. Applications may therefore provide their own CKOUT procedures or supplement the system's by redirecting this vector to their own routine.

## 65484 $FFCC CLRCHN <a name="FFCC"></a>
CLRCHN (alias CLRCH) is used to clear all open channels and restore the system default I/O channels after other channels have been established via CHKIN and/or CHKOUT. The keyboard is the default input device and the screen is the default output device. If the input channel was to a serial device, CLRCH first UNTLKs it. If the output channel was to a serial device, it is UNLSNed first.

The path to CLRCH is through an indirect RAM vector at $0322. Applications may therefore provide their own CLRCH procedures or supplement the system's by redirecting this vector to their own routine.

## 65487 $FFCF CHRIN/BASIN <a name="FFCF"></a>
CHRIN (alias BASIN) reads a character from the current input device (DFLTN [153/$99](0000#99)) and returns it in A. Input from devices other than the keyboard (the default input device) must be OPENed and CHKINed. The character is read from the input buffer associated with the current input channel:

* Cassette data is returned a character at a time from the cassette buffer at $0B00, with additional tape blocks being read when necessary.
* RS-232 data is returned a character at a time from the RS-232 input buffer at $0C00, waiting until a character is received if necessary. If RSSTAT ($0A14) is bad from a prior operation, input is skipped and null input (carriage return) is substituted.
* Serial data is returned a character at a time directly from the serial bus, waiting until a character is sent if necessary. If STATUS ([144/$90](0000#90)) is bad from a prior operation, input is skipped and null input (carriage return) is substituted.
* Screen data is read from screen RAM starting at the current cursor position and ending with a pseudo carriage return at the end of the logical screen line. The way the BASIN routine is written, the end of line (EOL) is not recog- nized. Users must therefore count characters themselves or otherwise detect when the logical EOL has been reached.
* Keyboard data is input by turning on the cursor, reading characters from the keyboard buffer, and echoing them on the screen until a carriage return is encountered. Characters are then returned one at a time from the screen until all characters input have been passed, including the carriage return. Any calls after the EOL will start the process over again.

The path to CHKIN is through an indirect RAM vector at $0324. Applications may therefore provide their own BASIN procedures or supplement the system's by redirecting this vector to their own routine.

## 65490 $FFD2 CHROUT/BSOUT <a name="FFD2"></a>
CHROUT (alias BSOUT) writes the character in .A to the current output device (DFLTO [154/$9A](0000#9A)). Output to devices other than the screen (the default output device) must be OPENed and CKOUTed. The character is written to the output buffer associated with the current output channel:

* Cassette data is put a character at a time into the cassette buffer at $0B00, with tape blocks being written when necessary.
* RS-232 data is put a character at a time into the RS-232 output buffer at $0D00, waiting until there is room if necessary.
* Serial data is passed to CIOUT, which buffers one character and sends the previous character.
* Screen data is put into screen RAM at the current cursor position.
* Keyboard output is illegal.

The path to CHROUT is through an indirect RAM vector at $0326. Applications may therefore provide their own CHROUT procedures or supplement the system's by redirecting this vector to their own routine.

## 65493 $FFD5 LOAD <a name="FFD5"></a>
This routine LOADs data from an input device into C128 memory. It can also be used to VERIFY that data in memory matches that in a file. LOAD performs device-specific tasks for serial and cassette LOADs. You cannot LOAD from RS-232 devices, the screen or the keyboard. While LOAD performs all the tasks of an OPEN, it does not create any logical files as an OPEN does. Also note that LOAD cannot "wrap" memory banks. As with any FO, the I/O status is updated appropriately and can be read via READSS. LOAD has two options that the user must select:

* LOAD vs. VERIFY: The contents of A passed at the call to LOAD deter- mines which mode is in effect. If A is zero, a LOAD operation will be performed and memory will be overwritten. If .A is nonzero, a VERIFY operation will be performed and the result passed via the error mechanism.
* LOAD ADDRESS: the secondary address (SA) setup by the call to SETLFS determines where the LOAD starting address comes from. If the SA is zero, the user wants the address in .X and .Y at the time of the call to be used. If the SA is nonzero, the LOAD starting address is read from the file header itself and the file is loaded into the same place from which it was SAVEd.

The C128 serial LOAD routine automatically attempts to BURST load a file, and resorts to the normal load mechanism (but still using the FAST serial routines) if the BURST handshake is not returned.

The path to LOAD is through an indirect RAM vector at $330. Applications may therefore provide their own LOAD procedures or supplement the system procedures by redirecting this vector to their own routine.

### Example
EXAMPLE:
```Assembly
  LDA	#length		;fnlen
  LDX	#filename
  JSR	$FFBD		;SETNAM

  LDA	#0		;load/verify bank (RAM0)
  LDX	#0		;fnbank (RAM0)
  JSR	$FF68		;SETBNK

  LDA	#0		;la (not used)
  LDX	#8		;fa
  LDY	#$FF		;sa (SA>0 normal load)
  JSR	$FFBA		;SETLFS

  LDA	#0		;load, not verify
  LDX	#<load adr	;(used only if SA = 0)
  LDY	#>load adr	;(used only if SA = 0)
  JSR	$FFD5		;LOAD
  BCS	error
  STX	end_lo
  STY	end_hi
```

## 65508 $FFE4 GETIN <a name="FFE4"></a>
GETIN reads a character from the current input device (DFLTN [153/$99](0000#99)) buffer and returns it in A. Input from devices other than the keyboard (the default input device) must be OPENed and CHKINed. The character is read from the input buffer associated with the current input channel:

* Keyboard input: A character is removed from the keyboard buffer and passed in .A. If the buffer is empty, a null ($00) is returned.
* RS-232 input: A character is removed from the RS-232 input buffer at $0C00 and passed in .A. If the buffer is empty, a null ($00) is returned (use READST to check validity).
* Serial input: GETIN automaticallyjumps to BASIN. See BASIN serial I/O.
* Cassette input: GETIN automatically jumps to BASIN. See BASIN cassette I/O.
* Screen input: GETIN automaticallyjumps to BASIN. See BASIN serial I/O.

The path to GETIN is through an indirect RAM vector at $032A. Applications may therefore provide their own GETIN procedures or supplement the system's by redirecting this vector to their own routine.

## 65511 $FFE7 CLALL <a name="FFE7"></a>
CLALL deletes all logical file table entries by resetting the table index, LDTND ([152/$98](0000#98)). It clears current serial channels (if any) and restores the default I/O channels via CLRCHN. The path to CLALL is through an indirect RAM vector at $032C. Applications may therefore provide their own CLALL procedures or supplement the system's by redirecting this vector to their own routine.

## 65520 $FFF0 PLOT <a name="FFF0"></a>
PLOT is an Editor routine that has been slightly changed from previous CBM systems. Instead of using absolute coordinates when referencing the cursor position, PLOT now uses relative coordinates, based upon the current screen window. The following local Editor variables are useful:

* SCBOT	$E4	->	window bottom
* SCTOP	$E5	->	Window top
* SCLF	$E6	->	window left side
* SCRT	$E7	->	window right side
* TBLX	$EB	->	cursor line
* PNTR	$EC	->	cursor column
* LINES	$ED	->	maximum screen height
* COLUMNS	$EE	->	maximum screen width

When called with the carry status set, PLOT returns the current cursor position relative to the current window origin (not screen origin). When called with the carry status clear, PLOT attempts to move the cursor to the indicated line and column relative to the current window origin (not screen origin). PLOT will return a clear carry status if the cursor was moved, and a set carry status if the requested position was outside the current window (no change has been made).

### Example
```Assembly
  SEC
  JSR	$FFF0   //read current position
  INX		      // move down one line
  INY		      // move right one space
  CLC
  JSR	$FFF0	  // set cursor position
  BCS	error	  // new position outside window
```

## 65523 $FFF3 IOBASE <a name="FFF3"></a>
IOBASE is not used in the C128 but is included for compatibility and completeness. It returns the address of the I/O block in X and Y.

