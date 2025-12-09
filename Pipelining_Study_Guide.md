# Pipelining - Study Guide

## Key Formulas

**Clock Period = Max{time delay of stage} + overhead (skew, latch delay)**

**Speedup = (k × n) / (k + n - 1) ≈ k when n >> k**
- k = number of pipeline stages
- n = number of instructions

**Efficiency = Actual Speedup / Ideal Speedup**

**Throughput = Number of instructions completed per unit time**

**Pipeline CPI = Ideal CPI + Structural Stalls + Data Stalls + Control Stalls**

**Speedup with Stalls = Pipeline Depth / (1 + Pipeline stall cycles per instruction)**

**Branch Impact = Pipeline Depth / (1 + Branch freq × Mispredict rate × Branch penalty)**

---

## Pipelining Basics

### Definition
**Pipelining**: Implementation technique where multiple instructions are overlapped in execution

**Key Concept**: Divide logic into shallow slices (pipeline stages) with uniform delays
- New input can be provided as soon as quick, shallow network finishes
- Clock cycle only as long as slowest stage

### Five Classic RISC Pipeline Stages

1. **IF (Instruction Fetch)**: Fetch instruction from memory
   - IR = IM[PC]
   - PC = PC + 4

2. **ID (Instruction Decode)**: Decode instruction and read registers
   - Decode opcode
   - Read source operands from register file

3. **EX (Execute)**: Execute operation or calculate address
   - ALU operation
   - Effective address calculation
   - Branch target calculation

4. **MEM (Memory Access)**: Access memory operand
   - Load from memory
   - Store to memory

5. **WB (Write Back)**: Write result to register
   - Update destination register

### Advantages of Pipelining
- Increases instruction throughput
- Each instruction still takes k cycles
- But completes 1 instruction per cycle (ideally)
- Speedup ≈ k for large number of instructions

### Limits of Pipelining
- At least 1 layer of logic gates per stage
- Practical minimum: 2-10 gates per stage
- Overhead from pipeline registers and clock skew
- Commercial designs approaching theoretical limits

---

## Three Types of Pipeline Hazards

### 1. Structural Hazards
**Definition**: Not enough hardware resources to keep all instructions moving

**Example**: Single memory port
- Instruction fetch (IF stage) needs memory
- Load/Store (MEM stage) needs memory simultaneously
- Cannot both access memory in same cycle

**Solutions**:
- Stall pipeline (insert bubble)
- Add more hardware resources (separate I-cache and D-cache)
- Schedule instructions to avoid conflicts

### 2. Data Hazards
**Definition**: Instruction depends on result of earlier instruction still in pipeline

**Three Types**:

| Hazard | Description | Occurs When | Solution |
|--------|-------------|-------------|----------|
| **RAW** | Read After Write | j reads before i writes | Forwarding, stalling |
| **WAW** | Write After Write | j writes before i writes | Only with multiple write stages |
| **WAR** | Write After Read | j writes before i reads | Only with out-of-order writes |

**Note**: In simple 5-stage in-order pipeline, only RAW hazards are possible

**RAW Example**:
```assembly
add x1, x2, x3    # x1 written in WB stage
sub x4, x1, x5    # x1 needed in EX stage
```

**Solutions**:

**a) Data Forwarding (Bypassing)**
- Forward result from later pipeline stage to earlier stage
- EX/MEM → EX: Forward ALU result to next instruction's ALU input
- MEM/WB → EX: Forward memory data to ALU input
- Reduces stalls but doesn't eliminate them

**b) Pipeline Stalling**
- When forwarding cannot solve hazard (load-use hazard)
- Insert bubble (NOP) into pipeline
- Example:
```assembly
lw  x2, 0(x1)     # Data available at MEM/WB
add x3, x2, x4    # Needs data at ID/EX → 1 cycle stall required
```

**c) Compiler Scheduling**
- Reorder instructions to avoid hazards
- Example:
```assembly
# Before (stall):
lw  x2, 0(x4)
add x1, x2, x3    # Stall here

# After (no stall):
lw  x2, 0(x4)
lw  x5, 4(x4)     # Independent instruction
add x1, x2, x3    # No stall
```

