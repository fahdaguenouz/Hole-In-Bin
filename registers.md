Absolutely. Since you're learning reverse engineering and binary exploitation, here's a cheat sheet you can keep nearby. It includes the register hierarchy, its usual purpose, and a simple example.

---
##
# General Purpose Registers

| 64-bit | 32-bit | 16-bit | 8-bit | Common Purpose |
|--------|--------|--------|--------|----------------|
| **RAX** | EAX | AX | AL | Return value, syscall number, arithmetic |
| **RBX** | EBX | BX | BL | General-purpose (callee-saved) |
| **RCX** | ECX | CX | CL | Loop counter, 4th function argument |
| **RDX** | EDX | DX | DL | 3rd function argument |
| **RSI** | ESI | SI | SIL | 2nd function argument |
| **RDI** | EDI | DI | DIL | 1st function argument |
| **RSP** | ESP | SP | SPL | Stack Pointer |
| **RBP** | EBP | BP | BPL | Base Pointer |
| **R8** | R8D | R8W | R8B | 5th function argument |
| **R9** | R9D | R9W | R9B | 6th function argument |
| **R10** | R10D | R10W | R10B | 4th syscall argument |
| **R11** | R11D | R11W | R11B | Temporary register |
| **R12** | R12D | R12W | R12B | General-purpose |
| **R13** | R13D | R13W | R13B | General-purpose |
| **R14** | R14D | R14W | R14B | General-purpose |
| **R15** | R15D | R15W | R15B | General-purpose |

---

##

# x86-64 General Purpose Registers Cheat Sheet

## 1. RAX — Accumulator

```text
64 bits:  +-----------------------------------------------+
          |                    RAX                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |          EAX          |
                    |----------|
16 bits:            |    AX    |
                    |----|
8 bits:             | AL |
```

### Common uses

* Return value from functions
* System call number
* Arithmetic operations

### Example

```asm
mov rax, 60      ; exit syscall
syscall
```

or

```asm
add rax, 5
```

---

## 2. RBX — Base Register

```text
64 bits:  +-----------------------------------------------+
          |                    RBX                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |          EBX          |
                    |----------|
16 bits:            |    BX    |
                    |----|
8 bits:             | BL |
```

### Common uses

* General-purpose storage
* Often preserved across function calls

### Example

```asm
mov rbx, 100
```

---

## 3. RCX — Counter Register

```text
64 bits:  +-----------------------------------------------+
          |                    RCX                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |          ECX          |
                    |----------|
16 bits:            |    CX    |
                    |----|
8 bits:             | CL |
```

### Common uses

* Loop counter
* 4th function argument

### Example

```asm
mov rcx, 10
loop_start:
dec rcx
jnz loop_start
```

---

## 4. RDX — Data Register

```text
64 bits:  +-----------------------------------------------+
          |                    RDX                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |          EDX          |
                    |----------|
16 bits:            |    DX    |
                    |----|
8 bits:             | DL |
```

### Common uses

* 3rd function argument
* Multiplication/division

### Example

```asm
mov rdx, 100
```

---

## 5. RSI — Source Index

```text
64 bits:  +-----------------------------------------------+
          |                    RSI                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |          ESI          |
                    |----------|
16 bits:            |    SI    |
                    |----|
8 bits:             | SIL |
```

### Common uses

* 2nd function argument
* Source pointer for memory operations

### Example

```asm
mov rsi, message
```

or

```asm
mov rdi, rsi
```

---

## 6. RDI — Destination Index

```text
64 bits:  +-----------------------------------------------+
          |                    RDI                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |          EDI          |
                    |----------|
16 bits:            |    DI    |
                    |----|
8 bits:             | DIL |
```

### Common uses

* 1st function argument
* Destination pointer

### Example

```asm
mov rdi, 42
```

or

```asm
mov rdi, rsi
```

---

## 7. RBP — Base Pointer

```text
64 bits:  +-----------------------------------------------+
          |                    RBP                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |          EBP          |
                    |----------|
16 bits:            |    BP    |
                    |----|
8 bits:             | BPL |
```

### Common uses

* Base of the current stack frame
* Access local variables

### Example

```asm
push rbp
mov rbp, rsp
```

