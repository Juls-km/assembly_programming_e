# Stack Analysis: `03.asm`

## 1. Program Overview
`03.asm` is a 64-bit x86-64 assembly program that demonstrates **implicit stack usage** via procedure calls (`call` and `ret`). Even though the source code contains no explicit `push` or `pop` instructions, the CPU automatically uses the stack to save and restore the instruction return address when calling and returning from `calculate`.

## 2. Step-by-Step Stack Execution Flow

1. **Program Entry (`_start`):**
   * Execution begins with `RSP = 0x7fffffffdb20` and `RAX = 0x0`.
   * `mov rax, 10` loads `10` (`0xa`) into `RAX` without modifying `RSP`.

2. **Procedure Call (`call calculate`):**
   * **`call calculate`:** Implicitly pushes the 8-byte return address (`0x40100a`, the memory address of the `after_call` label) onto the stack, decrementing `RSP` by **8 bytes** (`0x7fffffffdb20` $\rightarrow$ `0x7fffffffdb18`), and jumps to `calculate`.

3. **Inside `calculate`:**
   * **`add rax, 5`:** Adds `5` directly to `RAX` (`10 + 5 = 15` / `0xf`). Because `calculate` uses no local stack variables and pushes no registers, `RSP` remains unchanged at `0x7fffffffdb18`, with the return address `0x40100a` still sitting at `[RSP]`.

4. **Returning to Caller (`ret`):**
   * **`ret`:** Implicitly pops the 8-byte return address (`0x40100a`) from `[RSP]` into the instruction pointer `RIP` and increments `RSP` by **8 bytes** (`0x7fffffffdb18` $\rightarrow$ `0x7fffffffdb20`), transferring execution back to `after_call`.

5. **Continuation & Termination (`after_call`):**
   * Execution resumes at `after_call` (`nop`) with `RAX = 15` (`0xf`) and `RSP` restored to `0x7fffffffdb20`.
   * The program then sets `RAX = 60` (`sys_exit`), zeroes `RDI` (`exit code 0`), and invokes `syscall` to exit cleanly.

## 3. Execution & `gdb` Observations

### Build & Run Commands
```bash
nasm -f elf64 -g -F dwarf 03.asm -o 03.o
ld 03.o -o 03_program
./03_program
echo $?   # Output: 0