### 3. Control Hazards (Branch Hazards)
**Definition**: Don't know which instruction to fetch next due to pending branch

**Problem**: Branch outcome not known until EX stage (or later)
- Must stall or guess which instruction to fetch next

**Solutions**:

**a) Stall (Freeze Pipeline)**
- Wait until branch outcome known
- Simple but slow
- 1-3 cycle penalty per branch

**b) Predict Not Taken**
- Continue fetching sequentially
- If branch actually taken, flush incorrectly fetched instructions
- Works well for forward branches (if-then-else)

**c) Predict Taken**
- Fetch from branch target
- Requires computing target early
- Works well for backward branches (loops)

**d) Delayed Branch**
- Always execute instruction(s) after branch (delay slots)
- Compiler fills delay slot with useful instruction
- Three scheduling options:
  1. **From before**: Independent instruction before branch
  2. **From target**: Instruction from branch target (if safe)
  3. **From fall-through**: Instruction after branch (if safe)

**e) Dynamic Branch Prediction**
- Use branch history to predict
- See separate section below

---

## Branch Prediction

### Static Branch Prediction

**Predict Always Not Taken**:
- Simplest scheme
- Average misprediction = branch taken frequency ≈ 34%

**Compiler-Based Static Prediction**:
- **Option 1**: Forward branches not taken, backward branches taken
  - Rationale: Loops iterate (backward), if-then-else often false (forward)
- **Option 2**: Profile-based prediction
  - Use program profiling data
  - More accurate than simple heuristics

### Dynamic Branch Prediction

**Branch Prediction Buffer (BPB)** / Branch History Table (BHT)
- Small memory indexed by low-order bits of branch PC
- Each entry stores prediction state
- Updated after each branch execution

**1-bit Predictor**:
- Two states: Taken (T) or Not Taken (NT)
- Last outcome predicts next outcome

**State Transition**:
```
Taken → T state → Predict Taken
Not Taken → NT state → Predict Not Taken
```

**Problem**: Two mispredictions per loop
- First iteration (when entering loop)
- Last iteration (when exiting loop)

**Example**: Inner loop iterations
```
Pattern: T T T T T NT (exit)
Predictions: NT T T T T T
Mispredictions: First and last iterations
```

**2-bit Predictor** (Bimodal Predictor):
- Four states: 00, 01, 10, 11
- Requires two consecutive mispredictions to change prediction

**State Machine**:
```
       00 (Strong NT) ←→ 01 (Weak NT)
            ↕                  ↕
       11 (Strong T)  ←→ 10 (Weak T)

Predict NT: States 00, 01
Predict T: States 10, 11

Taken: Move right/up
Not Taken: Move left/down
```

**Advantage**: Only 1 misprediction per loop (on exit)

**n-bit Predictor**:
- Counter: 0 to 2^n - 1
- Increment on taken (up to max)
- Decrement on not taken (down to 0)
- Predict taken if counter ≥ 2^(n-1)

**Empirical Result**: Little improvement beyond 2 bits

### Branch Prediction Performance

**4096-entry 2-bit BPB**: 1-18% misprediction rate on SPEC89
- Varies widely by benchmark
- Integer programs: typically 5-15%
- FP programs: typically 1-5%

**Ways to Improve**:
1. Increase buffer size (diminishing returns beyond 4K entries)
2. Use correlated predictors (see ILP study guide)
3. Use tournament predictors (combine local and global)

---

## Early Branch Resolution

### Original Design
- Branch decision in EX stage (3rd stage)
- 2 cycles penalty for taken branch

### Optimization: Move to ID Stage
**Add hardware to ID stage**:
- Equality comparator for branch condition
- Branch target adder (PC + offset)

**Benefits**:
- Branch decision in ID (2nd stage)
- Reduces penalty to 1 cycle
- 50% reduction in branch penalty

**Additional Forwarding Required**:
- Forward to ID stage for branch comparison
- More complex hazard detection logic

