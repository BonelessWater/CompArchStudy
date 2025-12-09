# Instruction Set Architecture (ISA) - Study Guide

## What is an ISA?

**Definition**: A specification of a standardized programmer-visible interface to hardware

**Three Key Components**:
1. **Set of instructions**: With arguments, syntax, and encoding
2. **Named storage locations**: Registers and memory
3. **Addressing modes**: Ways to name/access locations

**Example Instructions**:
```assembly
add  x1, x2, x3     # x1 = x2 + x3 (register-to-register)
addi x1, x2, 3      # x1 = x2 + 3 (immediate)
lw   x1, 4(x0)      # x1 = Memory[x0 + 4] (load)
sw   x1, 4(x0)      # Memory[x0 + 4] = x1 (store)
```

**ISA vs Implementation**:
- **ISA**: The interface (what programmer sees)
- **Implementation**: How it's built (pipeline, cache, memory)

---

## Classifying Instruction Set Architectures

### By Operand Storage

**1. Stack Architecture**
- Operands implicitly on top of stack
- Example: `C = A + B`
```assembly
push A
push B
add        # Pop two operands, push result
pop C
```

**Advantages**: Simple, compact code
**Disadvantages**: Stack becomes bottleneck, not flexible

**2. Accumulator Architecture**
- One operand implicitly in accumulator register
- Example: `C = A + B`
```assembly
load  A     # ACC = A
add   B     # ACC = ACC + B
store C     # C = ACC
```

**Advantages**: Minimal internal state, short instructions
**Disadvantages**: Accumulator is bottleneck, high memory traffic

**3. Register-Memory Architecture**
- One operand can be in memory
- Example: `C = A + B`
```assembly
load  r1, A      # r1 = A
add   r1, B      # r1 = r1 + Memory[B]
store r1, C      # C = r1
```

