# TMC MICROCODE

TMC - Tolek's Minecraft Computer,
a general purpose computer featuring:
- 8-bit data bus
- 4-bit address bus

## Architecture choices

All bits are 0-indexed.
Opcodes are 8-bit.
Addresses are 4-bit.
Microcode is 16-bit.

Opcode layout is as follows:
```
7 6 5 4 3 2 1 0
M A A A C C C C
```
where:
- M - 1-bit addressing mode on bit 7: 0 - absolute/indirect/implied, 1 immediate
- A A A - 3-bit ALU operation subselector on bits 4-6: 000 - add/subtract, 001 - not, 010 - xor, 100 - and
- C C C C - 4-bit opcode

NB: ALU operation subselector is ignored when non-ALI opcodes are processed.

Example: LDAI - load register A immediate
```
7 6 5 4 3 2 1 0 | 7 6 5 4 3 2 1 0
1 0 0 0 0 0 0 0 | 0 0 0 0 1 1 1 1 
^ - - - ^ ^ ^ ^ | Immediate value (i.e. the last byte of 4-bit address space)
|             |
|             Opcode for LDAI
|
Immediate flag
```

## Preamble to all opcodes

All opcodes are prefixed with the following microcode:
``CO MI / RO II / CI``

## Memory-register transfers

``00000000 LDA addr`` - Load A from memory.<br/>
Microcode: ``CO MI / RO MI / RO AI / CI``

``10000000 LDAI val`` - Load A immediate<br/>
Microcode: ``CO MI / RO AI / CI``

``00000001 LDB addr`` - Load B from memory<br/>
Microcode: ``CO MI / RO MI / RO BI / CI``

``10000001 LDBI val`` - Load B immediate<br/>
Microcode: ``CO MI / RO BI / CI``

``00000010 STA addr`` - Store A to memory<br/>
Microcode: ``CO MI / RO MI / AO RI / CI``

``00000011 STB addr`` - Store B to memory<br/>
Microcode: ``CO MI / RO MI / BO RI / CI``

## Arithmetics

``00000100 ADD addr`` - Add registers A and B and store to memory<br/>
Microcode: ``CO MI / RO MI / +O RI / CI``

``00000101 SUB addr`` - Subtract registers A and B and store to memory<br/>
Microcode: ``CO MI / RO MI / +O RI +S / CI``

## Logic

``00011000 NOTB`` - Logic NOT on register B<br/>
``00101000 XOR`` - Logic A XOR B<br/>
``01001000 AND`` - Logic A AND B<br/>
Microcode: ``CO MI / RO MI / AND || XOR || NOTB RI / CI ``

## Branching

``10000110 JMP address`` - Jump to address immediate<br/>
Microcode: ``CO MI / RO J / CI``
(This is a bug. There should be no CI at the end.)

## Port-mapped operations

``00000110 OUTA`` - Output from register A to port<br/>
Microcode: ``AO DI / CI``

``00000111 OUTB`` - Output from register A to port<br/>
Microcode: ``BO DI / CI``

# Control lines

Control lines go out from instruction decoder to concrete building blocks of processor.
AI - Register A input
AO - Register A outpout
BI - Register B input
BO - Register B output
CI - Program counter increment
CO - Program counter output
J  - Jump (program counter input)
MI - Memory address register input
RI - RAM input
RO - RAM output
+O - ALU output
+S - ALU subtract mode
DO - Data output to port
II - Instruction register input
ST - Step pulse
CLK- Clock pulse

```
00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
AI AO BI BO CI J  CO MI RI RO +O +S DO II ST CLK
```

## Instruction decoder steps

Each opcode is executed in maximum 8 steps.

Example LDA:
```
Step        | 1     | 2     | 3     | 4     | 5     | 6     |
Microcode   | CO MI | RO II | CI    | CO MI | RO AI | CI    |
Values      | 192   | 8704  | 16    | 192   | 513   | 16    |
```

## Microcode ROM mappings

Instruction microcode is indexed by 8-bit value made of 1-bit addressing mode, 4-bit opcode and 3-bit step number.

Example LDAI:
```
Mode Opcode Step
0    0000   000  => 192
0    0000   001  => 8704
0    0000   010  => 16
0    0000   011  => 192
0    0000   100  => 513
0    0000   101  => 16
```