---

## Scheduling the Branch Delay Slot

### Three Options

**1. From Before**
```assembly
# Before:
    add  x1, x2, x3
    beq  x1, x0, L
    sub  x4, x5, x6

# After:
    beq  x1, x0, L
    add  x1, x2, x3    # Moved into delay slot
    sub  x4, x5, x6
```
- Always safe
- Best option when available

**2. From Target**
```assembly
# Before:
    beq  x1, x0, L
    sub  x4, x5, x6
L:  add  x7, x8, x9

# After:
    beq  x1, x0, L
    add  x7, x8, x9    # From target
L:  ...
```
- Only if instruction always executed when branch taken
- Must be safe if branch not taken

**3. From Fall-Through**
```assembly
# Before:
    beq  x1, x0, L
    add  x7, x8, x9    # Fall-through path
    ...

# After:
    beq  x1, x0, L
    add  x7, x8, x9    # In delay slot
    ...
```
- Only if instruction always executed when branch not taken
- Must be safe if branch taken

**Success Rate**: 50-70% of delay slots can be usefully filled

---

## Pipeline Implementation Details

### Pipeline Registers
**Five inter-stage registers**:
1. **IF/ID**: Instruction, PC+4
2. **ID/EX**: Control signals, register values, immediate, PC+4
3. **EX/MEM**: Control signals, ALU result, register value, branch target
4. **MEM/WB**: Control signals, memory data or ALU result
5. **PC**: Program counter

### Hazard Detection Unit

**Detects Load-Use Hazard**:
```
if (ID/EX.MemRead and
    ((ID/EX.RegisterRd = IF/ID.RegisterRs1) or
     (ID/EX.RegisterRd = IF/ID.RegisterRs2)))
    Stall pipeline
```

**Actions**:
- Stall IF and ID stages
- Insert bubble (NOP) in EX stage
- Prevent IF/ID and PC from updating

### Forwarding Unit

**EX Hazard** (from EX/MEM):
```
if (EX/MEM.RegWrite and
    EX/MEM.RegisterRd ≠ 0 and
    EX/MEM.RegisterRd = ID/EX.RegisterRs1)
    ForwardA = 10 (forward from EX/MEM)

if (EX/MEM.RegWrite and
    EX/MEM.RegisterRd ≠ 0 and
    EX/MEM.RegisterRd = ID/EX.RegisterRs2)
    ForwardB = 10 (forward from EX/MEM)
```

**MEM Hazard** (from MEM/WB):
```
if (MEM/WB.RegWrite and
    MEM/WB.RegisterRd ≠ 0 and
    not EX_hazard and
    MEM/WB.RegisterRd = ID/EX.RegisterRs1)
    ForwardA = 01 (forward from MEM/WB)

if (MEM/WB.RegWrite and
    MEM/WB.RegisterRd ≠ 0 and
    not EX_hazard and
    MEM/WB.RegisterRd = ID/EX.RegisterRs2)
    ForwardB = 01 (forward from MEM/WB)
```

**Forwarding Mux Control**:
- 00: Use register file value
- 01: Forward from MEM/WB
- 10: Forward from EX/MEM

---

## Exceptions and Interrupts

### Types of Exceptions

| Type | Example | Synchronous? | User Requested? | Maskable? | Within/Between Inst? | Resume? |
|------|---------|--------------|-----------------|-----------|---------------------|---------|
| **I/O interrupt** | Disk ready | No | No | Yes | Between | Yes |
| **Invoke OS** | System call | Yes | Yes | No | Between | Yes |
| **Tracing** | Debug breakpoint | Yes | Yes | No | Between | Yes |
| **Integer overflow** | Add overflow | Yes | No | No | Within | Yes |
| **FP overflow** | FP result too large | Yes | No | No | Within | Yes |
| **Page fault** | TLB miss | Yes | No | No | Within | Yes |
| **Undefined inst** | Bad opcode | Yes | No | No | Within | No |
| **Hardware fault** | Parity error | No | No | No | Within | No |
| **Power failure** | Power loss | No | No | No | Within | No |

