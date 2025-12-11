# MEMORY HIERARCHY - Study Guide
## Computer Architecture Final Exam - Fall 2025

---

## OVERVIEW: What to Expect on the Exam

Based on HW patterns, expect **calculation-heavy problems** involving:
- Cache optimization trade-off analysis
- AMAT calculations with different optimizations
- Memory timing calculations (DRAM access time, bandwidth)
- Virtual memory address translation
- Cache tag/index/offset bit calculations with optimizations

---

## 1. THE 11 ADVANCED CACHE OPTIMIZATIONS

### Category 1: Reducing Hit Time

#### **Optimization 1: Small and Simple First-Level Caches**
**Key Concept**: L1 cache must be fast → smaller size, lower associativity
- Direct-mapped or 2-way set-associative L1
- Typical size: 16-64 KB for L1
- **Trade-off**: Lower hit rate but faster hit time

**Exam Application**:
```
Given: 32KB direct-mapped L1 vs 32KB 4-way set-associative L1
Direct-mapped: Hit time = 1 cycle, Miss rate = 5%
4-way set-assoc: Hit time = 2 cycles, Miss rate = 3%
Miss penalty = 50 cycles

AMAT_direct = 1 + (0.05 × 50) = 1 + 2.5 = 3.5 cycles
AMAT_4way = 2 + (0.03 × 50) = 2 + 1.5 = 3.5 cycles
→ Same performance, choose simpler direct-mapped
```

#### **Optimization 2: Way Prediction**
**Key Concept**: Predict which way in set-associative cache will hit
- Access predicted way first (like direct-mapped speed)
- If wrong, check other ways (penalty: 1 extra cycle typically)
- **Trade-off**: Reduces effective hit time when prediction is good

**Exam Application**:
```
4-way set-associative cache with way prediction
Prediction accuracy = 90%
Hit time (correct prediction) = 1 cycle
Hit time (wrong prediction) = 2 cycles
Overall hit rate = 95%

Average hit time = 0.90 × 1 + 0.10 × 2 = 1.1 cycles
AMAT = (0.95 × 1.1) + (0.05 × Miss_penalty)
```

---

### Category 2: Increasing Cache Bandwidth

#### **Optimization 3: Pipelined Cache Access**
**Key Concept**: Allow multiple cache accesses in flight
- Increases bandwidth (throughput)
- Does NOT reduce latency of single access
- **Trade-off**: Greater penalty on branch misprediction

**Example**:
- Non-pipelined: 1 access per 2 cycles → 0.5 accesses/cycle
- Pipelined: 1 access per cycle → 1 access/cycle (but still 2-cycle latency)

#### **Optimization 4: Multibanked Caches**
**Key Concept**: Divide cache into independent banks
- Sequential interleaving: Bank = (Block address) % Number_of_banks
- Allows simultaneous accesses to different banks
- **Used heavily in**: GPUs, vector processors

**Exam Application**:
```
4-banked cache, 64-byte blocks
Access sequence: Addresses 0x0000, 0x0040, 0x0080, 0x00C0

Bank assignments (assuming 64-byte blocks):
0x0000 → Block 0 → Bank 0
0x0040 → Block 1 → Bank 1
0x0080 → Block 2 → Bank 2
0x00C0 → Block 3 → Bank 3
→ All 4 accesses can proceed in parallel!
```

#### **Optimization 5: Nonblocking Caches (Hit Under Miss)**
**Key Concept**: Continue servicing cache hits during a miss
- **Hit under miss**: Allow hits while 1 miss outstanding
- **Hit under multiple miss**: Allow hits while N misses outstanding
- **Critical for**: Out-of-order processors

**Metrics**:
- **Bandwidth** improves (more accesses/time)
- **Latency** unchanged (individual access time same)

---

### Category 3: Reducing Miss Penalty

#### **Optimization 6: Critical Word First & Early Restart**
**Key Concept**: Don't wait for entire block before restarting CPU
- **Critical word first**: Request missed word first from memory
- **Early restart**: Resume execution as soon as requested word arrives

**Exam Application**:
```
Block size = 64 bytes (16 words)
Miss penalty without optimization = 16 cycles (fetch all words sequentially)
Critical word first: Fetch requested word in 4 cycles, continue program
Effective miss penalty = 4 cycles (program resumes early)
```

#### **Optimization 7: Merging Write Buffer**
**Key Concept**: Combine multiple writes to same block in write buffer
- Reduces memory bandwidth usage
- Especially effective for write-through caches

