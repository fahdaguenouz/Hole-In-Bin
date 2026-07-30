# Hole in bin
# Binary Analysis Commands

## Navigate

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