### Exception Handling Requirements

**Precise Exception**:
1. All instructions before faulting instruction complete
2. Faulting instruction and later instructions can be restarted
3. No side effects from later instructions

**Implementation**:
- Save PC of faulting instruction
- Clear pipeline behind faulting instruction
- Force trap instruction into pipeline
- Exception handler deals with exception
- Return and restart from saved PC

### Out-of-Order Exceptions

**Problem**:
```assembly
ld   x1, 0(x2)    # IF  ID  EX  MEM WB
add  x3, x4, x5   #     IF  ID  EX  MEM WB
```
- add may fault in IF before ld faults in MEM
- Cannot simply restart at add's PC!

**Solution**:
- Status vector carried through pipeline
- Note exception but disable writes for that instruction
- Resolve exceptions at late stage (WB)
- Ensures in-order exception handling

### Imprecise vs Precise

**Imprecise**:
- Only handle common cases correctly
- Statistical guarantee through testing
- Up to 10x faster
- Difficult for debugging

**Precise**:
- Handle all exception combinations correctly
- Required for some systems
- Easier for integer than floating-point
- Essential for debugging

---

## Multi-Cycle Operations (Floating-Point)

### Why Multi-Cycle?

**Floating-point operations are complex**:
- Require multiple stages
- Different operations have different latencies
- Example: FP divide takes much longer than FP add

### Key Terms

**Latency**: How long an instruction waits for the result
**Initiation Interval**: How long to wait before issuing same type of instruction

### Typical FP Latencies

| Operation | Latency | Initiation Interval | Stages |
|-----------|---------|---------------------|--------|
| **FP Add** | 4 | 1 | 4 |
| **FP Multiply** | 7 | 1 | 7 |
| **FP Divide** | 25 | 25 | 25 (non-pipelined) |

**Note**: Initiation interval = 1 means fully pipelined

### Four-Stage FP Adder

**Stage 1: Align**
- Compare exponents
- Shift mantissa of smaller number

**Stage 2: Add**
- Add/subtract mantissas

**Stage 3: Normalize**
- Normalize result
- Check for overflow/underflow

**Stage 4: Round**
- Round result
- Renormalize if necessary

### FP Addition Example (Decimal)

```
9.999 × 10^1 + 1.610 × 10^-1

1. Align: 9.999 × 10^1 + 0.016 × 10^1
2. Add: 10.015 × 10^1
3. Normalize: 1.0015 × 10^2
4. Round: 1.002 × 10^2
```

### Issues with Multi-Cycle FP Operations

**New Hazards**:
1. **Structural Hazards**:
   - Multiple instructions may want MEM or WB stage simultaneously
   - Non-pipelined divider causes structural hazard

2. **Out-of-Order Completion**:
   - Shorter operations finish before longer ones
   - Creates potential for WAW hazards
   
3. **Extended RAW Hazards**:
   - Longer stalls waiting for FP results

**Example Out-of-Order Completion**:
```assembly
fmul.d f0, f2, f4    # IF ID M1 M2 M3 M4 M5 M6 M7 ME WB (11 cycles)
fadd.d f6, f0, f8    #    IF ID A1 A2 A3 A4 ME WB (8 cycles)
fld    f10, 0(x1)    #       IF ID EX ME WB (5 cycles)
```
- fld completes before fadd
- fadd completes before fmul

**WAW Hazard Example**:
```assembly
fmul.d f0, f2, f4
fadd.d f0, f6, f8    # Both write to f0 - order matters!
```

**Solution**: Issue stage checks for structural hazards and WAW hazards
- Stall instruction at issue if hazard detected
- Ensures writes occur in program order

---

## MIPS R4000 Pipeline

### Eight-Stage "SuperPipeline"

1. **IF**: Instruction cache fetch (first half)
2. **IS**: Instruction cache fetch (second half)
3. **RF**: Register fetch, decode, hazard check
4. **EX**: Execution (ALU, EA calc, branch target)
5. **DF**: Data cache access (first half)
6. **DS**: Data cache access (second half)
7. **TC**: Tag check (cache hit?)
8. **WB**: Write back

