# x86 Register Review (32-bit)

## What is a Register?

A **register** is a very small, extremely fast storage location **inside the CPU**.

Unlike RAM, registers are directly accessed by the processor every instruction.

Think of them as the CPU's workspace.

```text
CPU

+--------------------------------------+
|                                      |
|  EAX   EBX   ECX   EDX               |
|                                      |
|  ESP   EBP   EIP                     |
|                                      |
+--------------------------------------+

            │

            ▼

          Memory (RAM)
```

Registers hold things like:

* numbers
* memory addresses
* pointers
* function arguments
* return values

---

# General Purpose Registers

There are four classic general-purpose registers.

```
EAX
EBX
ECX
EDX
```

Historically, each had a preferred use, although modern compilers use them more flexibly.

---

# EAX — Accumulator

**EAX** is the most important general-purpose register.

It is commonly used for:

* arithmetic operations
* function return values
* system call numbers (Linux)

Example:

```c
int add()
{
    return 5;
}
```

Assembly:

```asm
mov eax, 5
ret
```

When the function returns:

```
EAX = 5
```

---

## Why Exploit Developers Care

Suppose GDB shows:

```text
EAX = 0x41414141
```

This means your input has overwritten EAX.

That's useful because it proves you control part of the program state.

---

# EBX — Base Register

Historically:

* base pointer for data
* general-purpose storage

Example:

```asm
mov ebx, eax
```

Copies EAX into EBX.

Today, it's often just another register the compiler uses.

---

# ECX — Counter Register

Traditionally used for loops.

Example:

```asm
mov ecx, 10

loop_start:

...

loop loop_start
```

The `loop` instruction automatically decrements ECX until it reaches zero.

You'll also see ECX used for:

* counters
* string operations
* function arguments in some calling conventions

---

# EDX — Data Register

Often paired with EAX.

Example:

Multiplication:

```asm
mul ebx
```

Result:

```
High 32 bits → EDX

Low 32 bits → EAX
```

Division:

```
Dividend

EDX:EAX
```

Quotient:

```
EAX
```

Remainder:

```
EDX
```

---

# ESP — Stack Pointer

Probably the most important register for exploitation.

ESP always points to the **top of the stack**.

Example:

```
Stack

High Address

----------------

Function arguments

Return Address

Saved EBP

↓

ESP

Local Variables

Low Address
```

When you execute:

```asm
push eax
```

ESP changes.

```text
Before

ESP

0xffffd020
```

After

```asm
push eax
```

```text
ESP

0xffffd01c
```

The stack grows **downward**.

---

# Why ESP Matters

Buffer overflows often overwrite memory **near ESP**.

You'll constantly inspect it:

```gdb
info registers
```

or

```gdb
x/20wx $esp
```

---

# EBP — Base Pointer (Frame Pointer)

EBP points to the beginning of the current function's stack frame.

Typical function:

```asm
push ebp

mov ebp, esp

sub esp, 0x40
```

Visualization:

```
High Address

Arguments

Return Address

Saved EBP  ← EBP

---------------

Local Variables

---------------

ESP

Low Address
```

Local variables use offsets from EBP.

Example:

```asm
mov eax, [ebp-4]
```

means

```
Load local variable
```

Arguments:

```asm
mov eax, [ebp+8]
```

means

```
Load first function argument
```

---

# Why EBP Matters

During a stack overflow:

```
Buffer

↓

Saved EBP

↓

Return Address
```

You often overwrite:

1. Local buffer
2. Saved EBP
3. Return address

Understanding EBP makes it much easier to calculate the correct offset to EIP.

---

# EIP — Instruction Pointer

EIP tells the CPU:

> **What instruction should execute next?**

Example:

```
EIP

0x08048464
```

CPU executes:

```
0x08048464
```

Then automatically moves to:

```
0x08048467
```

(or whatever the next instruction is)

---

# Why EIP Is Everything

Consider a vulnerable program:

```
Buffer

↓

Saved EBP

↓

Saved EIP
```

If you overwrite EIP:

```
EIP = winner()
```

The CPU immediately begins executing the `winner()` function.

That's exactly what you did in your **Ret2Win** exercises.

For example:

```text
Before

EIP

0x08048590
```

Overflow:

```text
EIP

0x08048484
```

Now the program jumps directly to:

```
winner()
```

This is the essence of many classic buffer overflow exploits.

---

# Register Relationships

```
CPU Registers

+----------------------+

EAX

Return values

Arithmetic

----------------------

EBX

General storage

----------------------

ECX

Loop counter

----------------------

EDX

Arithmetic

Multiply/Divide

----------------------

ESP

Top of stack

----------------------

EBP

Current stack frame

----------------------

EIP

Next instruction

+----------------------+
```

---

# What You'll See in GDB

Run:

```gdb
info registers
```

Example:

```text
eax            0x00000005

ebx            0x0804a000

ecx            0xffffd050

edx            0x00000000

esp            0xffffd020

ebp            0xffffd068

eip            0x08048484
```

Interpretation:

| Register | Meaning                              |
| -------- | ------------------------------------ |
| EAX      | Function returned `5`                |
| EBX      | Holds a memory address               |
| ECX      | Counter or temporary value           |
| EDX      | Additional arithmetic data           |
| ESP      | Current top of the stack             |
| EBP      | Base of the current stack frame      |
| EIP      | Instruction currently being executed |

---

# Exploitation Perspective

When debugging a crash, these are the registers you'll check first:

```
EIP
```

Did we control execution?

```
ESP
```

Where is our payload?

```
EBP
```

Did we overwrite the stack frame?

```
EAX
```

Did our input end up here?

Many exploit development sessions begin with:

```gdb
info registers
```

followed by:

```gdb
x/32wx $esp
```

to inspect the stack around the current stack pointer.

---

# Memory Layout During a Function Call

```text
Higher Memory Addresses
+----------------------------+
| Function Arguments         | ← [EBP+8], [EBP+12], ...
+----------------------------+
| Return Address             |
+----------------------------+
| Saved EBP                  | ← EBP
+----------------------------+
| Local Variables            |
| char buffer[64];           |
| int x;                     |
+----------------------------+
|                            | ← ESP (moves as values are pushed/popped)
+----------------------------+
Lower Memory Addresses
```

---

# Quick Cheat Sheet

| Register | Full Name           | Primary Purpose                            |
| -------- | ------------------- | ------------------------------------------ |
| **EAX**  | Accumulator         | Return values, arithmetic                  |
| **EBX**  | Base                | General-purpose storage                    |
| **ECX**  | Counter             | Loops, counters, string operations         |
| **EDX**  | Data                | Arithmetic, multiplication/division        |
| **ESP**  | Stack Pointer       | Points to the top of the stack             |
| **EBP**  | Base Pointer        | Points to the current stack frame          |
| **EIP**  | Instruction Pointer | Address of the next instruction to execute |

---

## Key Takeaways

* **EAX** usually contains a function's return value.
* **EBX**, **ECX**, and **EDX** are general-purpose registers with traditional roles in storage, counting, and arithmetic.
* **ESP** tracks the top of the stack and changes with every `push` and `pop`.
* **EBP** provides a stable reference point for accessing local variables and function arguments.
* **EIP** controls program execution; overwriting it changes where the CPU executes next.
* In exploit development, **EIP** is the primary target, while **ESP** helps locate your payload and **EBP** helps understand the stack frame.
