# Kernal

## 65478 $FFC6 CHKIN <a name="FFC6"></a>

CHKIN establishes an input channel to the device associated with the logical address (LA) passed in X, in preparation for at call to BASIN or GETIN. The Kernal variable DFLTN ([153/$99](0000#99)) is updated to indicate the current input device and the variables LA, FA and SA are updated with the file's parameters from its entry in the logical file tables (put there by OPEN). CHKIN performs certain device specific tasks: screen and keyboard channels pass through, cassette files are confirmed for input, and serial channels are sent a TALK command and the SA transmitted (if necessary). Call CLRCHN to restore normal I/O channels.

CHKIN is required for all input except the keyboard. If keyboard input is desired and no other input channel is established, you do not need to call CHKIN or OPEN. The keyboard is the default input device for BASIN and GETIN.

The path to CHKIN is through an indirect RAM vector at $031E. Applications may therefor provide their own CHKIN procedure or supplement the system's by redirecting this vector to their own routine.
