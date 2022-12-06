# Kernal

## 65445 $FFA5 ACPTR <a name="FFA5"></a>

ACPTR is a low-level serial I/O utility to accept a single byte from the current serial bus TALKer using full handshaking. To prepare for this routine, a device must first have been established as a TALKer (see [TALK](#FFB4)) and passed a secondary address if necessary (see [TKSA](#FF96)). The byte is returned in A. Most applications should use the higher-level I/O routines; see BASIN and GETIN.

## 65448 $FFA8 CIOUT <a name="FFA8"></a>

CIOUT is a low-level serial I/O utility to transmit a single byte to the current serial bus LISTENer using full handshaking. To prepare for this routine, a device must first have been established as a LISTENer (see [LISTEN](##FFB4)) and passed a secondary address if necessary (see [SECOND](#FF93)). The byte is passed in A. Serial output data is buffered by one character, with the last character being transmitted with EOI after a call to [UNLSN](#FFAE). Most applications should use the higher level I/O routines; see BSOUT.

## 65478 $FFC6 CHKIN <a name="FFC6"></a>

CHKIN establishes an input channel to the device associated with the logical address (LA) passed in X, in preparation for at call to BASIN or GETIN. The Kernal variable DFLTN ([153/$99](0000#99)) is updated to indicate the current input device and the variables LA, FA and SA are updated with the file's parameters from its entry in the logical file tables (put there by OPEN). CHKIN performs certain device specific tasks: screen and keyboard channels pass through, cassette files are confirmed for input, and serial channels are sent a TALK command and the SA transmitted (if necessary). Call CLRCHN to restore normal I/O channels.

CHKIN is required for all input except the keyboard. If keyboard input is desired and no other input channel is established, you do not need to call CHKIN or OPEN. The keyboard is the default input device for BASIN and GETIN.

The path to CHKIN is through an indirect RAM vector at $031E. Applications may therefor provide their own CHKIN procedure or supplement the system's by redirecting this vector to their own routine.

## 65481 $FFC9 CHKOUT <a name="FFC6"></a>

CHKOUT establishes an output channel to the device associated with the logical address (LA) passed in X, in preparation for a call to BSOUT. The Kernal variable DFLTO [154/$9A](0000#9A) is updated to indicate the current output device and the variables LA, FA and SA are updated with the file's parameters from its entry in the logical file tables (put there by OPEN). CKOUT performs certain device specific tasks: keyboard channels are illegal, screen channels pass through, cassette files are confirmed for output, and serial channels are sent a LISTN command and the SA transmitted (if necessary). Call CLRCH to restore normal I/O channels.

CKOUT is required for all output except the screen. If screen output is desired and no other output channel is established, you do not need to call CKOUT or OPEN. The screen is the default output device for BSOUT.

The path to CKOUT is through an indirect RAM vector at $0320. Applications may therefore provide their own CKOUT procedures or supplement the system's by redirecting this vector to their own routine.

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

