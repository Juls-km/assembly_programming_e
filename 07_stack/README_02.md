# Stack Analysis: `02.asm`

## 1. Program Overview
`02.asm` is a 64-bit x86-64 assembly program that demonstrates how a function call (`call`/`ret`) uses the stack to store return addresses and how a **function stack frame** (`RBP` and `RSP`) allocates and cleans up local variables in memory.

## 2. Step-by-Step Stack Execution Flow

1. **Program Entry (`_start`):**
   * Execution begins with `RSP = 0x7fffffffdb20` and `RBP = 0x0`.
   * **`call work`:** Implicitly pushes the 8-byte return address (`0x401005`, the address of `mov rax, 60` in `_start`) onto the stack, decrementing `RSP` by **8 bytes** (`0x7fffffffdb20` $\rightarrow$ `0x7fffffffdb18`), and jumps to `work`.

2. **Function Prologue (`work`):**
   * **`push rbp`:** Saves the caller's base pointer (`0x0`) onto the stack, decrementing `RSP` by **8 bytes** (`0x7fffffffdb18` $\rightarrow$ `0x7fffffffdb10`).
   * **`mov rbp, rsp`:** Sets `RBP = 0x7fffffffdb10` to anchor the base of `work`'s stack frame.
   * **`sub rsp, 16`:** Allocates **16 bytes** of local variable space on the stack by subtracting `16` from `RSP` (`0x7fffffffdb10` $\rightarrow$ `0x7fffffffdb00`).

3. **Using the Stack Frame for Local Variables:**
   * **`mov qword [rbp - 8], 100`:** Writes `100` (`0x64`) into the first 8-byte local slot at `0x7fffffffdb08`.
   * **`mov qword [rbp - 16], 200`:** Writes `200` (`0xc8`) into the second 8-byte local slot at `0x7fffffffdb00`.
   * **`mov rax, [rbp - 8]` & `mov rbx, [rbp - 16]`:** Reads those stack-allocated local variables back into `RAX` (`100`) and `RBX` (`200`) without altering `RSP` or `RBP`.

4. **Function Epilogue & Return:**
   * **`mov rsp, rbp`:** Deallocates the 16 bytes of local variable space by resetting `RSP` back to `RBP` (`0x7fffffffdb00` $\rightarrow$ `0x7fffffffdb10`).
   * **`pop rbp`:** Restores the caller's saved base pointer (`0x0`) into `RBP` and increments `RSP` by **8 bytes** (`0x7fffffffdb10` $\rightarrow$ `0x7fffffffdb18`).
   * **`ret`:** Pops the 8-byte return address (`0x401005`) from `[RSP]` into `RIP` and increments `RSP` by **8 bytes** (`0x7fffffffdb18` $\rightarrow$ `0x7fffffffdb20`), returning control to `_start`.

5. **Program Termination:**
   * Back in `_start` with `RSP` restored to `0x7fffffffdb20`, the program invokes `sys_exit` (`rax = 60`, `rdi = 0`) and exits cleanly with status `0`.

## 3. Execution & `gdb` Observations

### Build & Run Commands
```bash
nasm -f elf64 -g -F dwarf 02.asm -o 02.o
ld 02.o -o 02_program
./02_program
echo $?   # Output: 0