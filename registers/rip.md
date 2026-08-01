**`RIP` and `RBP`** 
These are two of the most important registers in x86-64.

---
    
# 1. RIP — Instruction Pointer ⭐⭐⭐⭐⭐

`RIP` (**Register Instruction Pointer**) tells the CPU:

> **"What instruction should I execute next?"**

Think of it as the **line number** in your program.

## Example

Suppose your program is:

```asm
0x401000: mov eax, 5
0x401005: add eax, 2
0x401008: ret
```

Execution looks like this:

```
RIP = 0x401000
↓
mov eax, 5

RIP = 0x401005
↓
add eax, 2

RIP = 0x401008
↓
ret
```

The CPU automatically updates `RIP` after each instruction.

---

## What changes RIP?

### Normal execution

```
mov eax, 5
```

RIP simply moves to the next instruction.

---

### Jump

```asm
jmp target
```

Instead of going to the next line:

```
RIP
 ↓
jmp target
```

After execution:

```
RIP
 ↓
target:
```

---

### Call

```asm
call printf
```

The CPU:

1. Saves the current RIP onto the stack (the return address).
2. Sets RIP to the address of `printf`.

```
Before:

RIP
 ↓
call printf

After:

RIP
 ↓
printf()
```

---

### Return

```asm
ret
```

`ret` pops the saved return address from the stack and puts it back into `RIP`.

```
Stack
---------------
0x401020   ← return address
---------------

ret

↓

RIP = 0x401020
```

---

# RIP Summary

```
RIP
│
├── Points to next instruction
├── Changes after every instruction
├── Modified by jmp
├── Modified by call
└── Restored by ret
```

---

# 2. RBP — Base Pointer ⭐⭐⭐⭐⭐

`RBP` (**Register Base Pointer**) marks the **base of the current function's stack frame**.

It provides a stable reference point for accessing:

* local variables
* function arguments
* saved registers

Unlike `RSP`, which changes frequently as values are pushed and popped, `RBP` usually stays fixed during a function.

---

## Typical function

```asm
push rbp
mov rbp, rsp
sub rsp, 32
```

Let's go step by step.

### Step 1

```
push rbp
```

Save the caller's `RBP`.

```
Stack

Old RBP
```

---

### Step 2

```
mov rbp, rsp
```

Now:

```
RBP
 ↓
+------------------+
| Old RBP          |
+------------------+
```

This becomes the base of the new stack frame.

---

### Step 3

```
sub rsp, 32
```

Reserve space for local variables.

```
High Address

+----------------+
| Function arg   | ← rbp+16
+----------------+
| Return address | ← rbp+8
+----------------+
| Old RBP        | ← rbp
+----------------+
| Local var #1   | ← rbp-8
+----------------+
| Local var #2   | ← rbp-16
+----------------+
| Local var #3   |
+----------------+
| Local var #4   |
+----------------+
        ↑
       RSP

Low Address
```

---

## Accessing variables

Compiler-generated assembly often uses `RBP` offsets.

```asm
mov eax, [rbp-4]
```

Meaning:

```
Load the integer stored
4 bytes below RBP.
```

---

Another example:

```asm
mov [rbp-8], rax
```

Store `RAX` in a local variable.

---

Function arguments may be accessed like this (especially after being spilled to the stack):

```asm
mov eax, [rbp+16]
```

This accesses data above the base pointer in the stack frame.

---

# Function ending

```asm
leave
ret
```

`leave` is equivalent to:

```asm
mov rsp, rbp
pop rbp
```

Then:

```
ret
```

restores `RIP` from the stack and returns to the caller.

---

# RBP vs RSP

| Register | Purpose                         | Changes Often? |
| -------- | ------------------------------- | -------------- |
| `RSP`    | Top of the stack                | ✅ Yes          |
| `RBP`    | Base of the current stack frame | ❌ Usually no   |

Example:

```
push rax
push rbx
pop rbx
push rcx
```

`RSP` moves after every push/pop.

`RBP` remains the same throughout the function (unless the compiler omits the frame pointer for optimization).

---

# Visual Summary

```
                    CPU
                 +--------+
                 |  RIP   | ---> Next instruction to execute
                 +--------+

                     |
                     v

0x401000  mov eax,5
0x401005  add eax,2
0x401008  call foo
0x40100D  ret
```

```
                  Stack

High Address
+----------------------+
| Function arguments   |  rbp + offset
+----------------------+
| Return Address       |
+----------------------+
| Saved RBP            | ← RBP
+----------------------+
| Local Variable #1    | ← rbp - 8
+----------------------+
| Local Variable #2    | ← rbp - 16
+----------------------+
| Local Variable #3    |
+----------------------+
            ↑
           RSP
Low Address
```

### In reverse engineering and binary exploitation

* **`RIP`** tells you **where the program is executing**. In a debugger like GDB, `info registers rip` or simply `i r rip` shows the current instruction address.
* **`RBP`** helps you **understand a function's stack layout**, making it easier to identify local variables and saved values.
* **`RSP`** tells you the **current top of the stack**, which is essential for following function calls, returns, and many exploitation techniques.

These three registers—**`RIP`**, **`RBP`**, and **`RSP`**—are the core registers you'll constantly inspect when debugging or reversing x86-64 programs.
