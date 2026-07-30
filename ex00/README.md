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
