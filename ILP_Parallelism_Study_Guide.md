# Instruction-Level Parallelism (ILP) - Study Guide

## Key Formula

**Pipeline CPI = Ideal CPI + Structural Stalls + Data Stalls + Control Stalls**

To improve performance: Decrease pipeline CPI (reduce hazards) or reduce Ideal CPI

---

## Instruction-Level Parallelism (ILP)

**Definition**: The potential overlap among instructions that can be evaluated in parallel

**Two Approaches to Exploit ILP**:
1. **Static (Software/Compiler-based)**: Dominates embedded systems
2. **Dynamic (Hardware/Processor-based)**: Dominates desktop and server markets

**Basic Block (BB)**: Straight-line code with no branches in or out
- Average size: 4-7 instructions
- ILP within BB is limited → Need to parallelize across multiple basic blocks

**Loop-Level Parallelism (LLP)**: Perform multiple loop iterations in parallel
- Can be converted to ILP via loop unrolling
- Statically by compiler or dynamically by hardware

---

## Three Types of Dependencies

### 1. Data Dependence (True Dependence)
**Definition**: Instruction B is data dependent on A if:
- B uses a result produced by A, OR
- B is data dependent on C, and C is data dependent on A

**Potential hazard**: RAW (Read After Write)

**Example**:
```
lw   x1, 0(x1)
add  x4, x1, x2    # Depends on lw
```

### 2. Name Dependence
Occurs when instructions access same storage location but are NOT data dependent

**Two sub-types**:
- **Antidependence**: A reads, then B writes → WAR (Write After Read) hazard
- **Output dependence**: A writes, then B writes → WAW (Write After Write) hazard

**Can be eliminated by register renaming**

### 3. Control Dependence
Execution of instruction depends on outcome of earlier conditional branch

**Preserved by**:
- Executing instructions in program order
- Detecting branch hazards before executing dependent instructions

**Two critical properties for correctness**:
1. **Exception behavior**: Must maintain proper exception handling
2. **Data flow**: Must preserve data flow dependencies

---

## Three Types of Data Hazards

| Hazard | Description | When It Occurs |
|--------|-------------|----------------|
| **RAW** | Read After Write | j reads before i writes (true dependence) |
| **WAW** | Write After Write | j writes before i writes (output dependence) |
| **WAR** | Write After Read | j writes before i reads (antidependence) |

**Note**: 
- RAW is a true data dependence
- WAW and WAR are name dependencies (can be eliminated by renaming)
- WAW/WAR only occur in out-of-order execution

---

## Advanced Pipelining Techniques Summary

| Technique | What It Reduces |
|-----------|-----------------|
| Forwarding and bypassing | Data hazard stalls |
| Delayed branches | Control hazard stalls |
| Dynamic scheduling (scoreboarding) | RAW stalls |
| Dynamic scheduling (register renaming) | WAR & WAW stalls |
| Branch prediction | Control hazard stalls |
| Multiple issue per cycle | Ideal CPI |
| Hardware speculation | Data & control stalls |
| Dynamic memory disambiguation | Memory data hazard stalls |
| Loop unrolling | Control hazard stalls |
| Compiler pipeline scheduling | Data hazard stalls |
| Compiler dependence analysis | Ideal CPI, data stalls |
| Software pipelining | Ideal CPI, data & control stalls |

---

## Loop Unrolling

### Original Loop
```c
for (i=999; i>=0; i=i-1)
    x[i] = x[i] + s;
```

### RISC-V Code (with stalls)
```assembly
Loop: fld    f0, 0(x1)      # Load x[i]
      stall                 # Load-use stall
      fadd.d f4, f0, f2     # Add scalar s
      stall                 # 2 stalls for FP
      stall
      fsd    f4, 0(x1)      # Store result
      addi   x1, x1, -8     # Decrement pointer
      bne    x1, x2, Loop   # Branch
```
**8 clock cycles per iteration**

### After Instruction Scheduling
```assembly
Loop: fld    f0, 0(x1)      # Load
      addi   x1, x1, -8     # Move addi up
      fadd.d f4, f0, f2     # Add
      stall                 # 2 stalls remain
      stall
      fsd    f4, 8(x1)      # Store (offset changed to 8)
      bne    x1, x2, Loop   # Branch
```
**7 clock cycles per iteration**

