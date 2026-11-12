# Stack Analysis: `01.asm`

## 1. Program Overview
`01.asm` is a 64-bit x86-64 assembly program that demonstrates basic stack operations (`push` and `pop`) and the **Last-In, First-Out (LIFO)** behavior of the stack by pushing values from `RAX` and `RBX` onto the stack and popping them into `RCX` and `RDX`.

## 2. Step-by-Step Stack Execution Flow

1. **Program Entry (`_start`):**
   * The OS initializes the stack pointer `RSP` (`0x7fffffffdb20` during testing), pointing to `argc` (`0x1`) on the initial process stack.
   * `mov rax, 100` (`0x64`) and `mov rbx, 200` (`0xc8`) load immediate values into general-purpose registers without modifying the stack.

2. **Pushing Values onto the Stack:**
   * **`push rax`:** Decrements `RSP` by **8 bytes** (`0x7fffffffdb20` $\rightarrow$ `0x7fffffffdb18`) because the x86-64 stack grows downward toward lower memory addresses, then writes `100` (`0x0000000000000064`) at `[RSP]`.
   * **`push rbx`:** Decrements `RSP` by another **8 bytes** (`0x7fffffffdb18` $\rightarrow$ `0x7fffffffdb10`) and writes `200` (`0x00000000000000c8`) at the new top of the stack `[RSP]`.

3. **Popping Values from the Stack (LIFO Order):**
   * **`pop rcx`:** Reads the 8-byte value at the current top of the stack (`0x7fffffffdb10`, holding `200`) into `RCX` and increments `RSP` by **8 bytes** (`0x7fffffffdb10` $\rightarrow$ `0x7fffffffdb18`).
   * **`pop rdx`:** Reads the next 8-byte value (`0x7fffffffdb18`, holding `100`) into `RDX` and increments `RSP` by **8 bytes** (`0x7fffffffdb18` $\rightarrow$ `0x7fffffffdb20`), restoring `RSP` to its exact initial address.

4. **Program Termination:**
   * With the stack balanced (`RSP = 0x7fffffffdb20`), the program loads `60` (`sys_exit`) into `RAX`, zeroes `RDI` (`exit code 0`), and invokes `syscall`.

## 3. Execution & `gdb` Observations

### Build & Run Commands
```bash
nasm -f elf64 -g -F dwarf 01.asm -o 01.o
ld 01.o -o 01_program
./01_program
echo $?   # Output: 0