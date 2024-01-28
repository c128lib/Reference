---
layout: disasm
title: $4023
---
4023: 20 7A 41  JSR $417A   // Set Preconfig Registers
4026: 20 51 42  JSR $4251   // Set Basic Links
4029: 20 45 40  JSR $4045   // Set-Up Basic Constants
402C: 20 9B 41  JSR $419B   // Print Startup Message
402F: AD 04 0A  LDA $0A04
4032: 09 01     ORA #$01
4034: 8D 04 0A  STA $0A04
4037: A2 03     LDX #$03
4039: 8E 00 0A  STX $0A00   // Restart System (BASIC Warm)
403C: A2 FB     LDX #$FB
403E: 9A TXS
403F: 20 56 FF  JSR $FF56   // foenix
4042: 4C 1C 40  JMP $401C
