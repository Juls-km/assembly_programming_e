# Div: EFLAGS analysis

`div` divides an unsigned value by the operand. For an 8-bit divisor, AX is divided by the operand: the quotient goes in AL and the remainder in AH. After `div`, the flags CF, OF, SF, ZF, AF and PF are all undefined according to the Intel manual. The real results of `div` are in the registers, not in the flags.

## Program 1: div1.asm
Divides a 16-bit value by an 8-bit value: 100 ÷ 7, using `ax` and `bl`.

    100 ÷ 7 = 14 remainder 2
    AL = 14 = 0x0E (quotient)
    AH = 2  = 0x02 (remainder)

GDB output after the div: `eflags 0x212 [ AF IF ]`, `al = 14`, `ah = 2`

Flags:

- CF, OF, SF, ZF, PF and AF are all undefined after `div`. The CPU does not report anything about the division through them.
- GDB shows AF = 1 and the others as 0 in this run. This is not a result of dividing 100 by 7. It is a side effect of how this CPU performs the division, so it should not be relied on and may differ on other processors.

IF is the interrupt flag, set by the operating system and not affected by `div`.

Takeaway: unlike `add` and `sub`, `div` gives no information in the flags, so a program cannot test them to see how the division went. The results to check are the quotient and remainder. The only failure signal is a divide error exception, which happens when the divisor is 0 or the quotient is too large for the destination register (AL here).


## Program 2: div2.asm
Divides a 32-bit value by a 16-bit value: 50000 ÷ 300, using DX:AX and `bx`.

    DX:AX = 0x0000C350 (50000), BX = 0x012C (300)
    50000 ÷ 300 = 166 remainder 200
    AX = 166 = 0x00A6 (quotient)
    DX = 200 = 0x00C8 (remainder)

DX is set to 0 before the `div`, because `div` always divides the full DX:AX pair. A leftover value in DX would change the answer or cause a divide error.

GDB output after the div: `eflags 0x212 [ AF IF ]`, `ax = 166`, `dx = 200`

Flags:

- CF, OF, SF, ZF, PF and AF are all undefined after `div` according to the Intel manual.
- GDB shows AF = 1 and the others as 0, the same as in Program 1. This is not a result of dividing 50000 by 300. It is a side effect of how this CPU performs the division, so it should not be relied on.

IF is the interrupt flag, set by the operating system and not affected by `div`.

Takeaway: as in Program 1, the flags tell us nothing about the division. The quotient (AX) and remainder (DX) are the results to check. The quotient 166 fits in 16 bits, so no divide error occurred. If it had not fitted, or if BX had been 0, the CPU would have raised a divide error exception and the program would have crashed.