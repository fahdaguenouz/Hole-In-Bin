# Reverse Engineering & Binary Exploitation Roadmap

## Phase 1 — Binary Basics

### File Identification
- [x] What is an ELF file?
- [x] Use `file`
- [ ] Use `readelf`
- [ ] ELF Header
- [ ] Program Headers
- [ ] Sections (`.text`, `.data`, `.rodata`, `.bss`)
- [ ] Dynamic vs Static Linking
- [ ] Symbols & Stripped Binaries
- [ ] BuildID
- [ ] Shared Libraries (`ldd`)

### Linux Tools
- [x] `file`
- [ ] `readelf`
- [ ] `objdump`
- [ ] `strings`
- [ ] `nm`
- [ ] `ldd`
- [ ] `hexdump`
- [ ] `xxd`

---

# Phase 2 — CPU Fundamentals

### Architecture
- [x] 32-bit vs 64-bit
- [x] Intel i386
- [x] Little Endian
- [ ] Big Endian
- [ ] Virtual Memory Basics

### Registers (x86)
- [x] EAX
- [x] EBX
- [x] ECX
- [x] EDX
- [x] ESP
- [x] EBP
- [x] EIP

### Register Usage
- [x] Function Return Value (EAX)
- [ ] Stack Pointer
- [ ] Base Pointer
- [ ] Instruction Pointer
- [ ] Register Lifetime

---

# Phase 3 — The Stack

- [ ] What is the Stack?
- [ ] Stack Growth
- [ ] push
- [ ] pop
- [ ] Return Address
- [ ] Local Variables
- [ ] Function Arguments
- [ ] Stack Frame
- [ ] Function Prologue
- [ ] Function Epilogue

---

# Phase 4 — Assembly (Read Only)

## Data Movement
- [ ] mov
- [ ] lea
- [ ] push
- [ ] pop

## Arithmetic
- [ ] add
- [ ] sub
- [ ] inc
- [ ] dec
- [ ] xor

## Comparisons
- [ ] cmp
- [ ] test

## Jumps
- [ ] jmp
- [ ] je
- [ ] jne
- [ ] jg
- [ ] jl
- [ ] jge
- [ ] jle

## Functions
- [ ] call
- [ ] ret
- [ ] leave

## Misc
- [ ] nop

---

# Phase 5 — Memory

- [ ] Memory Layout
- [ ] Stack
- [ ] Heap
- [ ] Data Segment
- [ ] BSS
- [ ] Code (.text)
- [ ] Pointers
- [ ] Dereferencing
- [ ] Addressing Modes

---

# Phase 6 — GDB

### Basics
- [ ] Start Program
- [ ] Run
- [ ] Quit

### Breakpoints
- [ ] break
- [ ] delete
- [ ] info breakpoints

### Execution
- [ ] next
- [ ] step
- [ ] continue
- [ ] finish

### Registers
- [ ] info registers
- [ ] print $eax
- [ ] modify registers

### Memory
- [ ] x/x
- [ ] x/s
- [ ] x/i
- [ ] x/20wx

### Stack
- [ ] backtrace
- [ ] frame

---

# Phase 7 — Reading Binaries

- [ ] Find main()
- [ ] Follow function calls
- [ ] Identify loops
- [ ] Identify if statements
- [ ] Recognize switch statements
- [ ] Understand local variables
- [ ] Track register values
- [ ] Follow pointers

---

# Phase 8 — Reverse Engineering

- [ ] Read simple functions
- [ ] Recover C code mentally
- [ ] Identify input validation
- [ ] Find hidden functions
- [ ] Find strings
- [ ] Follow execution flow

---

# Phase 9 — Binary Exploitation Basics

- [ ] Buffer Overflow
- [ ] Saved EBP
- [ ] Return Address
- [ ] Overwriting EIP
- [ ] NOP Sled
- [ ] Shellcode (Concept)
- [ ] ret2libc (Concept)
- [ ] ASLR
- [ ] NX
- [ ] PIE
- [ ] Stack Canaries

---

# Practice

## hole-in-bin
- [ ] ex00
- [ ] ex01
- [ ] ex02
- [ ] ex03
- [ ] ex04
- [ ] ex05
- [ ] ex06
- [ ] ex07
- [ ] ex08
- [ ] ex09

---

# Resources

- [ ] GDB
- [ ] objdump
- [ ] readelf
- [ ] pwn.college (Relevant Modules)
- [ ] Ghidra