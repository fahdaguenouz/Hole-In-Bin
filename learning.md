The **Hole-In-Bin** project isn't just about making a program print the success message. Each exercise is designed to teach a specific binary exploitation concept. If you understand the concepts behind each level, you'll be able to explain them during the audit instead of just repeating the solution.

Here's the learning roadmap.

| Exercise | Main Concept                      | What You Should Learn                                                                                                                           |
| -------- | --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **ex00** | Basic Stack Buffer Overflow       | How overflowing a buffer changes an adjacent variable. Learn the stack layout, `gets()`, and why it is unsafe.                                  |
| **ex01** | Buffer Overflow + Specific Value  | How to overwrite a variable with an exact integer value. Learn hexadecimal, ASCII, little-endian representation, and payload construction.      |
| **ex02** | Environment Variable Overflow     | Same overflow technique, but input comes from `getenv()` instead of command-line arguments. Learn that any untrusted input can be dangerous.    |
| **ex03** | Function Pointer Overwrite        | Learn what a function pointer is and how overwriting it changes program execution.                                                              |
| **ex04** | Return Address Overwrite          | Learn how the stack frame works and how changing the return address hijacks execution.                                                          |
| **ex05** | Shellcode                         | Learn what shellcode is, how it is placed in memory, and how execution jumps to it.                                                             |
| **ex06** | Return-to-libc                    | Learn how to execute existing library functions (such as `system`) instead of injecting shellcode, especially when the stack is non-executable. |
| **ex07** | Format String Vulnerability       | Learn how uncontrolled format strings (`printf(user_input)`) can leak memory or overwrite values.                                               |
| **ex08** | Heap Exploitation                 | Understand the heap, dynamic allocation (`malloc`/`free`), and common heap corruption vulnerabilities.                                          |
| **ex09** | Return-Oriented Programming (ROP) | Learn to chain existing instructions ("gadgets") to execute code when shellcode cannot be used.                                                 |
| **ex10** | Advanced Exploitation             | Usually combines several techniques together (depends on the challenge).                                                                        |
| **ex11** | Final Challenge                   | Apply everything learned in the previous exercises to solve a more complex binary.                                                              |

---

# Core Skills You Should Learn

By the end of the project, you should be comfortable with:

### 1. Linux ELF Binaries

* 32-bit vs 64-bit executables
* ELF structure
* Sections (`.text`, `.data`, `.bss`, `.plt`, `.got`)
* Entry point

---

### 2. Assembly Language

Understand instructions such as:

* `mov`
* `push`
* `pop`
* `lea`
* `call`
* `ret`
* `cmp`
* `test`
* `jmp`
* `je`
* `jne`
* `leave`

You should be able to explain what each instruction does and how it affects registers, memory, and control flow.

---

### 3. CPU Registers

For x86 (32-bit):

* `EAX`
* `EBX`
* `ECX`
* `EDX`
* `ESP`
* `EBP`
* `EIP`

Know what each register is commonly used for and how it changes during function calls.

---

### 4. Stack Memory

Understand:

* Function prologue and epilogue
* Local variables
* Saved frame pointer
* Return address
* Stack growth direction

A typical stack frame looks like:

```text
Return Address
Saved EBP
Local Variables
Buffer
```

---

### 5. Buffer Overflows

Learn:

* Why `gets()` is unsafe
* Why `strcpy()` is unsafe
* How overflows happen
* How adjacent variables are overwritten
* How return addresses are overwritten

---

### 6. Endianness

Understand how multi-byte values are stored in memory.

For example:

```text
0x61626364
```

is stored in memory as:

```text
64 63 62 61
```

on little-endian systems.

---

### 7. Payload Construction

Know how to build payloads like:

```python
b"A"*64 + b"\x64\x63\x62\x61"
```

and why each part is needed.

---

### 8. Reverse Engineering

Using:

* `strings`
* `readelf`
* `objdump`
* `gdb`
* `radare2`

to understand how a program works without its source code.

---

### 9. Debugging

Use GDB to:

* Set breakpoints
* Step through instructions
* Inspect registers
* Examine memory
* Understand program execution

---

### 10. Security Protections

Know what these protections do:

* Stack Canary
* NX (Non-Executable Stack)
* PIE (Position Independent Executable)
* ASLR (Address Space Layout Randomization)
* RELRO

Understand how they make exploitation harder and why many learning binaries disable them.

---

## For the Audit

The auditors are likely to ask questions such as:

* Why is `gets()` dangerous?
* Why is `strcpy()` unsafe?
* What does `cmp` do?
* What is the purpose of `lea`?
* Why does the overflow reach the `modified` variable first?
* Why did you use `64` bytes of padding?
* Why are the payload bytes reversed?
* What is little-endian format?
* What is stored in the return address?
* What is the difference between the stack and the heap?
* How would you fix this vulnerability?

If you can answer those questions confidently while demonstrating the exploit, you'll show that you understand the underlying concepts rather than just the solution.