### Unrolled 4 Times (without scheduling)
**26 cycles for 4 iterations = 6.5 cycles/iteration**

### Unrolled 4 Times + Scheduled
```assembly
Loop: fld     f0, 0(x1)
      fld     f6, -8(x1)
      fld     f10, -16(x1)
      fld     f14, -24(x1)
      fadd.d  f4, f0, f2
      fadd.d  f8, f6, f2
      fadd.d  f12, f10, f2
      fadd.d  f16, f14, f2
      fsd     f4, 0(x1)
      fsd     f8, -8(x1)
      fsd     f12, -16(x1)
      fsd     f16, -24(x1)
      addi    x1, x1, -32
      bne     x1, x2, Loop
```
**14 cycles for 4 iterations = 3.5 cycles/iteration**
- Requires more registers (9 FP vs 3 FP originally)

### Three Unrolling Considerations
1. **Amdahl's Law**: Decreasing overhead amortized with each unrolling
2. **Code size growth**: May increase instruction cache miss rate
3. **Register pressure**: May exhaust available registers

---

## Branch Prediction

### Static Branch Prediction
- **Delayed branch**: Reorder code around branches
- **Predict as taken**: Average misprediction = 34% (untaken frequency)
- **Profile-based**: Use program profiling for better accuracy

### Why Prediction Works
- Underlying algorithms have regularities
- Data has regularities
- Instruction sequences have redundancies

### Dynamic Branch Prediction Components

**Branch Prediction Buffer (BPB)** / Branch History Table (BHT)
- Low-order n bits of branch address index a table
- Each entry has k bits of history (k = 1, 2, or larger)
- May have collisions between distant branches

### 1-bit Branch Prediction
**Two states**:
- Bit = 1: Last time taken, predict taken
- Bit = 0: Last time not taken, predict not taken

**Problem**: Makes 2 mistakes per loop (first and last iterations)

**Example**:
```
Loop 1: for (i=1; i<=100; i++) {
    Loop 2: for (k=1; k<=5; k++) { ... }
}
```
Inner loop causes mispredictions: T NT T NT (alternating)

### 2-bit Branch Prediction
**Four states** (00, 01, 10, 11):
- Prediction mirrors most recent run of 2
- Only 1 misprediction per loop execution (on last iteration)

**State Transition Diagram**:
```
State 0 (00) ←→ State 1 (01)
     ↕              ↕
State 3 (11) ←→ State 2 (10)

Predict Not Taken: States 00, 01
Predict Taken: States 10, 11
```

**Branch taken**: Move right/up
**Branch not taken**: Move left/down

### n-bit Branch Prediction
- Entry = integer in [0, 2^n - 1]
- Taken: entry = min(entry+1, 2^n - 1)
- Not taken: entry = max(entry-1, 0)
- Predict taken if entry ≥ 2^(n-1)

**Empirically**: Not much better than 2-bit

### Correlated Prediction

**Concept**: Different predictions based on previous branch outcomes

**Notation**: (m,n) predictor
- **m**: Number of previous branches considered
- **n**: Bits of history per predictor
- Total predictors: 2^m (one for each pattern of m branches)

**Example**: (1,1) predictor
- 2 predictors based on last branch (taken/not taken)

**Example**: (2,2) predictor
- 4 predictors based on last 2 branches
- Each is a 2-bit predictor
- Access using: branch address concatenated with 2-bit shift register

### Tournament Predictor
**Combines**:
- Local predictor (looks at specific branch history)
- Global predictor (looks at recent branch patterns)

**Selection**:
- 2-bit counter selects which predictor to use
- Changes predictor after 2 consecutive mispredicts
- Counter states 0,1 → use predictor 1; states 2,3 → use predictor 2

**FSM**:
```
If predictor 1 correct: counter = min(counter+1, 3)
If predictor 2 correct: counter = max(counter-1, 0)
```

### Predictor Size Example
1024 entry buffer with:
- 2-bit local: 2 bits
- (3,2) global: 2^3 × 2 = 16 bits
- 2-bit tournament: 2 bits
- Target address: 32 bits
- **Total**: (2 + 16 + 2 + 32) × 1024 = 53,248 bits = 53 Kbits

