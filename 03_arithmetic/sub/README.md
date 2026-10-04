# Sub: EFLAGS analysis

## Program 1: sub1.asm
Subtracts two 8-bit numbers: 50 - 80, using `al`.

      0011 0010   (50)
    - 0101 0000   (80)
    = 1110 0010   (0xE2 = 226 unsigned, -30 signed)

GDB output after the sub: `eflags 0x287 [ CF PF SF IF ]`, `al = 0xe2`

Flags set:

- CF = 1 (set): Treated as unsigned, 50 is smaller than 80, so the subtraction needs a borrow. For `sub`, CF acts as the borrow flag.
- SF = 1 (set): Bit 7 of the result `11100010` is 1.
- PF = 1 (set): The low byte `11100010` contains four 1s, an even count.

Flags cleared:

- OF = 0 (cleared): Treated as signed, 50 - 80 = -30 fits in 8 bits (range -128 to +127), so there is no signed overflow.
- ZF = 0 (cleared): The result is not zero.
- AF = 0 (cleared): The low nibbles 2 - 0 = 2 need no borrow from bit 4.

IF is the interrupt flag, set by the operating system and not affected by `sub`.

Takeaway: the same result `0xE2` means 226 unsigned (which is wrong, hence the borrow in CF) and -30 signed (which is correct, hence OF = 0). CF and OF report on the two interpretations separately.q


## Program 2: sub2.asm
Subtracts two 16-bit numbers: 1000 - 2000, using `ax`.

      0000 0011 1110 1000   (1000 = 0x03E8)
    - 0000 0111 1101 0000   (2000 = 0x07D0)
    = 1111 1100 0001 1000   (0xFC18 = 64536 unsigned, -1000 signed)

GDB output after the sub: `eflags 0x287 [ CF PF SF IF ]`, `ax = 0xfc18`

Flags set:

- CF = 1 (set): Treated as unsigned, 1000 is smaller than 2000, so the subtraction needs a borrow out of bit 15.
- SF = 1 (set): Bit 15 of the result `1111 1100 0001 1000` is 1.
- PF = 1 (set): The low byte `0x18` is `00011000`, which contains two 1s, an even count.

Flags cleared:

- OF = 0 (cleared): Treated as signed, 1000 - 2000 = -1000 fits in 16 bits (range -32768 to +32767), so there is no signed overflow.
- ZF = 0 (cleared): The result is not zero.
- AF = 0 (cleared): The low nibbles 8 - 0 = 8 need no borrow from bit 4.

Takeaway: this is the 16-bit version of Program 1 and gives the same flags. A smaller number minus a larger number always sets CF (unsigned borrow) and SF (negative result), and OF stays 0 as long as the negative answer fits in the register.