**Advantages**: Compact code (don't need to load both operands)
**Disadvantages**: Variable operand access time, complicates pipeline

**4. Load-Store Architecture (Register-Register)**
- ALL operands must be in registers (except load/store)
- Example: `C = A + B`
```assembly
load  r1, A      # r1 = Memory[A]
load  r2, B      # r2 = Memory[B]
add   r3, r1, r2 # r3 = r1 + r2
store r3, C      # Memory[C] = r3
```

**Advantages**: Simple, fixed-cycle instructions, easy to pipeline
**Disadvantages**: More instructions, larger code size

**Modern Trend**: Load-store (RISC architectures)

---

## Comparison of Architecture Types

| Feature | Stack | Accumulator | Reg-Mem | Load-Store |
|---------|-------|-------------|---------|------------|
| **# of explicit operands** | 0 | 1 | 2 or 3 | 3 |
| **Memory operands** | All | 1 | 1 | 0 (except load/store) |
| **Code size** | Small | Small | Medium | Large |
| **Instruction encoding** | Simple | Simple | Medium | Complex |
| **Pipeline efficiency** | Poor | Poor | Medium | Excellent |
| **Examples** | Java VM, x87 FP | Early CPUs | x86, VAX | RISC-V, ARM, MIPS |

### Notation
**(m, n)** means:
- **m**: Number of memory operands
- **n**: Total number of operands

Examples:
- Stack: (0, 0) - both operands implicit
- Accumulator: (1, 1) - one memory operand, one total
- Register-Memory: (1, 2) or (1, 3)
- Load-Store: (0, 3) except load/store which are (1, 2)

---

## Memory Addressing

### Byte Order (Endianness)

**Big-Endian**: Most significant byte at lowest address
```
Address:  0    1    2    3
Value:   12   34   56   78  (0x12345678)
```

**Little-Endian**: Least significant byte at lowest address
```
Address:  0    1    2    3
Value:   78   56   34   12  (0x12345678)
```

**Important**: String "1234" is stored as '1' '2' '3' '4'
- LSB of hex 0x1234 is 34
- LSB of string "1234" is '1'

### Alignment

**Definition**: Data at address divisible by its size

**Examples**:
- Byte: Can be at any address
- Halfword (2 bytes): Address must be multiple of 2
- Word (4 bytes): Address must be multiple of 4
- Doubleword (8 bytes): Address must be multiple of 8

**Aligned Example**:
```
Address:  0  1  2  3  4  5  6  7
Word 1:  [-------]           ✓ Aligned (address 0, multiple of 4)
Word 2:           [-------]  ✓ Aligned (address 4, multiple of 4)
```

**Misaligned Example**:
```
Address:  0  1  2  3  4  5  6  7
Word:        [-------]        ✗ Misaligned (address 1, not multiple of 4)
```

**Why Alignment Matters**:
- Faster memory access
- Simpler hardware
- Some architectures require alignment (fault on misaligned access)

---

## Addressing Modes

| Mode | Example | Meaning | Use Case |
|------|---------|---------|----------|
| **Register** | add x1, x2, x3 | x1 = x2 + x3 | Local variables |
| **Immediate** | addi x1, x2, 3 | x1 = x2 + 3 | Constants |
| **Displacement** | lw x1, 100(x2) | x1 = M[x2 + 100] | Array access, struct fields |
| **Register Indirect** | lw x1, 0(x2) | x1 = M[x2] | Pointer dereference |
| **Indexed** | add x1, x2(x3) | x1 = M[x2 + x3] | Array in loop |
| **Direct/Absolute** | lw x1, 1000 | x1 = M[1000] | Static data, globals |
| **PC-relative** | beq x1, x0, 100 | if (x1==0) PC = PC+100 | Branches, loops |

### Addressing Mode Usage (SPEC89 on VAX)

**Distribution**:
- Register mode: 45-55% (most common)
- Displacement: 20-35%
- Immediate: 15-20%
- Others: <5%

**Conclusion**: Register mode accounts for almost half of operand accesses

### Displacement Size

**SPEC CPU2000 on Alpha** (supports 16-bit displacement):
- 1 bit: ~25% (0 or 1)
- 4 bits: ~50%
- 8 bits: ~75%
- 12 bits: ~90%
- **16 bits: >99%**

**Recommendation**: 12-16 bits sufficient for displacement

### Immediate Size

**SPEC CPU2000 on Alpha** (supports 16-bit immediate):
- 4 bits: ~25%
- 8 bits: ~50%
- 12 bits: ~75%
- **16 bits: >95%**

**Recommendation**: 8-16 bits for immediate values

---

## Type and Size of Operands

### Common Data Types

| Type | Size | RISC-V Support |
|------|------|----------------|
| **Byte** | 8 bits | lb, lbu, sb |
| **Halfword** | 16 bits | lh, lhu, sh |
| **Word** | 32 bits | lw, lwu, sw |
| **Doubleword** | 64 bits | ld, sd |
| **Single-precision float** | 32 bits | flw, fsw |
| **Double-precision float** | 64 bits | fld, fsd |

**Note**: 
- `lb` = load byte (sign-extended)
- `lbu` = load byte unsigned (zero-extended)
- All integer loads sign-extend to fill 64-bit register (except unsigned variants)

### Why Decimal?

**Problem**: Some decimal fractions have no exact binary representation

**Example**: 0.10₁₀ = ?₂
- 0.10₁₀ = 0.0001100110011001... (repeating)
- Cannot be exactly represented in binary

**Solution**: Binary-Coded Decimal (BCD)
- Each decimal digit stored in 4 bits
- 0.10₁₀ = 0001 0000 (BCD)
- Used in financial calculations where exact decimal representation required

---

## Operations in the Instruction Set

### Instruction Categories

| Category | Examples | Typical % |
|----------|----------|-----------|
| **Arithmetic/Logical** | add, sub, and, or, shift | 45-50% |
| **Data Transfer** | load, store, move | 30-35% |
| **Control Flow** | branch, jump, call, return | 15-20% |
| **System** | syscall, trap | <1% |
| **Floating-Point** | fadd, fmul, fdiv | Varies |

### Top 10 Instructions (80x86, SPECint92)

1. **load** - 22%
2. **conditional branch** - 20%
3. **compare** - 16%
4. **store** - 12%
5. **add** - 8%
6. **and** - 6%
7. **sub** - 5%
8. **move register-register** - 4%
9. **call** - 1%
10. **return** - 1%

**Total**: Top 10 = ~96% of all instructions!

---

## SIMD and Multimedia Instructions

### SIMD (Single Instruction Multiple Data)

**Concept**: Perform same operation on multiple data elements simultaneously

**Example**: 4 additions in one instruction
```
Register A: [a1][a2][a3][a4]  (4 × 16-bit values)
Register B: [b1][b2][b3][b4]  (4 × 16-bit values)
Result:     [a1+b1][a2+b2][a3+b3][a4+b4]
```

**Why SIMD Works**:
- Wide registers (64-bit) with narrow data (8-bit pixels, 16-bit audio)
- Can pack 2-8 values per register
- Multimedia data is inherently parallel

**Features**:
- Carry disabled at word boundaries (prevent overflow between elements)
- Optional saturating arithmetic (clamp to max/min instead of wrap)
- Specialized operations (e.g., half-word add HADD in HP PA)

**Applications**:
- Image processing (pixel operations)
- Audio/video encoding
- Graphics rendering
- Signal processing

---

## Control Flow Instructions

### Four Basic Types

1. **Conditional Branches**: `beq`, `bne`, `blt`, `bge`
2. **Unconditional Jumps**: `j`, `jal`
3. **Procedure Calls**: `jal` (jump and link)
4. **Procedure Returns**: `jalr x0, 0(x1)` or `ret`

### Distribution (SPEC CPU2000 on Alpha)

- Conditional branches: ~75%
- Jumps: ~3%
- Calls: ~12%
- Returns: ~10%

### Addressing Modes for Control Flow

**1. PC-Relative** (PC + displacement)
```assembly
beq  x1, x2, 100    # if (x1 == x2) PC = PC + 100
```
**Uses**:
- Conditional branches
- Position-independent code (relocatable)
- Target known at compile time

**2. Register Indirect** (register contains address)
```assembly
jalr x1, 0(x2)      # x1 = PC + 4; PC = x2
```
**Uses**:
- Procedure returns
- Switch/case statements
- Virtual functions (C++ methods)
- Function pointers
- Dynamically shared libraries

### Branch Distance Distribution

**SPEC CPU2000 on Alpha**:
- 4 bits: ~50% of branches
- 8 bits: ~75%
- 12 bits: ~90%
- **16 bits: ~99%**

**Most frequent branches**: Can be encoded in 4-8 bits

### Branch Comparison Types

**Distribution** (SPEC CPU2000):
- **Compare to zero**: ~50% (most common)
- **Compare two registers**: ~30%
- **Compare to constant**: ~20%

**Design Implication**: Provide efficient zero-comparison instructions

---

## Conditional Branch Options

### Three Common Approaches

**1. Condition Code (CC)**
```assembly
sub  x1, x2, x3     # Sets N, Z, V, C flags
beq  target         # Branch if Z flag set
```
**Advantages**: Condition already computed
**Disadvantages**: Extra state, complicates out-of-order execution

**2. Condition Register**
```assembly
slt  x1, x2, x3     # x1 = (x2 < x3) ? 1 : 0
bne  x1, x0, target # Branch if x1 ≠ 0
```
**Advantages**: Explicit comparison, flexible
**Disadvantages**: Extra instruction

**3. Compare and Branch**
```assembly
blt  x2, x3, target # Branch if x2 < x3
```
**Advantages**: Single instruction, clear semantics
**Disadvantages**: May increase instruction complexity

**RISC-V Choice**: Compare and branch (option 3)

---

## Procedure Calling Conventions

### Two Major Conventions

**1. Caller Saves**
- Caller saves registers it will need after call
- Even if callee doesn't use them

**Advantages**: Callee doesn't need to save anything
**Disadvantages**: Caller must save everything it might need

**2. Callee Saves**
- Called procedure saves registers it will overwrite

**Advantages**: More efficient if many small procedures (don't use many regs)
**Disadvantages**: Overhead even if caller doesn't need saved values

### RISC-V Convention (Hybrid)

**Caller-saved (Temporary registers)**:
- x1 (ra): Return address
- x5-x7 (t0-t2): Temporaries
- x10-x17 (a0-a7): Function arguments/return values
- x28-x31 (t3-t6): More temporaries

**Callee-saved (Saved registers)**:
- x8-x9 (s0-s1): Saved registers
- x18-x27 (s2-s11): More saved registers

**Special**:
- x0: Always zero (hardwired)
- x2 (sp): Stack pointer
- x3 (gp): Global pointer
- x4 (tp): Thread pointer

**Optimal Strategy**: Combine both for best performance

---

## Instruction Encoding

### Encoding Design Trade-offs

| Feature | Fixed-Length | Variable-Length |
|---------|--------------|-----------------|
| **Decode complexity** | Simple | Complex |
| **Instruction fetch** | Easy | Difficult |
| **Code size** | Larger | Smaller |
| **Performance** | Higher | Lower (decode bottleneck) |
| **Examples** | RISC-V, ARM, MIPS | x86, VAX |

### RISC-V Instruction Formats (All 32-bit)

**R-Format** (Register-to-register operations)
```
[funct7 7-bit][rs2 5-bit][rs1 5-bit][funct3 3-bit][rd 5-bit][opcode 7-bit]
Example: add x5, x6, x7
```

**I-Format** (Immediate and loads)
```
[imm[11:0] 12-bit][rs1 5-bit][funct3 3-bit][rd 5-bit][opcode 7-bit]
Example: addi x15, x1, -50
         lw x5, 40(x6)
```

**S-Format** (Stores)
```
[imm[11:5] 7-bit][rs2 5-bit][rs1 5-bit][funct3 3-bit][imm[4:0] 5-bit][opcode 7-bit]
Example: sw x5, 40(x6)
```

**B-Format** (Branches)
```
[imm[12,10:5] 7-bit][rs2 5-bit][rs1 5-bit][funct3 3-bit][imm[4:1,11] 5-bit][opcode 7-bit]
Example: beq x5, x6, 100
```

**U-Format** (Upper immediate)
```
[imm[31:12] 20-bit][rd 5-bit][opcode 7-bit]
Example: lui x5, 0x12345
```

**J-Format** (Jumps)
```
[imm[20,10:1,11,19:12] 20-bit][rd 5-bit][opcode 7-bit]
Example: jal x1, 2048
```

### Format Design Principles

1. **Opcode always in same position**: Bits [6:0]
2. **Register specifiers in same position**: 
   - rs1: [19:15]
   - rs2: [24:20]
   - rd: [11:7]
3. **Immediate fields**: Vary by format but source bits consistent
4. **All instructions 32 bits**: Simplifies fetch and decode

---

## R-Format Example

```assembly
add x5, x6, x7
```

**Binary Encoding**:
```
funct7  rs2  rs1  funct3  rd   opcode
0000000 00111 00110 000 00101 0110011
```

**Fields**:
- funct7 = 0000000 (add function)
- rs2 = 00111 (x7)
- rs1 = 00110 (x6)
- funct3 = 000 (add variant)
- rd = 00101 (x5)
- opcode = 0110011 (R-type ALU)

**Hexadecimal**: 0x007302B3

---

## I-Format Example

```assembly
addi x15, x1, -50
```

**Binary Encoding**:
```
imm[11:0]      rs1  funct3  rd   opcode
111111001110 00001 000 01111 0010011
```

**Fields**:
- imm = 111111001110 (-50 in 2's complement)
- rs1 = 00001 (x1)
- funct3 = 000 (addi)
- rd = 01111 (x15)
- opcode = 0010011 (I-type ALU)

**Hexadecimal**: 0xFCE08793

---

## Role of Compilers

### Compiler Structure

**Frontend** (Language-dependent):
1. Lexical analysis (tokenization)
2. Syntax analysis (parsing)
3. Semantic analysis (type checking)
4. Intermediate representation (IR) generation

**Optimizer** (Machine-independent):
1. High-level optimizations (loop unrolling, inlining)
2. Common subexpression elimination
3. Dead code elimination
4. Constant propagation

**Backend** (Machine-dependent):
1. Instruction selection
2. Register allocation
3. Instruction scheduling

### Example Compilation

**C Code**:
```c
void swap(int v[], int k) {
    int temp;
    temp = v[k];
    v[k] = v[k+1];
    v[k+1] = temp;
}
```

**Assembly** (RISC-V concept):
```assembly
swap:
    slli x5, x11, 2      # x5 = k * 4 (byte offset)
    add  x5, x10, x5     # x5 = address of v[k]
    lw   x6, 0(x5)       # x6 = v[k]
    lw   x7, 4(x5)       # x7 = v[k+1]
    sw   x7, 0(x5)       # v[k] = v[k+1]
    sw   x6, 4(x5)       # v[k+1] = temp
    ret
```

### Common Compiler Optimizations

| Optimization | Effect | Speedup |
|--------------|--------|---------|
| **None** | Baseline | 1.0× |
| **Local** | Within basic blocks | 1.2-1.5× |
| **Global** | Within procedures | 1.5-2.0× |
| **Interprocedural** | Across procedures | 2.0-3.0× |
| **Loop optimizations** | Unrolling, fusion | 1.5-5.0× |
| **Inlining** | Eliminate call overhead | 1.1-1.3× |

**Combined Effect**: Can achieve 2-5× speedup with all optimizations

### Phase Ordering Problem

**Challenge**: Order of optimization passes affects final code quality

**Example**:
```c
R = a + b - c + d × (g + b - c)
```

**Common Subexpression Elimination**:
```c
temp = b - c
R = a + temp + d × (g + temp)
```
- Needs temporary variable
- Increases register pressure

**Register Allocation**:
- Done late in compilation
- May cause register spills if too many temps

**Dilemma**: 
- Eliminate subexpression → save computation but use register
- Recompute → waste computation but save register
- **Depends on register pressure!**

---

## Architectural Support for Compilers

### Design Principles

**1. Provide Regularity (Orthogonality)**
- Any register can be used with any instruction
- Any addressing mode can be used with any operation
- Independence of features

**2. Provide Primitives, Not Solutions**
- Don't directly support specific languages or algorithms
- Provide basic operations that compiler can combine

**3. Simplify Trade-offs**
- Make it easy to generate efficient code at compile time
- Don't force runtime decisions on things known at compile time

**4. Don't Interpret Compile-Time Values**
- Hardware shouldn't interpret constants/immediates
- Leave interpretation to compiler

**Good Example** (RISC):
```assembly
add  x1, x2, x3    # Any register combination works
addi x1, x2, 100   # Immediate version available
```

**Bad Example** (Complex):
```assembly
auto_increment_add [x1++], [x2++], x3
# Forced behavior, hard to optimize
```

---

## Putting It All Together - Design Guidelines

### Recommended ISA Features

**1. Use GPRs with Load-Store Architecture**
- 16-32 general-purpose registers
- Memory access only through load/store
- Simple, regular, pipelineable

**2. Support Simple Addressing Modes**
- Displacement: 12-16 bits
- Immediate: 8-16 bits  
- Register indirect
- PC-relative for branches

**3. Support Basic Types**
- 8, 16, 32, 64-bit integers
- 32, 64-bit floating-point
- All sign-extended to register width

**4. Support Most-Executed Operations**
- Load, store
- Add, subtract
- Move, shift
- Compare (equal, not equal, less than)
- Branch (PC-relative, at least 8 bits)
- Jump, call, return

**5. Fixed Instruction Encoding**
- For performance (simple decode)
- Variable encoding acceptable for code density
- Trade-off based on design goals

**6. Orthogonal Instruction Set**
- Any register with any instruction
- Any addressing mode with any operation
- Simplifies compiler, improves regularity

---

## RISC-V Architecture Summary

### Base Architecture

**RV64I** (64-bit base integer):
- 32 64-bit general-purpose registers (x0-x31)
  - x0 hardwired to zero
- 32 64-bit floating-point registers (f0-f31)
- Load-store architecture
- Fixed 32-bit instruction encoding
- Support for 8, 16, 32, 64-bit integers
- Support for 32, 64-bit floating-point

### Extensions

**RV64G** = RV64I + M + A + F + D
- **M**: Integer multiply/divide
- **A**: Atomic operations
- **F**: Single-precision floating-point
- **D**: Double-precision floating-point

**Other Extensions**:
- **C**: Compressed (16-bit) instructions
- **V**: Vector operations
- **Q**: Quad-precision floating-point

### Common RISC-V Instructions

**Arithmetic/Logical**:
```assembly
add, sub, addi          # Addition, subtraction
and, or, xor            # Logical operations
sll, srl, sra          # Shifts (logical/arithmetic)
slt, sltu              # Set less than (signed/unsigned)
```

**Load/Store**:
```assembly
lb, lh, lw, ld         # Load byte/half/word/double
lbu, lhu, lwu          # Load unsigned
sb, sh, sw, sd         # Store byte/half/word/double
```

**Control Flow**:
```assembly
beq, bne               # Branch equal/not equal
blt, bge               # Branch less/greater-equal
bltu, bgeu             # Unsigned versions
jal, jalr              # Jump and link (call/return)
```

### RISC-V Instruction Mix (SPECint2006)

- **ALU operations**: ~50%
- **Loads**: ~25% (typically more than stores)
- **Stores**: ~10%
- **Conditional branches**: ~12%
- **Unconditional jumps**: ~3%

---

## Important RISC-V Concepts

### Sign Extension

**All integer loads sign-extend to 64 bits** (except unsigned variants)

**Example**:
```assembly
lb  x1, 0(x2)    # Load byte, sign-extend
# If byte = 0xFF (-1), x1 = 0xFFFFFFFFFFFFFFFF

lbu x1, 0(x2)    # Load byte unsigned, zero-extend
# If byte = 0xFF, x1 = 0x00000000000000FF
```

### Register x0

**Always zero**, cannot be modified
```assembly
add x1, x0, x2   # x1 = 0 + x2 = x2 (move)
beq x1, x0, L    # if (x1 == 0) goto L
```

### Shift Operations

**Logical vs Arithmetic Right Shift**:

```assembly
# x2 = 32 (0b00100000)
srl x1, x2, 4    # x1 = 2 (0b00000010), shift in 0s

# x2 = -32 (0b11100000 in 8-bit)
srl x1, x2, 4    # x1 = 14 (0b00001110), shift in 0s
sra x1, x2, 4    # x1 = -2 (0b11111110), shift in 1s (sign bit)
```

**Left Shift** (always logical):
```assembly
# x2 = 2 (0b00000010)
sll x1, x2, 4    # x1 = 32 (0b00100000)
```

### Determining Sign of Binary Number

**Rule**: If MSB = 1 in 2's complement, number is negative

**Example**:
```assembly
jal x8, -20
# Binary: 11111111111111101100 01000 0000000
#         ^^^^ MSB = 1, so offset is negative
```

### Common Idioms

**NOP (No Operation)**:
```assembly
addi x0, x0, 0   # x0 = x0 + 0 (x0 always 0)
```

**Move**:
```assembly
addi x1, x2, 0   # x1 = x2 + 0 = x2
```

**Negate**:
```assembly
sub x1, x0, x2   # x1 = 0 - x2 = -x2
```

**Clear Register**:
```assembly
add x1, x0, x0   # x1 = 0 + 0 = 0
```

---

## Key Design Principles Summary

### RISC Philosophy

1. **Simplicity**: Simple, regular instructions
2. **Regularity**: Orthogonal design
3. **Efficiency**: Optimize common case
4. **Pipelinability**: Fixed-length, regular format
5. **Compiler-Friendly**: Easy to generate good code

### Common Case Optimization

**Based on empirical data**:
- Most operations are ALU (50%)
- Most memory accesses are loads (25% vs 10% stores)
- Most branches are conditional (vs unconditional jumps)
- Most branch targets are nearby (8-12 bit displacement)
- Most immediates are small (8-16 bits)
- Most register accesses use small register numbers

**Design implication**: Optimize these common cases!

---

## Quick Reference Tables

### RISC-V Instruction Categories

| Type | Format | Example | Usage |
|------|--------|---------|-------|
| **ALU Register** | R | add x1,x2,x3 | ~30% |
| **ALU Immediate** | I | addi x1,x2,10 | ~20% |
| **Load** | I | lw x1,0(x2) | ~25% |
| **Store** | S | sw x1,0(x2) | ~10% |
| **Branch** | B | beq x1,x2,L | ~12% |
| **Jump** | J | jal x1,func | ~3% |

### Data Type Sizes

| Type | Bits | RISC-V Name |
|------|------|-------------|
| Byte | 8 | b |
| Halfword | 16 | h |
| Word | 32 | w |
| Doubleword | 64 | d (or default) |
| Single Float | 32 | s |
| Double Float | 64 | d |

### Addressing Mode Summary

| Mode | Syntax | Calculation | Use |
|------|--------|-------------|-----|
| **Immediate** | addi x1,x2,100 | x2 + 100 | Constants |
| **Register** | add x1,x2,x3 | x2 + x3 | Local vars |
| **Displacement** | lw x1,100(x2) | M[x2+100] | Arrays, structs |
| **PC-relative** | beq x1,x2,L | PC + offset | Branches |
| **Register indirect** | jalr x0,0(x1) | M[x1] | Pointers, returns |

---

## Study Tips

1. **Understand trade-offs**: Fixed vs variable encoding, caller vs callee save
2. **Know the numbers**: Common instruction %, displacement sizes, immediate sizes
3. **Memorize formats**: R, I, S, B, U, J - which fields where
4. **Practice encoding**: Convert assembly to binary and hex
5. **Understand sign extension**: Critical for loads and immediates
6. **Know x0 tricks**: Used for comparisons, moves, NOPs
7. **Compiler perspective**: How do optimizations affect code generation