---

## Dynamic Scheduling: Tomasulo's Algorithm

### Three Key Components
1. **Dynamic Scheduling**: Execute instructions out-of-order
2. **Register Renaming**: Eliminate WAR/WAW hazards
3. **Dynamic Memory Disambiguation**: Resolve memory dependencies at runtime

### Register Renaming Example
**Before** (has WAR and WAW):
```
fdiv.d  f0, f2, f4
fadd.d  f6, f0, f8
fsd     f6, 0(x1)
fsub.d  f8, f10, f14    # WAR on f8
fmul.d  f6, f10, f8     # WAW on f6
```

**After** (renamed):
```
fdiv.d  f0, f2, f4
fadd.d  S, f0, f8       # Rename f6 → S
fsd     S, 0(x1)
fsub.d  T, f10, f14     # Rename f8 → T
fmul.d  f6, f10, T      # Use renamed T
```

### Dynamic Memory Disambiguation
**Question**: Are these dependent?
```
sd  100(x1), x6
ld  x7, 36(x2)
```
**Answer**: Depends on whether 100+x1 = 36+x2
- Cannot determine at compile time
- Hardware checks after effective address calculation

### Tomasulo Components

**Reservation Stations (RS)**:
- Buffer operands while waiting for execution units
- Provide extra renaming registers
- Dynamically avoid WAW/WAR hazards

**Issue Logic**:
- Renames instruction outputs to RS slots
- Results go directly to RSs (not through register file)

**Distributed Hazard Detection**:
- Handled separately by each functional unit

**Load/Store Buffers**:
- Queue memory access requests

### Reservation Station Fields
- **Op**: Operation to perform on S1 and S2
- **Qj, Qk**: RS slots that will produce S1, S2 (source RS)
- **Vj, Vk**: Values of S1, S2 (if available)
- **Busy**: RS and execution unit occupied
- **A**: Memory address (for load/store)

**Register File / Store Buffer**:
- **Qi**: RS slot containing operation whose result goes here

### Three Major Steps

**1. Issue**
- Get instruction from queue
- If RS slot available, send instruction; else stall
- Send operand values if available, else note operand names
- Rename destination register

**2. Execute**
- Monitor Common Data Bus (CDB) for operands
- When all operands available, begin execution
- No instruction dispatched until data operands available

**3. Write Result**
- When result ready and CDB free, write to CDB
- From CDB to registers and RS slots waiting for this result

### Drawbacks
- Complex, requires significant hardware
- Difficult to do associative access to many RS entries at high speed
- CDB can be a limiting factor (multiple CDBs add overhead)

### When Tomasulo is Most Useful
- Running binaries for earlier pipeline implementations
- Code difficult to schedule statically
- Many dynamically-resolved memory dependencies
- Insufficient programmer-visible registers for static renaming
- Many functional units (scoreboarding bottleneck)

---

## Hardware-Based Speculation

### Three Components
1. **Dynamic branch prediction**: Choose which instructions to execute
2. **Speculation**: Execute before control dependencies resolved
   - Must be able to undo incorrectly speculated sequences
3. **Dynamic scheduling**: Schedule different basic block combinations

### Adding Speculation to Tomasulo

**Reorder Buffer (ROB)**:
- Holds results of instructions that finished execution but haven't committed
- Passes results among speculated instructions
- Extends architectural registers like RS extends register file

**Four ROB Entry Fields**:
1. **Instruction Type**: Branch, store, or ALU/load
2. **Destination**: Register number or memory address
3. **Value**: Result value until instruction commits
4. **Ready**: Execution completed, value ready

### Four Steps of Speculative Tomasulo

**1. Issue (Dispatch)**
- Get instruction from queue
- If RS and ROB slots free, issue instruction
- Send operands and ROB number for destination

**2. Execute (EX)**
- When operands ready, execute
- Monitor CDB for missing operands
- Checks RAW hazards

**3. Write Result (WB)**
- Write to CDB → all awaiting units and ROB
- Mark RS available

**4. Commit (Graduate)**
- When instruction at ROB head and result present:
  - Update register/memory with result
  - Remove from ROB
