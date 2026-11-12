# Stack Analysis: `06.asm`

## 1. Program Overview
`06.asm` is a **32-bit x86** (`BITS 32`) assembly program that demonstrates **32-bit register preservation** on the stack, **4-byte (`dword`) stack slot width**, and **little-endian byte ordering** in memory. It is the 32-bit counterpart to `05.asm`, pushing `EAX` and `EBX` onto the stack and popping them in reverse order (`pop ebx`, `pop eax`) to restore their original values.

## 2. Step-by-Step Stack Execution Flow

1. **Program Entry (`_start`):**
   * Execution begins in 32-bit mode with `ESP = 0xffffcc90`.
   * `mov eax, 0x11223344` loads a 4-byte (32-bit) value into `EAX`.
   * `mov ebx, 0x1234` loads a 16-bit immediate into `EBX`, zero-extending the upper 16 bits (`0x00001234`).

2. **Pushing 32-Bit Registers onto the Stack:**
   * **`push eax`:** Decrements `ESP` by **4 bytes** (`0xffffcc90` $\rightarrow$ `0xffffcc8c`) and writes the 4 bytes of `EAX` (`0x11223344`) to `[0xffffcc8c]` in little-endian order (`44 33 22 11`).
   * **`push ebx`:** Decrements `ESP` by another **4 bytes** (`0xffffcc8c` $\rightarrow$ `0xffffcc88`) and writes the full 4-byte doubleword `0x00001234` to `[0xffffcc88]` (`34 12 00 00`).

3. **Restoring Original Registers (Reverse Order):**
   * **`pop ebx`:** Reads the top 4 bytes from `[0xffffcc88]` (`0x00001234`) back into `EBX` and increments `ESP` by **4 bytes** (`0xffffcc88` $\rightarrow$ `0xffffcc8c`).
   * **`pop eax`:** Reads the next 4 bytes from `[0xffffcc8c]` (`0x11223344`) back into `EAX` and increments `ESP` by **4 bytes** (`0xffffcc8c` $\rightarrow$ `0xffffcc90`), returning `ESP` to its initial address and restoring both registers.

4. **Program Termination:**
   * With `ESP` balanced (`0xffffcc90`), the program loads `1` (`sys_exit`) into `EAX`, zeroes `EBX` (`exit code 0`), and invokes `int 0x80` to exit cleanly.

## 3. Execution & `gdb` Observations

### Build & Run Commands
```bash
nasm -f elf32 -g -F dwarf 06.asm -o 06.o
ld -m elf_i386 06.o -o 06_program
./06_program
echo $?   # Output: 0