### Load Delay: 2 Cycles

**Timing**:
```assembly
ld   x1, 0(x2)    # IF IS RF EX DF DS TC WB
add  x3, x1, x4   #       IF IS RF EX DF DS TC WB (stall 2 cycles)
```

**Why 2 cycles?**
- Data available at TC stage
- Needed at EX stage of dependent instruction
- TC to EX = 2 cycle gap

### Branch Delay: 3 Cycles

**Timing**:
- Branch decision in EX stage (stage 4)
- Next instruction fetch in IF stage (stage 1)
- Gap of 3 stages

**Impact on CPI** (assuming ideal CPI = 1):
- Branch frequency: 14%
- Predict not taken accuracy: 65%
- Branch penalty: 3 cycles
- **CPI = 1 + (0.14 × 0.35 × 3) = 1.147**

### R4000 FP Unit

**Stages**:
- **U**: Unpack FP numbers
- **A**: Mantissa ADD
- **R**: Round
- **S**: Operand shift
- **E**: Exception test
- **M**: Multiply first stage
- **N**: Multiply second stage
- **D**: Divide stage

**Operation Latencies**:

| Operation | Stages | Latency | Initiation |
|-----------|--------|---------|------------|
| **FP Add** | U+S+A+A+R+S | 6 | 3 |
| **FP Multiply** | U+E+M+M+M+M+M+M+N+N+A+R | 12 | 8 |
| **FP Divide** | U+A+R + D (iterations) | 112 | 112 |

---

## Scoreboarding (Dynamic Scheduling)

### Purpose
- Allow out-of-order execution to maximize performance
- Issue instructions when resources available and data ready
- Avoid structural, RAW, WAR, and WAW hazards dynamically

### Five Pipeline Stages with Scoreboard

1. **IF**: Instruction fetch
2. **ID/Issue**: Decode and issue when no structural/WAW hazards
3. **RO (Read Operands)**: Wait until no RAW hazards, then read
4. **EX**: Execute (may be multi-cycle)
5. **WB**: Write result when no WAR hazards

### Three Scoreboard Tables

**1. Instruction Status**
- Which stage each instruction is in

**2. Functional Unit Status** (for each FU):
- **Busy**: Is unit busy?
- **Op**: Operation being performed
- **Fi**: Destination register
- **Fj, Fk**: Source registers
- **Qj, Qk**: FUs producing Fj, Fk (0 if ready)
- **Rj, Rk**: Are Fj, Fk ready? (Yes/No)

**3. Register Result Status** (for each register):
- **Result**: Which FU will write to this register (0 if none)

### Hazard Detection

**Issue Stage**:
- Structural: Check if FU is busy
- WAW: Check if register result status shows another FU writing to Fi

**Read Operands Stage**:
- RAW: Wait until Rj=Yes and Rk=Yes (operands ready)

**Write Result Stage**:
- WAR: Wait until all earlier instructions have read their operands
  - Check if any FU has Qj=this FU or Qk=this FU

### Scoreboard Example

**Code**:
```assembly
fld   f6, 0(x1)
fld   f2, 0(x2)
fmul.d f0, f2, f4
fsub.d f8, f6, f2
fdiv.d f10, f0, f6
fadd.d f6, f8, f2
```

**Key Events**:
1. fmul.d must wait for f2 from second fld
2. fsub.d can execute when f6 and f2 ready
3. fdiv.d must wait for f0 from fmul.d
4. fadd.d has WAR hazard with fdiv.d on f6
   - Must wait for fdiv.d to read f6 before writing new f6

### Limitations

**Scoreboard Limitations**:
- No forwarding (results through register file)
- Limited by number of FUs
- Centralized issue logic can be bottleneck
- WAR/WAW hazards still cause stalls

**Improvements** (covered in Tomasulo):
- Register renaming eliminates WAR/WAW
- Reservation stations provide forwarding
- Distributed issue logic

---

## ISA Design Impact on Pipelining

