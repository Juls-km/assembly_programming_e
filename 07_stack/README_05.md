# Stack Analysis: `05.asm`

## 1. Program Overview
`05.asm` is a 64-bit x86-64 assembly program that demonstrates **register preservation** on the stack, **64-bit stack slot width**, and **little-endian byte ordering** in memory. Unlike `01.asm` (which popped values into different registers), `05.asm` pops in reverse order (`pop rbx` then `pop rax`) to restore the exact original register state.

## 2. Step-by-Step Stack Execution Flow

1. **Program Entry (`_start`):**
   * Execution begins with `RSP = 0x7fffffffdb20`.
   * `mov rax, 0x1122334455667788` loads an 8-byte (64-bit) value into `RAX`.
   * `mov rbx, 0x1234` loads a 16-bit immediate into `RBX`, zero-extending the upper 48 bits (`0x0000000000001234`).

2. **Pushing 64-Bit Registers onto the Stack:**
   * **`push rax`:** Decrements `RSP` by **8 bytes** (`0x7fffffffdb20` $\rightarrow$ `0x7fffffffdb18`) and writes all 8 bytes of `RAX` (`0x1122334455667788`) to `[0x7fffffffdb18]` in little-endian order (`88 77 66 55 44 33 22 11`).
   * **`push rbx`:** Decrements `RSP` by another **8 bytes** (`0x7fffffffdb18` $\rightarrow$ `0x7fffffffdb10`) and writes the full 8-byte quadword `0x0000000000001234` to `[0x7fffffffdb10]` (`34 12 00 00 00 00 00 00`).

3. **Restoring Original Registers (Reverse Order):**
   * **`pop rbx`:** Reads the top 8 bytes from `[0x7fffffffdb10]` (`0x0000000000001234`) back into `RBX` and increments `RSP` by **8 bytes** (`0x7fffffffdb10` $\rightarrow$ `0x7fffffffdb18`).
   * **`pop rax`:** Reads the next 8 bytes from `[0x7fffffffdb18]` (`0x1122334455667788`) back into `RAX` and increments `RSP` by **8 bytes** (`0x7fffffffdb18` $\rightarrow$ `0x7fffffffdb20`), returning `RSP` to its initial address and restoring both registers.

4. **Program Termination:**
   * With `RSP` balanced (`0x7fffffffdb20`), `RAX` is overwritten with `60` (`sys_exit`), `RDI` is zeroed (`exit code 0`), and `syscall` exits cleanly.

## 3. Execution & `gdb` Observations

### Build & Run Commands
```bash
nasm -f elf64 -g -F dwarf 05.asm -o 05.o
ld 05.o -o 05_program
./05_program
echo $?   # Output: 0