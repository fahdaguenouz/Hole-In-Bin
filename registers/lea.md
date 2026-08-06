# Module — The `LEA` Instruction

## What is `LEA`?

**LEA** stands for:

> **Load Effective Address**

Its job is to **calculate an address and store that address in a register**.

**Syntax:**

```asm
lea destination, [address_expression]
```

The important part is:

> **`lea` computes the address—it does not dereference it.**

---

# `MOV` vs `LEA`

This is the key difference.

Suppose memory looks like this:

```text
Address      Value
--------     -----
0x1000       42
```

### Using `mov`

```asm
mov eax, [0x1000]
```

CPU does:

1. Go to memory address `0x1000`
2. Read the value stored there
3. Put it into `EAX`

Result:

```text
EAX = 42
```

---

### Using `lea`

```asm
lea eax, [0x1000]
```

CPU does:

1. Compute the address `0x1000`
2. Store the address itself

Result:

```text
EAX = 0x1000
```

Notice:

**No memory is read.**

---

# Visual Comparison

Memory:

```text
0x1000
+------+
| 42   |
+------+
```

### MOV

```asm
mov eax, [0x1000]
```

```text
Memory ─────► 42

EAX = 42
```

---

### LEA

```asm
lea eax, [0x1000]
```

```text
Address

0x1000

↓

EAX = 0x1000
```

---

# Why the Brackets?

The brackets mean:

```asm
[address_expression]
```

For `mov`, they mean:

> Access memory.

For `lea`, they mean:

> Compute this address.

The syntax looks similar, but the behavior is different.

---

# Example with Local Variables

Suppose a function has:

```c
char buffer[64];
```

The compiler might generate:

```asm
lea eax, [ebp-0x40]
```

What does this mean?

```
EBP = 0xffffd080
```

Calculate:

```
0xffffd080 - 0x40
```

Result:

```
EAX = 0xffffd040
```

That is the **address of `buffer`**.

This is common when passing a buffer to another function.

---

# Real Example

C code:

```c
char buffer[64];

gets(buffer);
```

Possible assembly:

```asm
lea eax, [ebp-0x40]
push eax
call gets
```

Step by step:

```
lea eax, [ebp-0x40]
```

EAX now contains:

```
&buffer
```

Then:

```asm
push eax
```

passes that pointer to `gets()`.

This is one of the most common uses of `lea`.

---

# Another Example

Suppose:

```text
EBP = 0xffffd080
```

Instruction:

```asm
lea eax, [ebp-8]
```

CPU computes:

```text
0xffffd080 - 8
```

Result:

```text
EAX = 0xffffd078
```

No memory is accessed.

---

# Compare with `MOV`

Suppose memory contains:

```text
Address        Value

0xffffd078     1234
```

Now:

```asm
mov eax, [ebp-8]
```

Result:

```text
EAX = 1234
```

Because `mov` actually reads memory.

---

# `LEA` for Arithmetic

Modern compilers also use `lea` as a fast arithmetic instruction because it can compute expressions without affecting the CPU flags.

Example:

```asm
lea eax, [ebx+4]
```

Equivalent to:

```asm
mov eax, ebx
add eax, 4
```

but done in a single instruction.

---

Another example:

```asm
lea eax, [ebx+ecx]
```

Result:

```text
EAX = EBX + ECX
```

Still **no memory access**.

---

Even more interesting:

```asm
lea eax, [ebx+ecx*4]
```

Computes:

```text
EAX = EBX + ECX×4
```

This is perfect for array indexing.

---

# Arrays

Suppose:

```c
int numbers[10];
```

Each `int` is 4 bytes.

Access:

```c
numbers[i]
```

Compiler may generate:

```asm
lea eax, [ebx+ecx*4]
```

Where:

```
EBX = &numbers
ECX = i
```

CPU computes:

```
address = base + index × element_size
```

Exactly what you need to find `numbers[i]`.

---

# Why Reverse Engineers Care

You'll constantly encounter instructions like:

```asm
lea eax, [ebp-0x20]
```

This tells you:

* A local variable lives at `[ebp-0x20]`.
* The program wants the **address** of that variable, not its contents.

If you instead see:

```asm
mov eax, [ebp-0x20]
```

The program wants the **value stored in the variable**.

Recognizing this difference helps you reconstruct the original C code.

---

# `LEA` in Ghidra

Suppose Ghidra shows:

```asm
lea eax,[ebp-0x40]
push eax
call gets
```

You can mentally translate it to:

```c
gets(buffer);
```

because:

```c
buffer
```

decays to a pointer (`&buffer[0]`) when passed to a function.

---

# Common Beginner Mistake

Many people think:

```asm
lea eax,[ebp-4]
```

means:

```
Load the value at ebp-4
```

It does **not**.

It means:

```
Calculate ebp-4
Store that address in eax
```

To load the value, the instruction would be:

```asm
mov eax,[ebp-4]
```

---

# Cheat Sheet

| Instruction            | Meaning                            |
| ---------------------- | ---------------------------------- |
| `mov eax, [ebp-4]`     | Load the value stored at `[EBP-4]` |
| `lea eax, [ebp-4]`     | Load the address `[EBP-4]` itself  |
| `lea eax, [ebx+4]`     | Compute `EBX + 4`                  |
| `lea eax, [ebx+ecx*4]` | Compute `EBX + ECX×4`              |

---

# Key Takeaways

* `LEA` stands for **Load Effective Address**.
* It **calculates an address** instead of reading memory.
* `MOV` with brackets reads from memory; `LEA` with brackets computes the address expression.
* Compilers commonly use `LEA` to pass pointers (e.g., local buffers) to functions.
* `LEA` is also widely used as an efficient arithmetic instruction for pointer arithmetic and array indexing.
* In reverse engineering, seeing `lea eax, [ebp-offset]` usually indicates that the code is working with the **address of a local variable**, not its value.

---

## Practice

Assume:

```text
EBP = 0xffffd100
Memory[0xffffd0f0] = 0xdeadbeef
```

What are the results of these instructions?

```asm
lea eax, [ebp-0x10]
mov ebx, [ebp-0x10]
```

**Answer:**

* `lea eax, [ebp-0x10]` → `EAX = 0xffffd0f0` (the address)
* `mov ebx, [ebp-0x10]` → `EBX = 0xdeadbeef` (the value stored at that address)

