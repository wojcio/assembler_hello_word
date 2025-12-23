Writing "Hello, World!" in x86 assembly depends heavily on the operating system you are using (Linux vs. Windows) and the syntax flavor (Intel vs. AT&T).

The most common way to learn x86 assembly today is using **Linux** with the **NASM (Netwide Assembler)** syntax, which uses the Intel format. It is cleaner and easier to read.

Here is the complete code for a 64-bit Linux system.

### The Code (`hello.asm`)

```nasm
section .data
    msg db 'Hello, World!', 0xa   ; The string (0xA is newline)
    len equ $ - msg               ; Calculate the length of the string

section .text
    global _start                 # Declare the entry point for the linker

_start:
    ; --- SYSCALL: write ---
    mov rax, 1                    ; System call number for 'write' (64-bit Linux)
    mov rdi, 1                    ; File descriptor: 1 is Standard Output (stdout)
    mov rsi, msg                  ; Address of the string to print
    mov rdx, len                  ; The number of bytes to write
    syscall                       ; Invoke the kernel

    ; --- SYSCALL: exit ---
    mov rax, 60                   ; System call number for 'exit' (64-bit Linux)
    xor rdi, rdi                  ; Status code 0 (no errors)
    syscall                       ; Invoke the kernel to exit the program
```

---

### How it Works

#### 1. The Data Section (`.data`)
This is where we store static data.
*   **`db`**: Define Byte. We define the string "Hello, World!".
*   **`0xa`**: This is the hexadecimal code for a "New Line" (Line Feed), so the cursor moves to the next line after printing.
*   **`len`**: We calculate the length of the string so the computer knows how many characters to print. `$` represents the current memory address, and `msg` is the start of the string. Subtracting them gives us the length (13 bytes).

#### 2. The Text Section (`.text`)
This is where the code lives.

**The "Write" Logic:**
In 64-bit Linux, you don't print directly to the screen. You ask the Operating System (Kernel) to do it for you using a "System Call."
1.  **`mov rax, 1`**: We put the number `1` into the `rax` register. In Linux, system call #1 corresponds to **sys_write**.
2.  **`mov rdi, 1`**: The first argument for `sys_write` is the file descriptor. `1` stands for Standard Output (the screen).
3.  **`mov rsi, msg`**: The second argument is a pointer to the memory address of our message.
4.  **`mov rdx, len`**: The third argument is the count of how many bytes to write.
5.  **`syscall`**: This is a special instruction that switches the processor into "Kernel Mode" to execute the write operation.

**The Exit Logic:**
Once the message is printed, we must manually tell the OS to stop the program. If we don't, the computer might keep running into random memory and crash.
1.  **`mov rax, 60`**: System call #60 is **sys_exit**.
2.  **`xor rdi, rdi`**: This sets the `rdi` register to `0`. It is a trick using XOR (Exclusive OR) that sets the value to zero efficiently. This tells the OS "No errors occurred."
3.  **`syscall`**: The program ends.

---

### How to Run It

To run this code, you need the **NASM** assembler and **LD** linker (usually included with GCC).

1.  **Save the file:** Save the code above as `hello.asm`.
2.  **Assemble:**
    ```bash
    nasm -f elf64 hello.asm -o hello.o
    ```
3.  **Link:**
    ```bash
    ld hello.o -o hello
    ```
4.  **Run:**
    ```bash
    ./hello
    ```

**Output:**
```text
Hello, World!
```
