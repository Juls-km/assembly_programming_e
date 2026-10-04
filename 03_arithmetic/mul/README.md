# Mul: EFLAGS analysis

`mul` multiplies the accumulator by the operand and puts the full result in a register pair twice the size (8-bit: AL × operand → AX). It only defines CF and OF. SF, ZF, PF and AF are undefined after `mul`.

## Program 1: mul1.asm
Multiplies two 8-bit numbers: 25 × 10, using `al`.

      0001 1001   (25)
    × 0000 1010   (10)
    = 0000 0000 1111 1010   (250 = 0x00FA, stored in AX)

GDB output after the mul: `eflags 0x202 [ IF ]`, `ax = 0xfa`, `ah = 0x0`

Defined flags:

- CF = 0 (cleared): CF is set when the upper half of the result (AH) is non-zero. Here AH = 0 because 250 fits in AL (max 255), so no significant bits were pushed into the upper half.
- OF = 0 (cleared): OF follows CF for `mul`. It is set under the same condition, and the condition is false here.

Undefined flags:

- SF, ZF, PF and AF are undefined after `mul` according to the Intel manual. GDB shows them as 0 in this run, but that is not a result of the multiplication, so they should not be relied on.

IF is the interrupt flag, set by the operating system and not affected by `mul`.

Takeaway: after `mul`, CF = OF = 0 means the result fits in the lower half, so you can safely ignore the upper half. CF = OF = 1 means the upper half contains real data.


## Program 2: mul2.asm
Multiplies two 16-bit numbers: 3000 × 200, using `ax`. The 32-bit result is split across DX:AX.

      0000 1011 1011 1000   (3000 = 0x0BB8)
    × 0000 0000 1100 1000   (200  = 0x00C8)
    = 0000 0000 0000 1001 0010 0111 1100 0000   (600000 = 0x000927C0)

    DX = 0x0009 (upper 16 bits), AX = 0x27C0 (lower 16 bits)

GDB output after the mul: `eflags 0xa03 [ CF IF OF ]`, `ax = 0x27c0`, `dx = 0x9`

Defined flags:

- CF = 1 (set): CF is set when the upper half of the result (DX) is non-zero. Here DX = 9, because 600000 is larger than 65535 and does not fit in 16 bits, so significant bits were pushed into DX.
- OF = 1 (set): OF follows CF for `mul`. It is set under the same condition, and that condition is true here.

Undefined flags:

- SF, ZF, PF and AF are undefined after `mul` according to the Intel manual. GDB shows them as 0 in this run, but that is not a result of the multiplication, so they should not be relied on.

IF is the interrupt flag, set by the operating system and not affected by `mul`.

Takeaway: compared with Program 1, where the result fit in the lower half and CF = OF = 0, here the result needs both halves and CF = OF = 1. After a `mul`, these two flags tell you whether the upper half (AH, DX or EDX) holds real data or can be ignored.