**Example**:
```
Without merging: 4 separate writes to block X → 4 memory transactions
With merging: 4 writes combined → 1 memory transaction
Memory bandwidth saved = 75%
```

---

### Category 4: Reducing Miss Rate

#### **Optimization 8: Compiler Optimizations**

**A. Loop Interchange** (fix stride issues)
```c
// BAD: Column-major access in row-major language (C)
for (j = 0; j < 100; j++)      // outer loop
    for (i = 0; i < 5000; i++) // inner loop
        x[i][j] = 2 * x[i][j];

// GOOD: Row-major access (sequential)
for (i = 0; i < 5000; i++)     // outer loop
    for (j = 0; j < 100; j++)   // inner loop
        x[i][j] = 2 * x[i][j];

Cache impact:
- BAD: 5000 iterations × miss every iteration = high miss rate
- GOOD: Sequential access → miss only on new cache blocks
```

**B. Blocking** (improve temporal locality)
```c
// Matrix multiplication without blocking
for (i = 0; i < N; i++)
    for (j = 0; j < N; j++)
        for (k = 0; k < N; k++)
            C[i][j] += A[i][k] * B[k][j];

// With blocking (block size B)
for (ii = 0; ii < N; ii += B)
    for (jj = 0; jj < N; jj += B)
        for (kk = 0; kk < N; kk += B)
            for (i = ii; i < min(ii+B, N); i++)
                for (j = jj; j < min(jj+B, N); j++)
                    for (k = kk; k < min(kk+B, N); k++)
                        C[i][j] += A[i][k] * B[k][j];

Benefits: Submatrices fit in cache → fewer capacity misses
```

**C. Merging Arrays** (improve spatial locality)
```c
// BAD: Arrays of structures (poor spatial locality)
int val[SIZE];
int key[SIZE];

// GOOD: Structure of arrays (better spatial locality)
struct merged {
    int val;
    int key;
} data[SIZE];
```

#### **Optimization 9: Hardware Prefetching**
**Key Concept**: Hardware detects access patterns and prefetches data
- **Instruction prefetch**: Fetch next sequential block
- **Data prefetch**: Detect stride patterns

