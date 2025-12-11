# PIPELINING STUDY GUIDE
## Computer Architecture Final Exam - Fall 2025

---

## TABLE OF CONTENTS
1. [Introduction to Pipelining](#1-introduction-to-pipelining)
2. [Pipeline Performance Metrics](#2-pipeline-performance-metrics)
3. [Pipeline Hazards](#3-pipeline-hazards)
4. [Data Hazards and Solutions](#4-data-hazards-and-solutions)
5. [Control Hazards and Branch Prediction](#5-control-hazards-and-branch-prediction)
6. [Multi-Cycle Operations](#6-multi-cycle-operations)
7. [MIPS R4000 Pipeline](#7-mips-r4000-pipeline)
8. [Dynamic Scheduling with Scoreboarding](#8-dynamic-scheduling-with-scoreboarding)
9. [Worked Example Problems](#9-worked-example-problems)
10. [Exam Strategy](#10-exam-strategy)

---

## 1. INTRODUCTION TO PIPELINING

### What is Pipelining?

**Definition**: Implementation technique where multiple instructions are overlapped in execution

**Key Concept**: Takes advantage of parallelism among the actions needed to execute an instruction

### Classic 5-Stage RISC Pipeline

```
IF  - Instruction Fetch
ID  - Instruction Decode / Register Fetch
EX  - Execute / Address Calculation
MEM - Memory Access
WB  - Write Back
```

### Pipeline Benefits

- **Increases throughput** (not latency)
- Each instruction still takes the same time
- But more instructions complete per unit time
- Clock period determined by slowest stage

### Pipelining Visualization

```
Time →
Instruction 1:  IF  ID  EX  MEM  WB
Instruction 2:      IF  ID  EX   MEM  WB
Instruction 3:          IF  ID   EX   MEM  WB
Instruction 4:              IF   ID   EX   MEM  WB
Instruction 5:                   IF   ID   EX   MEM  WB
```

---

## 2. PIPELINE PERFORMANCE METRICS

### Clock Period Formula

```
Clock Period τ = Max{stage delays} + overhead

where:
  overhead = latch delay + clock skew
```

### Frequency

```
Frequency f = 1/τ
```

### Speedup (CRITICAL!)

```
For k-stage pipeline, n instructions:

Speedup Sₖ = (n × k) / (k + n - 1)

When n >> k:
Speedup Sₖ ≈ k  (ideal speedup)
```

### Efficiency

```
Efficiency η = Sₖ / k

where:
  Sₖ = actual speedup
  k  = number of pipeline stages
```

### Throughput

```
Throughput w = η/τ = n / [(k + n - 1) × τ]

Ideal Throughput = 1/τ  (one instruction per cycle)
```

### Pipeline with Stalls

```
Speedup = Pipeline Depth / (1 + Pipeline stall cycles per instruction)

For branches:
Speedup = Pipeline Depth / (1 + Branch_freq × Mispred_rate × Branch_penalty)
```

---

## 3. PIPELINE HAZARDS

### Three Types of Hazards

#### 1. Structural Hazards
**Definition**: Not enough HW resources to keep all instructions moving

**Example**: Single memory port needed by both IF and MEM stages

**Solutions**:
- Add more hardware (e.g., separate instruction and data caches)
- Stall the pipeline
- Careful scheduling

#### 2. Data Hazards
**Definition**: Instruction depends on result of prior instruction still in pipeline

**Types**:
- **RAW (Read After Write)**: Most common, true dependency
- **WAW (Write After Write)**: Can occur with out-of-order execution
- **WAR (Write After Read)**: Can occur with out-of-order execution

**Solutions**:
- Forwarding (bypassing)
- Stalling
- Compiler scheduling

#### 3. Control Hazards
**Definition**: Branch decision not known until later in pipeline

**Solutions**:
- Stall (freeze/flush pipeline)
- Predict not-taken
- Predict taken
- Delayed branch
- Dynamic branch prediction

---

## 4. DATA HAZARDS AND SOLUTIONS

### RAW Hazard Example

```assembly
ADD R1, R2, R3    # R1 written in WB stage (cycle 5)
SUB R4, R1, R5    # R1 needed in EX stage (cycle 4)
                  # HAZARD! R1 not ready yet
```

### Forwarding (Bypassing)

**Concept**: Use result as soon as computed, don't wait for WB

**Forwarding Paths**:
```
EX/MEM → EX  (forward from ALU output)
MEM/WB → EX  (forward from memory output)
```

### Forwarding Timing Rules

```
Forward from EX/MEM if:
  (EX/MEM.RegisterRd == ID/EX.RegisterRs1) OR
  (EX/MEM.RegisterRd == ID/EX.RegisterRs2)

Forward from MEM/WB if:
  (MEM/WB.RegisterRd == ID/EX.RegisterRs1) OR
  (MEM/WB.RegisterRd == ID/EX.RegisterRs2)
```

### Load-Use Data Hazard

**Cannot be solved by forwarding alone!**

```assembly
LD  R1, 0(R2)    # R1 available after MEM (cycle 4)
ADD R3, R1, R4   # R1 needed in EX (cycle 3)
                 # Must stall 1 cycle!
```

**Solution**: Insert 1 bubble (stall cycle)

### Data Hazard Detection Logic

```
Check for load-use hazard:
  If (ID/EX.MemRead AND
      ((ID/EX.RegisterRd == IF/ID.RegisterRs1) OR
       (ID/EX.RegisterRd == IF/ID.RegisterRs2)))
  Then stall 1 cycle
```

---

## 5. CONTROL HAZARDS AND BRANCH PREDICTION

### Branch Penalty

**Definition**: Number of stall cycles due to branch

```
For 5-stage pipeline with branch resolution in EX:
Branch penalty = 1 cycle (if predict not-taken)

For deeper pipelines (e.g., MIPS R4000):
Branch penalty = 3 cycles
```

### Static Branch Prediction

#### Predict Not-Taken
```
- Continue fetching sequentially
- If branch taken, flush wrong instructions
- Penalty only when mispredicted
```

#### Predict Taken
```
- Fetch from branch target
- Requires early target calculation
```

#### Direction-Based Prediction
```
- Forward branches: predict not-taken
- Backward branches: predict taken (loops!)
```

### Dynamic Branch Prediction

#### 2-Bit Predictor State Machine

```
State 11 (Strongly Taken)     → Predict Taken
    ↓ (not taken)
State 10 (Weakly Taken)       → Predict Taken
    ↓ (not taken)
State 01 (Weakly Not-Taken)   → Predict Not-Taken
    ↓ (not taken)
State 00 (Strongly Not-Taken) → Predict Not-Taken
```

**Advantage**: Tolerates occasional misprediction in loops

### Branch Penalty with Prediction

```
CPI_with_branches = 1 + Branch_freq × Mispred_rate × Branch_penalty

Example:
  Branch frequency = 20% (0.20)
  Misprediction rate = 10% (0.10)
  Branch penalty = 3 cycles

  CPI = 1 + 0.20 × 0.10 × 3 = 1 + 0.06 = 1.06
```

### Delayed Branch

**Concept**: Branch takes effect after delay slot instruction(s)

```assembly
Original:
    BEQZ R1, TARGET
    ADD  R2, R3, R4   # Only executes if not taken

With delayed branch:
    ADD  R2, R3, R4   # Moved to delay slot
    BEQZ R1, TARGET   # Branch executes AFTER add
```

**Scheduling Options**:
1. From before (best)
2. From target (if branch likely taken)
3. From fall-through (if branch likely not-taken)

---

## 6. MULTI-CYCLE OPERATIONS

### Latency vs Initiation Interval

**Latency**: Cycles until result is ready

**Initiation Interval**: Cycles before can issue same type of instruction

### Typical FP Unit Latencies

```
Functional Unit               Latency    Initiation Interval
Integer ALU                   0          1
Data memory (load/store)      1          1
FP add/subtract               3-4        1
FP multiply                   6-7        1-4
FP divide                     24-25      24-25
```

### Additional Hazards with Multi-Cycle Ops

#### 1. WAW Hazards

```assembly
MULTD F0, F2, F4   # F0 written in cycle 8
ADDD  F0, F6, F8   # F0 written in cycle 5
                   # WAW! Wrong value in F0!
```

**Solution**: Check at issue, stall if WAW exists

#### 2. Multiple Writes Same Cycle

**Problem**: Two instructions try to write in same cycle

**Solution**:
- Stall one instruction
- Add more write ports (expensive)

#### 3. Out-of-Order Completion

**Problem**: Later instructions finish before earlier ones

**Impact**: Exception handling becomes complex

---

## 7. MIPS R4000 PIPELINE

### 8-Stage "Superpipeline"

```
IF - Instruction Fetch (1st half)
IS - Instruction fetch (2nd half)
RF - Register Fetch / decode
EX - Execute
DF - Data Fetch (1st half)
DS - Data fetch (2nd half)
TC - Tag Check (cache hit?)
WB - Write Back
```

### Two-Cycle Load Latency

```
LD   R1, 0(R2)    IF  IS  RF  EX  DF  DS  TC  WB
ADD  R3, R1, R4           IF  IS  RF  Stall Stall EX  DF  DS
SUB  R5, R1, R6                   IF  Stall Stall IS  RF  EX
```

**2 stall cycles required** for load-use dependency!

### Three-Cycle Branch Delay

```
BEQZ R1, TARGET   IF  IS  RF  EX  DF  DS  TC  WB
Delay slot 1          IF  IS  RF  EX  DF  DS  TC  WB
Delay slot 2              IF  IS  RF  EX  DF  DS  TC  WB
Delay slot 3                  IF  IS  RF  EX  DF  DS  TC
TARGET                             IF  IS  RF  EX  DF
```

**3 delay slots** must be filled (or stalled)

### R4000 FP Pipeline Stages

```
U - Unpack FP numbers
A - Mantissa ADD
R - Round
S - Shift
E - Exception test
M - Multiply 1st stage
N - Multiply 2nd stage
D - Divide stage
```

### R4000 FP Latencies

```
FP Instruction    Latency    Initiation    Stages
Add/Subtract      4          3             U, S+A, A+R, R+S
Multiply          8          4             U, E+M, M, M, M, N, N+A, R
Divide            36         35            U, A, R, D^27, D+A, D+R...
```

---

## 8. DYNAMIC SCHEDULING WITH SCOREBOARDING

### Scoreboarding Concept

**Goal**: Execute instructions out-of-order when safe to do so

**Key Idea**: Hardware tracks dependencies dynamically

### Four Pipeline Stages with Scoreboarding

```
1. Issue (IS):     Decode, check structural/WAW hazards
2. Read Operands:  Wait for RAW hazards to clear, then read
3. Execute (EX):   Operate on operands
4. Write Result:   Check for WAR hazards, then write
```

### Scoreboard Data Structures

#### Instruction Status Table
```
Tracks: Issue, Read Operands, Execute Complete, Write Result
```

#### Functional Unit Status Table
```
For each FU:
  Busy:  Is unit busy? (Yes/No)
  Op:    Operation being performed
  Fi:    Destination register
  Fj,Fk: Source registers
  Qj,Qk: FUs producing Fj, Fk (or 0 if ready)
  Rj,Rk: Flags: Fj, Fk ready? (Yes/No)
```

#### Register Result Status Table
```
For each register:
  FU:  Which FU will write to this register? (or blank)
```

### Scoreboarding Hazard Detection

**Structural Hazard at Issue**:
```
Can issue if:
  - Required FU is free (not busy)
  - No other active instruction writing same destination
```

**RAW Hazard at Read Operands**:
```
Can read if:
  Rj == Yes AND Rk == Yes
  (both source operands ready)
```

**WAR Hazard at Write Result**:
```
Can write if:
  For all earlier instructions i in Read Operands stage:
    i.Fj != this.Fi AND i.Fk != this.Fi
```

**WAW Hazard at Issue**:
```
Cannot issue if:
  Another FU has Busy=Yes and Fi == this.Fi
```

---

## 9. WORKED EXAMPLE PROBLEMS

### Problem 1: Basic Pipeline Speedup

**Given**:
- 5-stage pipeline
- Each stage takes 200 ps
- Latch overhead: 20 ps
- Execute 100 instructions
- Compare to unpipelined execution

**Solution**:

```
Unpipelined:
  Time per instruction = 5 × 200 ps = 1000 ps
  Total time = 100 × 1000 ps = 100,000 ps

Pipelined:
  Clock period = 200 ps + 20 ps = 220 ps
  Total cycles = 5 + 100 - 1 = 104 cycles
  Total time = 104 × 220 ps = 22,880 ps

Speedup = 100,000 / 22,880 = 4.37

Theoretical maximum = 5
Efficiency = 4.37 / 5 = 0.874 = 87.4%
```

---

### Problem 2: Pipeline with Load-Use Stalls

**Given**:
```assembly
1. LD  F2, 0(R1)
2. LD  F4, 4(R1)
3. MULTD F0, F2, F4
4. SUBD F8, F2, F6
5. DIVD F10, F0, F6
6. ADDD F6, F8, F2
```

Latencies: LD=1, MULTD=6, SUBD=3, DIVD=24, ADDD=3

**Find**: Total execution cycles with forwarding

**Solution**:

```
Inst  IS  RO  EX              WR  Notes
1     1   2   3               4
2     2   3   4               5
3     3   4   5-6-7-8-9-10    11  Waits 1 for F2, F4 from LD
4     4   5   6-7-8           9   Waits 1 for F2
5     5   12  13-...-36       37  Waits for F0 from MULTD (cycle 11)
6     6   10  11-12-13        14  Waits for F8 from SUBD (cycle 9)

Total cycles = 37
```

---

### Problem 3: Branch Penalty Calculation

**Given**:
- 20% of instructions are branches
- Branch prediction accuracy = 85%
- Branch penalty (when wrong) = 3 cycles
- Base CPI = 1.0

**Find**: Actual CPI with branches

**Solution**:

```
Misprediction rate = 1 - 0.85 = 0.15 = 15%

CPI = Base_CPI + Branch_freq × Mispred_rate × Penalty
    = 1.0 + 0.20 × 0.15 × 3
    = 1.0 + 0.09
    = 1.09

Performance loss = (1.09 - 1.00) / 1.00 = 9%
```

---

### Problem 4: Comparing Pipeline Depths

**Given**:
- Design A: 5 stages, 250 ps per stage, 25 ps overhead
- Design B: 10 stages, 130 ps per stage, 30 ps overhead
- Execute 1000 instructions

**Find**: Which design is faster?

**Solution**:

```
Design A:
  Clock period = 250 + 25 = 275 ps
  Cycles = 5 + 1000 - 1 = 1004
  Time = 1004 × 275 = 276,100 ps

Design B:
  Clock period = 130 + 30 = 160 ps
  Cycles = 10 + 1000 - 1 = 1009
  Time = 1009 × 160 = 161,440 ps

Speedup B over A = 276,100 / 161,440 = 1.71×

Design B is 71% faster!
```

---

### Problem 5: Scoreboarding Example

**Given code**:
```assembly
1. LD    F6, 34(R2)
2. LD    F2, 45(R3)
3. MULTD F0, F2, F4
4. SUBD  F8, F6, F2
5. DIVD  F10, F0, F6
6. ADDD  F6, F8, F2
```

**Fill scoreboard after instruction 3 issues:**

**Instruction Status**:
```
Inst  Issue  Read  Exec  Write
1     √      √     √     √
2     √      √     √
3     √
4     √
5     √
6
```

**Functional Unit Status**:
```
Name     Busy  Op    Fi   Fj   Fk   Qj      Qk   Rj   Rk
Integer  Yes   Load  F2   R3              No
Mult1    Yes   Mult  F0   F2   F4   Integer      No   Yes
Add      Yes   Sub   F8   F6   F2                Yes  No
Divide   Yes   Div   F10  F0   F6   Mult1        No   Yes
```

**Register Result Status**:
```
F0: Mult1
F2: Integer
F8: Add
F10: Divide
```

---

### Problem 6: MIPS R4000 Load Delay

**Given**:
```assembly
LD   R1, 0(R2)
ADD  R3, R1, R4
SUB  R5, R3, R6
```

**Draw pipeline diagram for R4000 (8 stages)**

**Solution**:

```
      1   2   3   4   5   6   7   8   9   10  11
LD    IF  IS  RF  EX  DF  DS  TC  WB
ADD           IF  IS  RF  --  --  EX  DF  DS  TC  WB
SUB                   IF  --  --  IS  RF  EX  DF  DS

2 stall cycles inserted for ADD waiting for R1
SUB must also wait for ADD to clear
```

---

### Problem 7: Multi-Cycle FP Pipeline

**Given**:
```assembly
FMUL.D F0, F2, F4   (latency 7)
FADD.D F8, F0, F6   (latency 4)
```

**Pipeline stages**:
- FMUL: IF ID M1 M2 M3 M4 M5 M6 M7 WB
- FADD: IF ID A1 A2 A3 A4 WB

**Find**: When can FADD write?

**Solution**:

```
      1  2  3  4  5  6  7  8  9  10 11 12 13
FMUL  IF ID M1 M2 M3 M4 M5 M6 M7 WB
FADD     IF ID -- -- -- -- -- A1 A2 A3 A4 WB

FADD must stall in ID until F0 ready
F0 ready after M7 (cycle 9)
FADD starts A1 in cycle 10
FADD writes in cycle 13
```

---

### Problem 8: Amdahl's Law with Pipeline

**Given**:
- Program has 30% loads (1 cycle latency)
- Perfect forwarding except load-use
- 25% of instructions use load result immediately
- Base CPI = 1.0

**Find**: Actual CPI

**Solution**:

```
Load-use cases = 0.30 × 0.25 = 0.075 = 7.5% of instructions

Each load-use adds 1 stall cycle

CPI = 1.0 + 0.075 × 1 = 1.075
```

---

## 10. EXAM STRATEGY

### Most Likely Question Types

Based on HW2, HW3 pattern, expect:

1. **Pipeline Timing Diagrams** (VERY LIKELY!)
   - Draw pipeline execution for code sequence
   - Identify hazards and show stalls/forwarding
   - Calculate total cycles

2. **Speedup Calculations** (CRITICAL!)
   - Compare pipelined vs unpipelined
   - Calculate with different stage delays
   - Account for hazards and stalls

3. **Hazard Detection** (COMMON)
   - Identify RAW, WAW, WAR hazards
   - Determine when forwarding works
   - Calculate stall cycles needed

4. **Branch Prediction** (LIKELY)
   - Calculate CPI with branch penalties
   - Compare prediction schemes
   - Dynamic predictor state diagrams

5. **Scoreboarding** (MODERATE)
   - Fill in scoreboard tables
   - Determine when instruction can proceed
   - Identify hazards from tables

6. **Multi-Cycle Operations** (LIKELY)
   - Calculate completion times
   - Handle out-of-order execution
   - Structural hazards

---

### Step-by-Step Problem Solving

#### For Pipeline Timing Problems:

```
1. Set up table with columns for each cycle
2. Start first instruction in cycle 1
3. For each subsequent instruction:
   - Try to advance each stage
   - Check for hazards:
     * Structural: Can both use this resource?
     * Data: Is source ready?
     * Control: Is this the right instruction?
   - Insert stalls/bubbles as needed
   - Mark forwarding paths
4. Count total cycles = Last WB cycle
5. Calculate CPI = Total cycles / Num instructions
```

#### For Speedup Problems:

```
1. Calculate unpipelined time:
   Time = Instructions × Sum(all stage delays)

2. Calculate pipelined time:
   Clock = Max(stage delay) + overhead
   Cycles = k + n - 1 (+ stalls)
   Time = Cycles × Clock

3. Speedup = Unpipelined / Pipelined

4. Check if reasonable (should be < k stages)
```

#### For Hazard Detection:

```
1. For each instruction i:
   - List registers it reads (in EX)
   - List registers it writes (in WB)

2. Compare instruction i with i+1, i+2, i+3:
   - RAW: i writes R, i+j reads R (need forwarding/stall)
   - WAW: i writes R, i+j writes R (both write same reg)
   - WAR: i reads R, i+j writes R (out-of-order only)

3. Check if forwarding resolves:
   - EX/MEM → EX: 1 cycle gap ok
   - MEM/WB → EX: 2 cycle gap ok
   - Load-use: Need 1 stall even with forwarding
```

---

### Common Mistakes to Avoid

1. **Forgetting latch/clock overhead** in clock period calculation
2. **Counting cycles wrong**: It's k + n - 1, NOT k × n
3. **Assuming forwarding always works**: Load-use ALWAYS needs stall
4. **Wrong hazard type**:
   - RAW is "Read After Write" (true dependency)
   - WAR is "Write After Read" (anti-dependency)
   - WAW is "Write After Write" (output dependency)
5. **Branch penalty**: Count from when branch executed, not issued
6. **Multi-cycle ops**: Latency ≠ Initiation interval
7. **Scoreboarding**: Check ALL four conditions (structural, RAW, WAW, WAR)
8. **R4000 pipeline**: Has 2-cycle load delay, not 1!

---

### Formula Quick Reference

```
Clock Period:        τ = Max(stage delay) + overhead
Speedup:            S = (n × k) / (k + n - 1) ≈ k when n >> k
Efficiency:         η = S / k
CPI with branches:  CPI = 1 + f_br × miss_rate × penalty
Pipeline stalls:    S = k / (1 + stalls_per_inst)
```

---

### Exam Checklist

- [ ] Know 5-stage pipeline stages (IF, ID, EX, MEM, WB)
- [ ] Speedup formula and when it approaches k
- [ ] Three hazard types and solutions
- [ ] When forwarding works and when it doesn't
- [ ] Load-use hazard ALWAYS needs 1 stall
- [ ] Branch prediction schemes (static and 2-bit)
- [ ] Multi-cycle operation timing
- [ ] WAW and WAR only with out-of-order execution
- [ ] Scoreboarding tables and conditions
- [ ] MIPS R4000 has 8 stages and 2-cycle load delay

---

## FINAL TIPS

1. **Draw timing diagrams** - they help visualize hazards
2. **Show your work** - partial credit is available
3. **Check your arithmetic** - simple errors lose points
4. **Use forwarding when possible** - don't stall unnecessarily
5. **Remember**: Load-use hazards CANNOT be solved by forwarding alone

**Good luck on your exam!**
