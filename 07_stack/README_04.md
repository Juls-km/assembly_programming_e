# Stack Analysis: `04.asm`

## 1. Program Overview
`04.asm` is a **32-bit x86** (`BITS 32`) assembly program that demonstrates how a 32-bit function call (`call`/`ret`) and **stack frame** (`EBP` and `ESP`) operate in memory. It mirrors `02.asm` but uses 32-bit registers and 4-byte (`dword`) stack slots rather than 64-bit 8-byte slots.

## 2. Step-by-Step Stack Execution Flow

1. **Program Entry (`_start`):**
   * Execution begins in 32-bit mode with `ESP = 0xffffcc90` and `EBP = 0x0`.
   * **`call work`:** Implicitly pushes the **4-byte** return address (`0x08049005`, pointing to `mov eax, 1` in `_start`) onto the stack, decrementing `ESP` by **4 bytes** (`0xffffcc90` $\rightarrow$ `0xffffcc8c`), and jumps to `work`.

2. **32-Bit Function Prologue (`work`):**
   * **`push ebp`:** Saves the caller's 32-bit base pointer (`0x0`) onto the stack, decrementing `ESP` by **4 bytes** (`0xffffcc8c` $\rightarrow$ `0xffffcc88`).
   * **`mov ebp, esp`:** Sets `EBP = 0xffffcc88` to establish the base of `work`'s stack frame.
   * **`sub esp, 8`:** Allocates **8 bytes** of local variable space on the stack by subtracting `8` from `ESP` (`0xffffcc88` $\rightarrow$ `0xffffcc80`).

3. **Using the Stack Frame for Local Variables:**
   * **`mov dword [ebp - 4], 100`:** Stores `100` (`0x64`) in the first 4-byte local slot at `0xffffcc84`.
   * **`mov dword [ebp - 8], 200`:** Stores `200` (`0xc8`) in the second 4-byte local slot at `0xffffcc80`.
   * **`mov eax, [ebp - 4]` & `mov ebx, [ebp - 8]`:** Loads the two 32-bit local variables from the stack frame into `EAX` (`100`) and `EBX` (`200`) without altering `ESP` or `EBP`.

4. **Function Epilogue & Return:**
   * **`mov esp, ebp`:** Deallocates the 8 bytes of local variable space by resetting `ESP` to `EBP` (`0xffffcc80` $\rightarrow$ `0xffffcc88`).
   * **`pop ebp`:** Pops the saved 4-byte base pointer (`0x0`) back into `EBP` and increments `ESP` by **4 bytes** (`0xffffcc88` $\rightarrow$ `0xffffcc8c`).
   * **`ret`:** Pops the 4-byte return address (`0x08049005`) from `[ESP]` into `EIP` and increments `ESP` by **4 bytes** (`0xffffcc8c` $\rightarrow$ `0xffffcc90`), returning execution to `_start`.

5. **Program Termination:**
   * Back in `_start` with `ESP` restored to `0xffffcc90`, the program invokes the 32-bit Linux exit interrupt (`eax = 1` for `sys_exit`, `ebx = 0`, `int 0x80`) and exits cleanly with status `0`.

## 3. Execution & `gdb` Observations

### Build & Run Commands
```bash
nasm -f elf32 -g -F dwarf 04.asm -o 04.o
ld -m elf_i386 04.o -o 04_program
./04_program
echo $?   # Output: 0