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

### Using objdump:

```bash
objdump -d bin
```

### Use GDB:

```bash
gdb ./bin
```

after that

```bash
disassemble main
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

**C → Assembly → Stack**

---

# Understanding the Assembly

After running:

```bash
gdb ./bin
```

disassemble the `main` function:

```bash
(gdb) disassemble main
```

Output:

```asm
0x080483f4 <+0>:  push   %ebp
0x080483f5 <+1>:  mov    %esp,%ebp
0x080483f7 <+3>:  and    $0xfffffff0,%esp
0x080483fa <+6>:  sub    $0x60,%esp

0x080483fd <+9>:  movl   $0x0,0x5c(%esp)

0x08048405 <+17>: lea    0x1c(%esp),%eax
0x08048409 <+21>: mov    %eax,(%esp)
0x0804840c <+24>: call   gets@plt

0x08048411 <+29>: mov    0x5c(%esp),%eax
0x08048415 <+33>: test   %eax,%eax
0x08048417 <+35>: je     0x8048427
```

---

# Step-by-Step Analysis

## 1. Creating the Stack Frame

```asm
push ebp
mov ebp, esp
sub esp, 0x60
```

Equivalent C idea:

```c
// reserve 96 bytes for local variables
```

The instruction

```asm
sub esp, 0x60
```

means:

```
0x60 = 96 bytes
```

The compiler reserves **96 bytes** on the stack for local variables.

---

## 2. Initializing the Variable

```asm
movl $0x0,0x5c(%esp)
```

Meaning:

```
*(esp + 0x5c) = 0
```

Equivalent C:

```c
int modified = 0;
```

---

## 3. Finding the Buffer

```asm
lea 0x1c(%esp), %eax
```

This instruction often confuses beginners.

`lea` means:

> Load Effective Address

It **does not read memory**.

Instead it computes an address.

Example:

```
ESP = 0xffffd100

lea 0x1c(%esp), eax

↓

EAX = 0xffffd11c
```

Graphically:

```
ESP
│
├───────────────
│
│ buffer starts here
│
▼
0xffffd11c
```

So now the register contains

```
EAX = &buffer
```

Notice the difference:

```
mov eax,[esp+0x1c]
```

means

```
eax = buffer_contents
```

while

```
lea eax,[esp+0x1c]
```

means

```
eax = &buffer
```

This distinction is extremely important in reverse engineering.

---

## 4. Passing the Argument to gets()

Assembly:

```asm
mov %eax,(%esp)
call gets@plt
```

Before the call:

```
EAX = 0xffffd11c
```

After

```asm
mov %eax,(%esp)
```

the stack becomes

```
ESP
│
├───────────────
│
│ 0xffffd11c
│
└───────────────
```

When

```asm
call gets
```

is executed, it is exactly equivalent to

```c
gets(buffer);
```

because the first function argument on 32-bit x86 is passed on the stack.

---

# Register Flow

This is what happens to the registers.

### Before `lea`

```
ESP = 0xffffd100

EAX = ??????
```

---

### After `lea`

```asm
lea 0x1c(%esp),eax
```

```
ESP = 0xffffd100

EAX = 0xffffd11c
```

Notice that **EAX contains an address**, not data.

---

### After

```asm
mov eax,(esp)
```

```
Stack

ESP
│
▼
0xffffd11c
```

The address is copied to the stack as the first argument for `gets()`.

---

# Visualizing the Stack

Immediately before calling `gets()`:

```
Higher Addresses
──────────────────────────────

modified
│
│ 4 bytes
│
├─────────────────────────────

buffer
│
│ 64 bytes
│
├─────────────────────────────

argument for gets()

──────────────────────────────
Lower Addresses
```

Or with offsets:

```
ESP
│
├──────────────────────────────
│ argument for gets()
├──────────────────────────────
│
│ buffer[64]
│
│ starts at ESP+0x1c
│
├──────────────────────────────
│ modified
│
│ ESP+0x5c
└──────────────────────────────
```

---

# Why Exactly 64 Bytes?

The assembly tells us

```
buffer  -> ESP + 0x1c

modified -> ESP + 0x5c
```

Subtracting:

```
0x5c
-0x1c
-----
0x40
```

```
0x40 = 64
```

Therefore,

```
buffer size = 64 bytes
```

Anything beyond 64 bytes begins overwriting `modified`.

---

# Overflow Example

Input:

```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

The first 64 bytes fill the buffer.

```
buffer

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

The next bytes overwrite `modified`.

```
modified

AAAA
```

Since

```
'A' = 0x41
```

the integer becomes

```
0x41414141
```

instead of

```
0x00000000
```

The condition

```c
if(modified != 0)
```

becomes true.

---

# Reconstructing the Original C Code

From the assembly we can infer something very close to:

```c
int main()
{
    volatile int modified = 0;
    char buffer[64];

    gets(buffer);

    if (modified != 0)
        puts("you have changed the 'modified' variable");
    else
        puts("Try again?");
}
```

---
```markdown
sub esp,0x60
│
▼
Reserve 96 bytes

movl $0x0,0x5c(%esp)
│
▼
modified = 0

lea 0x1c(%esp),eax
│
▼
EAX = &buffer

mov eax,(esp)
│
▼
Pass &buffer as argument

call gets
│
▼
gets writes into buffer

Too many bytes?
│
▼
buffer overflows

Next memory = modified
│
▼
modified changes

if(modified != 0)
│
▼
Success!
```
---

## Extra Notes (Nice Addition)

At the end, I'd add a section called **Lessons Learned**:

```markdown
## Lessons Learned

- `lea` loads an **address**, not a value.
- `mov` copies data between registers and memory.
- On 32-bit x86, function arguments are passed on the stack.
- `gets()` has no bounds checking and is inherently unsafe.
- Local variables are stored next to each other on the stack, making adjacent variables vulnerable to overflow.
- Understanding stack offsets (e.g., `ESP + 0x1c`) helps reconstruct the original C source from assembly.
```

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