---

## 8. RSP — Stack Pointer

```text
64 bits:  +-----------------------------------------------+
          |                    RSP                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |          ESP          |
                    |----------|
16 bits:            |    SP    |
                    |----|
8 bits:             | SPL |
```

### Common uses

* Points to the top of the stack

### Example

```asm
push rax
pop rax
```

---

## 9. R8

```text
64 bits:  +-----------------------------------------------+
          |                     R8                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |         R8D           |
                    |----------|
16 bits:            |   R8W    |
                    |----|
8 bits:             | R8B |
```

### Common uses

* 5th function argument

Example

```asm
mov r8, 123
```

---

## 10. R9

```text
64 bits:  +-----------------------------------------------+
          |                     R9                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |         R9D           |
                    |----------|
16 bits:            |   R9W    |
                    |----|
8 bits:             | R9B |
```

### Common uses

* 6th function argument

---

## 11. R10

```text
64 bits:  +-----------------------------------------------+
          |                    R10                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |        R10D           |
                    |----------|
16 bits:            |   R10W   |
                    |----|
8 bits:             | R10B |
```

### Common uses

* 4th syscall argument
* Temporary register

---

## 12. R11

```text
64 bits:  +-----------------------------------------------+
          |                    R11                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |        R11D           |
                    |----------|
16 bits:            |   R11W   |
                    |----|
8 bits:             | R11B |
```

### Common uses

* Temporary register
* Modified by `syscall`

---

## 13. R12

```text
64 bits:  +-----------------------------------------------+
          |                    R12                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |        R12D           |
                    |----------|
16 bits:            |   R12W   |
                    |----|
8 bits:             | R12B |
```

### Common uses

* General-purpose register

---

## 14. R13

```text
64 bits:  +-----------------------------------------------+
          |                    R13                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |        R13D           |
                    |----------|
16 bits:            |   R13W   |
                    |----|
8 bits:             | R13B |
```

### Common uses

* General-purpose register

---

## 15. R14

```text
64 bits:  +-----------------------------------------------+
          |                    R14                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |        R14D           |
                    |----------|
16 bits:            |   R14W   |
                    |----|
8 bits:             | R14B |
```

### Common uses

* General-purpose register

---

## 16. R15

```text
64 bits:  +-----------------------------------------------+
          |                    R15                        |
          +-----------------------------------------------+
                    |-----------------------|
32 bits:            |        R15D           |
                    |----------|
16 bits:            |   R15W   |
                    |----|
8 bits:             | R15B |
```

### Common uses

* General-purpose register

---

# The two tables you should memorize

### Function arguments (System V ABI)

| Argument | Register |
| -------- | -------- |
| 1        | `rdi`    |
| 2        | `rsi`    |
| 3        | `rdx`    |
| 4        | `rcx`    |
| 5        | `r8`     |
| 6        | `r9`     |

---

### Linux `syscall` arguments

| Purpose        | Register |
| -------------- | -------- |
| Syscall number | `rax`    |
| Argument 1     | `rdi`    |
| Argument 2     | `rsi`    |
| Argument 3     | `rdx`    |
| Argument 4     | `r10`    |
| Argument 5     | `r8`     |
| Argument 6     | `r9`     |

---
##

RAX → Syscall Number / Return Value

RDI → Argument #1

RSI → Argument #2

RDX → Argument #3

RSP → Stack Pointer

RBP → Base Pointer
```

---

# Memory Trick

```text
Arguments (Functions)

1 → RDI
2 → RSI
3 → RDX
4 → RCX
5 → R8
6 → R9

Return → RAX
```

```text
Arguments (Syscalls)

RAX → Which syscall?

RDI → Arg1

RSI → Arg2

RDX → Arg3

R10 → Arg4

R8  → Arg5

R9  → Arg6
```

---

# Typical Program Flow

```text
          Program Starts
                 │
                 ▼
         Put syscall number
            into RAX
                 │
                 ▼
      Put arguments into
RDI → RSI → RDX → R10 → R8 → R9
                 │
                 ▼
             syscall
                 │
                 ▼
      Kernel executes request
                 │
                 ▼
      Return value stored in RAX
```