- Mispredicted branch flushes ROB

### Avoiding Memory Hazards with Speculation
- **WAW/WAR through memory**: Eliminated (memory updates in order)
- **RAW through memory**: Prevented by:
  1. Load cannot execute if active ROB store has matching address
  2. Load effective address computed in program order w.r.t. earlier stores

### Exceptions and Interrupts
- **Imprecise interrupts**: IBM 360/91 (not popular with programmers)
- **Precise exceptions**: In-order commit
- Exception recorded in ROB, raised only when instruction commits
- Allows rollback on incorrect speculation

### Alternative: Register Renaming
- Larger physical register set instead of ROB
- Issue maps architectural registers to physical registers
- Allocate new unused register for destination (avoids WAW/WAR)
- Physical register doesn't become architectural until commit
- **Most modern OoO processors use this approach**

---

## Multiple Issue Processors

### Three Flavors

**1. Statically-Scheduled Superscalar**
- In-order execution
- Compiler schedules instructions
- Issue varying number of instructions per clock

**2. Dynamically-Scheduled Superscalar**
- Out-of-order execution
- Hardware schedules instructions
- Issue varying number of instructions per clock

**3. VLIW (Very Long Instruction Word)**
- Fixed number of instructions per "packet"
- Parallelism explicitly indicated
- Compiler responsible for scheduling

### Comparison Table

| Feature | Superscalar (Static) | Superscalar (Dynamic) | VLIW |
|---------|---------------------|----------------------|------|
| **Issue Structure** | Dynamic | Dynamic | Static |
| **Hazard Detection** | Hardware | Hardware | Software |
| **Scheduling** | Static (compiler) | Dynamic (hardware) | Static (compiler) |
| **Speculation** | Software | Hardware | Software |
| **Examples** | ARM Cortex-A9 | Intel Core | IA-64 (Itanium) |

### VLIW Characteristics

**Advantages**:
- Simple hardware
- High performance potential
- Compiler has global view

