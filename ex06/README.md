# ex06 – Calling `winner()` with GDB

## Goal

Execute the hidden `winner()` function using GDB to display the success message.

---

## Step 1 – Find the Function Addresses

Disassemble the binary and locate the hidden function and `main()`.

```bash
objdump -d bin | grep "<winner>"
objdump -d bin | grep "<main>"
```

Output:

```text
08048864 <winner>:
08048889 <main>:
```

---

## Step 2 – Start GDB

```bash
gdb ./bin
```

---

## Step 3 – Set a Breakpoint

Stop execution at the beginning of `main()`.

```gdb
(gdb) break main
```

---

## Step 4 – Run the Program

Run the program with any arguments (or none if not required).

```gdb
(gdb) run AAAA BBBB CCCC
```

Execution stops at:

```text
Breakpoint 1, main (...)
```

---

## Step 5 – Redirect Execution to `winner()`

Change the instruction pointer (`EIP`) to the address of `winner()`.

```gdb
(gdb) set $eip = 0x08048864
```

---

## Step 6 – Continue Execution

```gdb
(gdb) continue
```

Output:

```text
that wasn't too bad now, was it?
```

The challenge is considered completed when this message appears.

---

## Complete GDB Session

```text
$ gdb ./bin

(gdb) break main
Breakpoint 1 at 0x08048889

(gdb) run AAAA BBBB CCCC

Breakpoint 1, main ()

(gdb) set $eip = 0x08048864

(gdb) continue

that wasn't too bad now, was it?
```

---

## Summary

Commands used:

```bash
objdump -d bin | grep "<winner>"
objdump -d bin | grep "<main>"
```

```gdb
gdb ./bin
break main
run AAAA BBBB CCCC
set $eip = 0x08048864
continue
```

This solution redirects execution from `main()` to the hidden `winner()` function by modifying the instruction pointer (`EIP`) in GDB.