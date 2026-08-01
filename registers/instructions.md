the **assembly instructions** (also called **opcodes** or **mnemonics**). You **do not** need to learn the entire x86-64 instruction set for cybersecurity and reverse engineering. About **20–30 instructions** cover most of what you'll see.


# x86-64 Instructions Roadmap

## 1. Data Movement ⭐⭐⭐⭐⭐ (Start Here)

These move data around.

| Instruction | Purpose        | Example            | Meaning                           |
| ----------- | -------------- | ------------------ | --------------------------------- |
| `mov`       | Copy data      | `mov rax, rbx`     | RAX = RBX                         |
| `lea`       | Load address   | `lea rax, [rbp-8]` | Put the address into RAX          |
| `push`      | Push to stack  | `push rax`         | Save RAX on the stack             |
| `pop`       | Pop from stack | `pop rbx`          | Restore value from stack into RBX |
| `xchg`      | Swap values    | `xchg rax, rbx`    | Exchange RAX and RBX              |

---

## 2. Arithmetic ⭐⭐⭐⭐⭐

| Instruction | Purpose         | Example         |
| ----------- | --------------- | --------------- |
| `add`       | Addition        | `add rax, 5`    |
| `sub`       | Subtraction     | `sub rsp, 32`   |
| `inc`       | Increment       | `inc rcx`       |
| `dec`       | Decrement       | `dec rcx`       |
| `imul`      | Signed multiply | `imul rax, rbx` |
| `idiv`      | Signed divide   | `idiv rcx`      |
| `neg`       | Negate          | `neg rax`       |

---

## 3. Logic ⭐⭐⭐⭐☆

Used constantly in malware and reverse engineering.

| Instruction | Purpose                    |
| ----------- | -------------------------- |
| `and`       | Bitwise AND                |
| `or`        | Bitwise OR                 |
| `xor`       | Bitwise XOR                |
| `not`       | Invert bits                |
| `test`      | AND without storing result |

Example:

```asm
xor eax, eax
```

Equivalent to

```c
eax = 0;
```

This is one of the most common instructions you'll ever see.

---

## 4. Comparison ⭐⭐⭐⭐⭐

| Instruction | Purpose            |
| ----------- | ------------------ |
| `cmp`       | Compare two values |
| `test`      | Test bits          |

Example

```asm
cmp eax, 5
je equal
```

Equivalent

```c
if (eax == 5)
```

---

## 5. Jumps ⭐⭐⭐⭐⭐

These are the assembly version of `if`.

| Instruction | Meaning           |
| ----------- | ----------------- |
| `jmp`       | Always jump       |
| `je`        | Jump if equal     |
| `jne`       | Jump if not equal |
| `jg`        | Greater           |
| `jl`        | Less              |
| `jge`       | Greater or equal  |
| `jle`       | Less or equal     |

Example

```asm
cmp eax, ebx
je same
```

Equivalent

```c
if (eax == ebx)
```

---

## 6. Function Instructions ⭐⭐⭐⭐⭐

Essential for reverse engineering.

| Instruction | Purpose              |
| ----------- | -------------------- |
| `call`      | Call a function      |
| `ret`       | Return from function |

Example

```asm
call printf
```

Equivalent

```c
printf();
```

---

## 7. Shift Instructions ⭐⭐⭐☆

Common in cryptography and malware.

| Instruction | Purpose          |
| ----------- | ---------------- |
| `shl`       | Shift left       |
| `shr`       | Shift right      |
| `sar`       | Arithmetic shift |
| `rol`       | Rotate left      |
| `ror`       | Rotate right     |

---

## 8. Stack Frame Instructions ⭐⭐⭐⭐☆

You'll see these at almost every function.

```asm
push rbp
mov rbp, rsp
sub rsp, 32
```

and

```asm
leave
ret
```

These create and destroy a function's stack frame.

---

# The Most Important Instructions

If you only learn these first, you'll understand the majority of simple binaries:

```text
mov
lea
push
pop

add
sub
inc
dec

cmp
test

jmp
je
jne
jg
jl

call
ret

and
or
xor

shl
shr
```

---

# Example

```asm
mov eax, 5
add eax, 3
cmp eax, 8
je success
```

Equivalent C:

```c
int eax = 5;
eax += 3;

if (eax == 8)
    goto success;
```

---

# Suggested Learning Order

* ✅ Registers (you've completed this)
* ⬜ Memory addressing (`[]`, pointers, offsets)
* ⬜ `mov`, `lea`
* ⬜ `push`, `pop`, stack basics
* ⬜ Arithmetic (`add`, `sub`, `inc`, `dec`)
* ⬜ Comparisons (`cmp`, `test`)
* ⬜ Conditional jumps (`je`, `jne`, `jg`, `jl`, etc.)
* ⬜ `call`, `ret`, and calling conventions
* ⬜ Bitwise operations (`xor`, `and`, `or`)
* ⬜ Loops and control flow
* ⬜ Reading compiler-generated assembly from simple C programs