**Types**:
1. **Tagged prefetch**: Prefetched data marked separately (doesn't pollute cache)
2. **Regular prefetch**: Prefetched data enters cache normally

**Trade-offs**:
- ✓ Reduces compulsory misses
- ✗ Can prefetch useless data (pollute cache)
- ✗ Uses memory bandwidth

#### **Optimization 10: Compiler-Controlled Prefetch**
**Key Concept**: Insert prefetch instructions in code
- **Binding prefetch**: Data must be in cache (causes exception on fault)
- **Non-binding prefetch**: Hint only (no exception)

**Example**:
```c
// Prefetch next iteration's data
for (i = 0; i < N; i++) {
    prefetch(a[i+1]);  // Prefetch for next iteration
    sum += a[i];       // Use current iteration's data
}
```

**Calculation Example**:
```
Loop without prefetch:
- Miss every 8 iterations (64-byte blocks, 8-byte elements)
- Miss penalty = 100 cycles
- Average penalty = 100/8 = 12.5 cycles/iteration

Loop with prefetch (assuming perfect prefetching):
- All misses hidden (data arrives before needed)
- Average penalty = 0 cycles/iteration
- Speedup = 12.5 / 0 (or calculate based on total execution time)
```

---

### Category 5: Reducing Miss Penalty OR Miss Rate via Parallelism

#### **Optimization 11: Multi-level Caches**
**Key Concept**: Use cache hierarchy L1, L2, L3
- L1: Small, fast (1-2 cycles)
- L2: Medium, moderate speed (10-20 cycles)
- L3: Large, slower (30-50 cycles)

**CRITICAL FORMULAS**:
```
Local miss rate = Misses_in_cache / Accesses_to_cache
Global miss rate = Misses_in_cache / Total_CPU_memory_accesses

Average Memory Access Time (2-level):
AMAT = Hit_time_L1 + Miss_rate_L1 × Miss_penalty_L1

where:
Miss_penalty_L1 = Hit_time_L2 + Miss_rate_L2 × Miss_penalty_L2

Full expansion:
AMAT = Hit_time_L1 + Miss_rate_L1 × (Hit_time_L2 + Miss_rate_L2 × Miss_penalty_L2)
```

**Exam Problem Example**:
```
L1: 32 KB, 1 cycle hit time, 5% miss rate
L2: 256 KB, 10 cycle hit time, 20% local miss rate
Main memory: 100 cycle access time

Global L2 miss rate = L1_miss_rate × L2_local_miss_rate
                    = 0.05 × 0.20 = 0.01 (1%)

AMAT = 1 + 0.05 × (10 + 0.20 × 100)
     = 1 + 0.05 × (10 + 20)
     = 1 + 0.05 × 30
     = 1 + 1.5
     = 2.5 cycles

Alternative calculation:
AMAT = Hit_time_L1 + Miss_rate_L1 × Hit_time_L2 + Global_miss_rate_L2 × Mem_time
     = 1 + 0.05 × 10 + 0.01 × 100
     = 1 + 0.5 + 1.0
     = 2.5 cycles ✓
```

---

## 2. MEMORY TECHNOLOGY

### SRAM (Static RAM)
**Characteristics**:
- 6 transistors per bit
- No refresh needed (holds data as long as powered)
- Fast (1-10 ns)
- Expensive
- **Used for**: Cache memory

### DRAM (Dynamic RAM)
**Characteristics**:
- 1 transistor + 1 capacitor per bit
- Needs periodic refresh (capacitor leaks)
- Slower than SRAM (50-70 ns)
- Cheaper, denser
- **Used for**: Main memory

### DRAM Access: RAS and CAS
**Row Access Strobe (RAS)**: Activates row in DRAM array
**Column Access Strobe (CAS)**: Selects column from activated row

**Timing**:
1. Assert RAS → row buffer loaded (T_RAS)
2. Assert CAS → column selected (T_CAS)
3. Data available after T_CAS

**Access Time = T_RAS + T_CAS**

### DDR SDRAM (Double Data Rate)

**Key Formula**:
```
Data Rate (MT/s) = 2 × I/O Bus Clock Rate

Peak Bandwidth (GB/s) = (Data Rate × Bus Width) / 8
                       = (Data Rate × 64 bits) / 8
                       = Data Rate × 8 bytes/transfer
```

**DIMM Naming Convention**:
```
PC-XXXX where XXXX = Peak Bandwidth in MB/s (rounded)

Example: DDR3-1600
I/O clock = 800 MHz
Data rate = 2 × 800 = 1600 MT/s
Peak BW = 1600 × 8 = 12,800 MB/s
DIMM name = PC3-12800 (or rounded to PC3-10600 for product)
```

**Evolution**:
- DDR1: 2× prefetch (2 bits per I/O clock)
- DDR2: 4× prefetch (4 bits per I/O clock)
- DDR3: 8× prefetch (8 bits per I/O clock)
- DDR4: 8× prefetch (improved voltage, speed)

**Exam Calculation**:
```
Given: DDR4-3200 DIMM
I/O clock = 3200/2 = 1600 MHz
Data rate = 3200 MT/s
Peak bandwidth = 3200 × 8 = 25,600 MB/s = 25.6 GB/s
DIMM name = PC4-25600
```

---

## 3. VIRTUAL MEMORY

### Address Translation
```
Virtual Address = [Virtual Page Number | Page Offset]
Physical Address = [Physical Page Number | Page Offset]

Page offset bits = log₂(Page Size)
VPN bits = Virtual address width - Page offset bits
PPN bits = Physical address width - Page offset bits
```

**Example**:
```
Virtual address: 32 bits
Physical address: 30 bits
Page size: 4 KB = 2^12 bytes

Page offset = 12 bits
VPN = 32 - 12 = 20 bits → 2^20 = 1M pages in virtual space
PPN = 30 - 12 = 18 bits → 2^18 = 256K pages in physical memory
```

### TLB (Translation Lookaside Buffer)
**Purpose**: Cache for page table entries

**Effective Access Time**:
```
EAT = Hit_rate_TLB × (TLB_hit_time + Memory_access)
    + Miss_rate_TLB × (TLB_hit_time + Page_table_access + Memory_access)

Simplified (assuming TLB hit time negligible):
EAT = Memory_access + Miss_rate_TLB × Page_table_access
```

**Example**:
```
TLB hit rate = 95%
Memory access time = 100 ns
Page table in memory access time = 100 ns

EAT = 100 + 0.05 × 100 = 100 + 5 = 105 ns
```

### Virtual Machines (VMs)
**Virtual Machine Monitor (VMM/Hypervisor)**: Software layer managing VMs

**Two Approaches**:
1. **Type 1 (Bare Metal)**: VMM runs directly on hardware
   - Examples: VMware ESXi, Xen
2. **Type 2 (Hosted)**: VMM runs on host OS
   - Examples: VMware Workstation, VirtualBox

**Paravirtualization** (Xen example):
- Guest OS modified to call hypervisor directly
- Better performance than full virtualization
- Trade-off: Guest OS must be modified

---

## 4. REAL-WORLD EXAMPLES

### ARM Cortex-A53 Memory Hierarchy
```
L1 I-cache: 8-64 KB, 2-way, 64-byte blocks
L1 D-cache: 8-64 KB, 4-way, 64-byte blocks
L2 unified: 128 KB-2 MB, 16-way, 64-byte blocks
```

### Intel Core i7 Memory Hierarchy
```
L1 I-cache: 32 KB, 8-way, 64-byte blocks
L1 D-cache: 32 KB, 8-way, 64-byte blocks
L2 unified: 256 KB, 8-way, 64-byte blocks (per core)
L3 unified: 8 MB, 16-way, 64-byte blocks (shared)
```

**Key Observation**: Higher levels → higher associativity (reduce conflict misses)

---

## 5. ADVANCED TOPICS

### Die Stacking (2.5D and 3D)
**2.5D Stacking**:
- Multiple dies on silicon interposer
- Short connections → high bandwidth, low power

**3D Stacking**:
- Dies stacked vertically with through-silicon vias (TSVs)
- Even higher bandwidth
- **Challenge**: Heat dissipation

**Application**: High-bandwidth memory (HBM) for GPUs

---

## 6. EXAM STRATEGY FOR MEMORY QUESTIONS

### What to Memorize:
1. ✓ AMAT formula (forwards and backwards)
2. ✓ Tag/Index/Offset bit calculations
3. ✓ DDR bandwidth formula
4. ✓ Local vs Global miss rates
5. ✓ All 11 optimization categories

### Common Calculation Types:
1. **AMAT with multi-level caches** (see section 1, Optimization 11)
2. **Trade-off analysis**: Given two cache designs, which is better?
3. **Bandwidth calculations**: DDR memory speed → peak bandwidth
4. **Virtual memory**: Page offset, VPN bits, TLB EAT
5. **Loop optimization**: Identify stride problems, apply blocking

### Red Flags (Common Mistakes):
- ✗ Confusing local vs global miss rate
- ✗ Forgetting to include L1 hit time in AMAT
- ✗ Using wrong units (cycles vs ns, MB vs GB)
- ✗ Not accounting for write-back vs write-through in miss penalty
- ✗ Forgetting that block offset bits change with block size

### Cross-Reference to Homework:
- **HW3 Q3**: Cache tag size calculations (direct, set-assoc, fully-assoc)
- **HW4 Q2**: Miss type identification (relates to cache optimizations)
- Formula sheet section 4: Cache memory formulas

---

## 7. PRACTICE PROBLEMS

### Problem 1: Multi-level Cache AMAT
```
L1: 16 KB, 2-way, 64-byte blocks, 1 cycle hit, 4% miss rate
L2: 256 KB, 8-way, 64-byte blocks, 10 cycle hit, 30% local miss rate
Memory: 200 cycle access

Calculate AMAT.

Solution:
Global L2 miss rate = 0.04 × 0.30 = 0.012 (1.2%)
AMAT = 1 + 0.04 × (10 + 0.30 × 200)
     = 1 + 0.04 × (10 + 60)
     = 1 + 0.04 × 70
     = 1 + 2.8
     = 3.8 cycles
```

### Problem 2: DDR Bandwidth
```
Given DDR4-2400 memory, 64-bit bus
Calculate peak bandwidth.

Solution:
I/O clock = 2400/2 = 1200 MHz
Data rate = 2400 MT/s
Peak BW = 2400 × 8 = 19,200 MB/s = 19.2 GB/s
```

### Problem 3: Optimization Identification
```
Scenario: Processor can continue executing loads that hit in cache
while a previous load miss is being serviced.

Which optimization is this?
Answer: Nonblocking cache (Hit under miss) - Optimization 5
```

---

## SUMMARY CHECKLIST

Before the exam, ensure you can:
- ✓ Calculate AMAT for 2-level and 3-level caches
- ✓ Distinguish local vs global miss rates
- ✓ Calculate tag/index/offset for all cache types
- ✓ Compute DDR peak bandwidth from data rate
- ✓ Identify which of the 11 optimizations applies to a scenario
- ✓ Analyze loop code for stride problems
- ✓ Calculate TLB effective access time
- ✓ Determine page offset bits from page size

**Most Likely Exam Topics** (based on HW emphasis):
1. AMAT calculations (multi-level caches)
2. Cache organization bit calculations
3. Optimization trade-off analysis
4. Memory bandwidth calculations

---

**Remember**: Show ALL work for partial credit! Write formulas first, then substitute values.
