# Add: EFLAGS analysis

## Program 1: add1.asm
Adds two 8-bit numbers: 120 + 10, using `al`.

```
  01111000   (120)
+ 00001010   (10)
= 10000010   (0x82 = 130 unsigned, -126 signed)
```

GDB output after the add: `eflags 0xa96 [ PF AF SF IF OF ]`, `eax = 0x82`

**Flags set:**

- **OF = 1 (set):** As a signed 8-bit result, 120 + 10 = 130 exceeds +127. Two positive numbers produced a result with the sign bit set.
- **SF = 1 (set):** Bit 7 of the result `10000010` is 1.
- **PF = 1 (set):** The low byte `10000010` contains two 1s, an even count.
- **AF = 1 (set):** The low nibbles 8 + 0xA = 0x12 carry out of bit 3.

**Flags cleared:**

- **CF = 0 (cleared):** As an unsigned 8-bit result, 130 fits (max 255), so there is no carry out of bit 7.
- **ZF = 0 (cleared):** The result is not zero.

IF is the interrupt flag. The operating system sets it and `add` does not affect it.

Takeaway: the same bits are valid unsigned (130, no carry) but overflow when read as signed (-126 is wrong), which is why CF = 0 while OF = 1.

## Program 2: add2.asm
Adds two 16-bit numbers: 32000 + 500, using `ax`.

```
  0111 1101 0000 0000   (32000 = 0x7D00)
+ 0000 0001 1111 0100   (500   = 0x01F4)
= 0111 1110 1111 0100   (32500 = 0x7EF4)
```

GDB output after the add: `eflags 0x202 [ IF ]`, `ax = 32500`

No flags are set (apart from IF, which the operating system sets). All six arithmetic flags are cleared:

- CF = 0 (cleared): 32500 fits in 16 bits unsigned (max 65535), so there is no carry out of bit 15.
- OF = 0 (cleared): 32500 also fits in 16 bits signed (max +32767), so there is no signed overflow. Compare this with Program 1, where 120 + 10 went past the signed limit of +127.
- SF = 0 (cleared): Bit 15 of the result is 0, so the result is positive.
- ZF = 0 (cleared): The result is not zero.
- PF = 0 (cleared): The low byte `0xF4` is `11110100`, which has five 1s, an odd count.
- AF = 0 (cleared): The low nibbles 0 + 4 = 4 do not carry out of bit 3.

Takeaway: when the result fits in the register for both signed and unsigned interpretations, and is positive, non-zero, with odd parity, no flags are set.


## Program 2: add2.asm
Adds two 16-bit numbers: 32000 + 500, using `ax`.

      0111 1101 0000 0000   (32000 = 0x7D00)
    + 0000 0001 1111 0100   (500   = 0x01F4)
    = 0111 1110 1111 0100   (32500 = 0x7EF4)

GDB output after the add: `eflags 0x202 [ IF ]`, `ax = 32500`

No flags are set (apart from IF, which the operating system sets). All six arithmetic flags are cleared:

- CF = 0 (cleared): 32500 fits in 16 bits unsigned (max 65535), so there is no carry out of bit 15.
- OF = 0 (cleared): 32500 also fits in 16 bits signed (max +32767), so there is no signed overflow. Compare this with Program 1, where 120 + 10 went past the signed limit of +127.
- SF = 0 (cleared): Bit 15 of the result is 0, so the result is positive.
- ZF = 0 (cleared): The result is not zero.
- PF = 0 (cleared): The low byte `0xF4` is `11110100`, which has five 1s, an odd count.
- AF = 0 (cleared): The low nibbles 0 + 4 = 4 do not carry out of bit 3.

Takeaway: when the result fits in the register for both signed and unsigned interpretations and is positive, non-zero, with odd parity, no flags are set.