**Disadvantages**:
- Large code size (unused operations = wasted bits)
- Binary compatibility issues (different #FUs need different code)
- Lock-step operation (stall in one unit stalls all)
- Cache behavior hard to predict at compile time

**VLIW Example** (7 operations unrolled):
- 9 cycles for 7 loop iterations = 1.3 cycles/iteration
- 2.5 operations per clock (50% efficiency)
- Requires 15 registers vs 9 for superscalar

### Intel IA-64 (EPIC - Explicitly Parallel Instruction Computer)

**Features**:
- 128 64-bit integer registers
- 128 82-bit floating-point registers
- Hardware dependency checking
- Control and data speculation
- Predicated execution (reduces mispredictions by 40%)

**Itanium** (2001):
- 6-wide, 10-stage pipeline at 800MHz

**Itanium 2** (2005):
- 6-wide, 8-stage pipeline at 1666MHz
- 32KB I&D, 128KB L2, 9216KB L3

---

## High-Performance Instruction Delivery

### Branch-Target Buffer (BTB)

**Purpose**: Predict next instruction address during fetch (not after decode)

**Operation**:
1. Branch PC indexes BTB
2. If hit: Get predicted target address immediately
3. If miss: Normal PC+4

**BTB Entry Contains**:
- Valid bit
- Branch prediction state
- Predicted target address

**Penalties**:
- Instruction not in BTB, branch not taken: 0 cycles
- Instruction not in BTB, branch taken: 2 cycles (must compute target)
- Mispredicted direction: 2 cycles

**Example Penalty Calculation**:
- Misprediction penalty: 2 cycles
- Prediction accuracy: 80%
- BTB hit rate (for taken branches): 90%
- Branch predicted taken but not taken: 90% × 20% = 0.18
- Branch taken but not in buffer: 10%
- **Total penalty**: (0.18 + 0.10) × 2 = 0.56 cycles/branch

### Integrated Instruction Fetch Unit

**Features**:
- Integrated branch predictor (part of fetch unit)
- Instruction prefetch and buffering
- Fetch multiple instructions (may require multiple cache lines)
- **Branch folding**: Store target instructions instead of addresses
  - Zero-cycle branches possible
  - Substitute destination instruction for branch in pipeline

### Return Address Predictors

**Problem**: Predict register/indirect branches from:
- Function calls/returns
- Switch statements
- Procedure returns

**Solutions**:
1. BTB (works but not optimal for returns)
2. **Return address stack** (CPU-internal)
   - Push return address on call
   - Pop on return
   - Much more accurate than BTB for returns

### Value Prediction

**Concept**: Predict value produced by instruction
- Focus on loads with infrequent value changes
- Mixed results in research
- Related: Address aliasing prediction (predict if addresses conflict)

---

## Performance Comparison: Speculation vs Non-Speculation

### Example: 2-issue Superscalar with Integer and FP Units

**Loop**:
```assembly
Loop: ld   x2, 0(x1)     # Load
      addi x2, x2, #1    # Increment
      sd   x2, 0(x1)     # Store
      addi x1, x1, #4    # Pointer increment
      bne  x2, x3, Loop  # Branch
```

**Without Speculation**: 5 cycles per iteration
**With Speculation**: 3.5 cycles per iteration (40% improvement)

**Key Difference**:
- Non-speculation: Must wait for branch resolution
- Speculation: Execute past branches, commit in order

---

## Key Concepts Summary

### ILP Exploitation Hierarchy
1. **Basic**: Pipelining, forwarding, hazard detection
2. **Intermediate**: Branch prediction, delayed branches, loop unrolling
3. **Advanced**: Dynamic scheduling, speculation, multiple issue
4. **Cutting-edge**: Tournament prediction, value prediction, VLIW/EPIC

### Critical Design Trade-offs

**Hardware Complexity vs Performance**:
- Tomasulo: Complex but high performance
- Scoreboarding: Simpler but limited by WAR/WAW
- VLIW: Simple hardware but complex compiler

**Register Pressure**:
- Loop unrolling requires more registers
- Tomasulo provides virtual registers via RS
- VLIW/IA-64 has large register files

**Speculation Overhead**:
- Energy cost of wrong-path execution
- Must be able to undo incorrect speculation
- ROB provides mechanism for precise exceptions

**Code Size**:
- Loop unrolling increases code
- VLIW has large instructions with unused operations
- Dynamic scheduling has no code size impact

### Important Performance Factors

1. **Branch prediction accuracy** (>95% needed for high ILP)
2. **Memory latency** (load-use dependencies limit ILP)
3. **Register renaming** (eliminates false dependencies)
4. **Instruction fetch bandwidth** (must supply multiple inst/cycle)
5. **Commit bandwidth** (must commit multiple inst/cycle)

### Modern Processor Techniques

**Most use combination of**:
- Dynamic scheduling (out-of-order execution)
- Register renaming (physical register file)
- Hardware speculation (ROB or similar)
- Tournament branch prediction
- BTB with return address stack
- Multiple issue (2-6 instructions/cycle)
- Deep pipelines (10-20+ stages)

---

## Quick Reference: Latencies Used in Examples

| Operation | Latency (cycles) | Stalls (cycles) |
|-----------|------------------|-----------------|
| FP ALU → FP ALU | 4 | 3 |
| FP ALU → Store | 3 | 2 |
| Load → FP ALU | 1 | 1 |
| Load → Store | 1 | 0 |
| Integer → Integer | 1 | 0 |

---

## Important Terminology

- **IPC**: Instructions Per Cycle (>1 with multiple issue)
- **CPI**: Cycles Per Instruction (want <1 with multiple issue)
- **Superscalar**: Issue varying number of instructions per cycle
- **VLIW**: Issue fixed number of instructions per packet
- **EPIC**: Explicitly Parallel Instruction Computer (IA-64)
- **ROB**: Reorder Buffer (for speculation)
- **RS**: Reservation Station (for Tomasulo)
- **CDB**: Common Data Bus (for broadcasting results)
- **BTB**: Branch Target Buffer (for branch prediction)
- **BHT**: Branch History Table (same as BPB)
- **Commit**: Make instruction results permanent (speculation)
- **Issue**: Send instruction to execution unit
- **Dispatch**: Same as issue in some contexts
- **Graduate**: Same as commit in some contexts