### Features that Complicate Pipelining

**1. Variable Instruction Lengths**
- Different decode times
- Complicates IF and ID stages
- Makes hazard detection harder

**2. Complex Addressing Modes**
- Post-autoincrement: WAR/WAW hazards
- Multiple-indirect: Complex timing
- Makes pipeline stages non-uniform

**3. Self-Modifying Code**
- Instruction in pipeline may be overwritten
- Requires instruction cache invalidation
- Major complication

**4. Implicit Condition Codes**
- Every ALU operation sets flags
- Creates WAR hazards
- Complicates restart after exceptions

### RISC Design Principles for Pipelining

**Simple, uniform instruction format**:
- Fixed length (32 bits)
- Regular encoding
- Easy decode

**Load-store architecture**:
- Memory access only through load/store
- Simplifies pipeline (MEM stage predictable)

**Simple addressing modes**:
- Register + offset
- No complex calculations in addressing

**Explicit condition codes**:
- Branch tests registers directly
- No implicit flags

---

## Performance Analysis

### Example: Effect of Stalls

**Given**:
- 5-stage pipeline
- 30% loads, 1-cycle load-use penalty
- 15% branches, 50% taken, 2-cycle penalty

**Load stalls**: 0.30 × 1.0 = 0.30 cycles
**Branch stalls**: 0.15 × 0.5 × 2.0 = 0.15 cycles
**Total stalls**: 0.45 cycles
**CPI**: 1.0 + 0.45 = 1.45

**Speedup**: 5 / 1.45 = 3.45 (vs 5 without stalls)

### Example: Branch Prediction Impact

**R4000 Pipeline**:
- Branch penalty: 3 cycles
- Branch frequency: 14%
- Misprediction rate: 35% (predict not taken)

**Without prediction** (stall): CPI = 1 + 0.14 × 3 = 1.42
**With prediction**: CPI = 1 + 0.14 × 0.35 × 3 = 1.147
**Improvement**: 1.42 / 1.147 = 1.24 (24% faster)

---

## Key Concepts Summary

### Pipeline Performance Factors
1. **Pipeline depth**: Deeper = faster clock but more hazards
2. **Hazard frequency**: Fewer hazards = better performance
3. **Hazard penalty**: Shorter penalty = less impact
4. **Forwarding**: Reduces data hazard stalls
5. **Branch prediction**: Reduces control hazard stalls

### Hazard Solutions Summary

| Hazard Type | Solutions |
|-------------|-----------|
| **Structural** | Add hardware, stall, schedule |
| **Data (RAW)** | Forwarding, stalling, scheduling |
| **Control** | Prediction, delay slots, early resolution |

### Trade-offs
- **Deeper pipeline**: Faster clock but more branch penalty
- **More FUs**: Less structural hazards but more area/power
- **Forwarding**: Less stalls but more complex hardware
- **Dynamic scheduling**: Better performance but much more complex

### Modern Trends
- 10-20+ stage pipelines (deeper for higher frequency)
- Multiple issue (superscalar)
- Out-of-order execution (beyond scoreboarding)
- Sophisticated branch prediction (>95% accuracy)
- Speculative execution

---

## Quick Reference

### Common Stall Patterns
- **Load-use**: 1 cycle (with forwarding)
- **Load-branch**: 1 cycle (compare in ID)
- **Branch taken**: 1 cycle (decision in ID) to 3+ cycles (deeper pipelines)
- **FP operation**: Variable (depends on latency)

### Typical Branch Statistics
- **Branch frequency**: 10-20% of instructions
- **Taken rate**: 50-70% (varies by workload)
- **Prediction accuracy**: 80-98% (depends on predictor)
- **Delay slots filled**: 50-70% (compiler dependent)

### Pipeline Stage Abbreviations
- **IF**: Instruction Fetch
- **ID**: Instruction Decode
- **EX**: Execute
- **MEM**: Memory Access
- **WB**: Write Back
- **RO**: Read Operands (scoreboarding)
- **IS**: Issue (scoreboarding or second IF in R4000)
