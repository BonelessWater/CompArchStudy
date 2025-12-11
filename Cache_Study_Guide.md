# CACHE BASICS - Study Guide
## Computer Architecture Final Exam - Fall 2025

---

## OVERVIEW: What to Expect on the Exam

Based on HW patterns, expect **calculation-heavy problems** involving:
- Tag/Index/Offset bit calculations for different cache types
- AMAT (Average Memory Access Time) calculations
- Hit rate and miss rate analysis
- Cache access examples (table-filling exercises)
- Virtual-to-physical address translation
- TLB operations

---

## 1. PRINCIPLE OF LOCALITY

### **Two Types of Locality**

**Temporal Locality**: Items accessed recently are likely to be accessed again soon
- Examples: Loop instructions, induction variables (loop counters)
- Cache benefit: Keep recently used data in cache

**Spatial Locality**: Items near those accessed now are likely to be accessed soon
- Examples: Sequential instruction access, array data
- Cache benefit: Fetch entire blocks (not just one word)

**Key Insight**: Programs access a small portion of their address space at any time

---

## 2. FOUR BASIC CACHE DESIGN QUESTIONS

Every cache design must answer these questions:

### **Q1: Block Placement - Where can a block be placed?**
- **Direct-mapped**: Only ONE location (block address mod number of blocks)
- **Fully-associative**: ANY location in cache
- **N-way set-associative**: ANY location within ONE set

### **Q2: Block Identification - How is a block found?**
- Use **tags** to identify which block is stored
- Use **valid bit** to indicate if block contains valid data
- Address breakdown: **Tag | Index | Offset**

### **Q3: Block Replacement - Which block to replace?**
- **Direct-mapped**: No choice (only one location)
- **Associative**: LRU, Random, FIFO

### **Q4: Write Strategy - How are writes handled?**
- **Write-through**: Also update memory (use write buffer)
- **Write-back**: Only update cache (use dirty bit)
- **Write allocation**: Fetch block on write miss vs. write around

---

## 3. CACHE TYPES AND ADDRESS BREAKDOWN

### **General Address Format**
```
Memory Address = [Tag | Index | Offset]

Block offset bits = log₂(Block size in bytes)
```

### **Direct-Mapped Cache**

**Placement**: (Block address) mod (Number of blocks)

**Address Breakdown**:
```
Tag | Index | Offset

Index bits = log₂(Number of blocks)
Tag bits = Address bits - Index bits - Offset bits
```

**Example**: 8 blocks, 1 word/block, 32-bit address
```
Block size = 4 bytes → Offset = 2 bits
8 blocks → Index = 3 bits (log₂ 8 = 3)
Tag = 32 - 3 - 2 = 27 bits
```

**Characteristics**:
- ✓ Fast (simple lookup)
- ✓ Low hardware cost
- ✗ High conflict miss rate

### **Fully-Associative Cache**

**Placement**: Block can go anywhere

**Address Breakdown**:
```
Tag (Block Address) | Offset

Index bits = 0 (no index needed!)
Tag bits = Address bits - Offset bits
```

**Characteristics**:
- ✓ Lowest miss rate (no conflict misses)
- ✗ Slow (must check ALL tags in parallel)
- ✗ High hardware cost (comparator for each block)
- Usage: Small caches only (TLBs, victim caches)

### **N-Way Set-Associative Cache**

**Placement**: Block goes to specific set, any way within that set

**Address Breakdown**:
```
Tag | Set | Offset

Number of sets = Number of blocks / Associativity
Set bits = log₂(Number of sets)
Tag bits = Address bits - Set bits - Offset bits
```

**Example**: 8 blocks, 2-way set-associative, 64-byte blocks
```
8 blocks / 2 ways = 4 sets
Block offset = log₂(64) = 6 bits
Set bits = log₂(4) = 2 bits
Tag bits = 32 - 2 - 6 = 24 bits
```

**Characteristics**:
- Compromise between direct-mapped and fully-associative
- Most common in real processors
- Higher associativity → lower miss rate, longer hit time

---

## 4. CACHE SIZE EQUATION

**CRITICAL FORMULA**:
```
Cache Size = Block Size × Number of Sets × Associativity

Block Size = 2^(Offset bits)
Number of Sets = 2^(Index/Set bits)
Tag bits = Address bits - Index bits - Offset bits
```

**Practice Problem**:
```
Given: 64 KB cache, 4-way set-associative, 64-byte blocks
Find: Number of sets, index bits, tag bits (32-bit address)

Solution:
64 KB = 65,536 bytes
Block size = 64 bytes → 65,536 / 64 = 1024 blocks total
Sets = 1024 blocks / 4 ways = 256 sets

Offset bits = log₂(64) = 6 bits
Index bits = log₂(256) = 8 bits
Tag bits = 32 - 8 - 6 = 18 bits
```

---

## 5. CACHE PERFORMANCE METRICS

### **Hit Rate and Miss Rate**
```
Hit Rate = Number of hits / Total accesses
Miss Rate = Number of misses / Total accesses = 1 - Hit Rate
```

### **Average Memory Access Time (AMAT)**
```
AMAT = Hit Time + (Miss Rate × Miss Penalty)
```

**Example**:
```
Hit time = 1 cycle
Miss rate = 5%
Miss penalty = 100 cycles

AMAT = 1 + (0.05 × 100) = 1 + 5 = 6 cycles
```

### **Multi-Level Cache AMAT**
```
AMAT = Hit_Time_L1 + Miss_Rate_L1 × Miss_Penalty_L1

where:
Miss_Penalty_L1 = Hit_Time_L2 + Miss_Rate_L2 × Miss_Penalty_L2

Full expansion:
AMAT = Hit_Time_L1 + Miss_Rate_L1 × (Hit_Time_L2 + Miss_Rate_L2 × Miss_Penalty_L2)
```

**Example**:
```
L1: 1 cycle hit, 5% miss rate
L2: 10 cycle hit, 20% local miss rate
Memory: 100 cycle access

AMAT = 1 + 0.05 × (10 + 0.20 × 100)
     = 1 + 0.05 × (10 + 20)
     = 1 + 0.05 × 30
     = 1 + 1.5
     = 2.5 cycles
```

### **Local vs Global Miss Rate**

**Local Miss Rate**: Miss rate at one level
```
Local Miss Rate = Misses at this level / Accesses to this level
```

**Global Miss Rate**: Miss rate for entire cache hierarchy
```
Global L2 Miss Rate = Miss_Rate_L1 × Local_Miss_Rate_L2
```

**Example**:
```
L1 miss rate = 5%
L2 local miss rate = 20%
Global L2 miss rate = 0.05 × 0.20 = 0.01 = 1%

This means 1% of ALL memory accesses miss in both L1 and L2
```

---

## 6. THREE TYPES OF CACHE MISSES

### **Compulsory Misses (Cold Start Misses)**
- First access to a block will always miss
- Cannot be avoided (except with prefetching)
- **How to reduce**: Larger block size (spatial locality)

### **Capacity Misses**
- Working set too large to fit in cache
- Would occur even in fully-associative cache
- **How to reduce**: Larger cache

### **Conflict Misses (Collision Misses)**
- Multiple blocks compete for same location
- Specific to direct-mapped and set-associative caches
- Would NOT occur in fully-associative cache
- **How to reduce**: Higher associativity, better replacement policy

**Example Scenario**:
```
Direct-mapped cache with 4 blocks
Access pattern: 0, 8, 0, 8, 0, 8...

If blocks 0 and 8 map to same location:
Access 0: Compulsory miss (first access)
Access 8: Conflict miss (evicts block 0)
Access 0: Conflict miss (evicts block 8)
Access 8: Conflict miss (evicts block 0)
...

Hit rate = 0% due to conflict misses!
4-way set-associative would have much better hit rate.
```

---

## 7. SIX BASIC CACHE OPTIMIZATIONS

### **Optimization 1: Larger Block Size**

**Effect on Miss Types**:
- ✓ Reduces compulsory misses (spatial locality)
- ✗ Increases capacity misses (fewer blocks fit)
- ✗ May increase conflict misses (fewer sets)
- ✗ Increases miss penalty (longer transfer time)

**When to Use**: Good for programs with high spatial locality

**Trade-off**: Miss rate goes UP if block too large relative to cache size

**Example**:
```
Cache Size  Block Size  Miss Rate
4 KB        16 bytes    8.57%
4 KB        64 bytes    7.00%
4 KB        256 bytes   9.51%  ← Too large!
```

### **Optimization 2: Larger Caches**

**Effect on Miss Types**:
- No effect on compulsory misses
- ✓ Reduces capacity misses
- ✓ Reduces conflict misses (more sets)
- ✗ Increases hit time
- ✗ Increases cost and power

**When to Use**: When capacity/conflict misses dominate

### **Optimization 3: Higher Associativity**

**Effect on Miss Types**:
- No effect on compulsory misses
- No effect on capacity misses
- ✓ Reduces conflict misses
- ✗ May increase hit time (more complex)

**Rule of Thumb**:
```
8-way set-associative ≈ fully-associative (for miss rate)
2:1 cache rule: Direct-mapped cache of size N has about the same
                miss rate as 2-way set-associative cache of size N/2
```

**Example**:
```
Given: 4 KB cache, miss rate = 9.8% (1-way), 7.1% (4-way)

Assume:
Hit_time_1way = 1.00 cycle
Hit_time_4way = 1.44 cycles (44% slower)
Miss penalty = 25 cycles

AMAT_1way = 1.00 + 0.098 × 25 = 1.00 + 2.45 = 3.45 cycles
AMAT_4way = 1.44 + 0.071 × 25 = 1.44 + 1.775 = 3.215 cycles

4-way wins despite longer hit time!
```

### **Optimization 4: Multi-Level Caches**

**Design Philosophy**:
- L1: Small and fast (minimize hit time)
- L2: Large and slower (minimize miss rate)
- L3: Even larger (if needed)

**Inclusion Property**:
- **Inclusive**: L1 ⊆ L2 (simplifies coherence, effective size = L2)
- **Exclusive**: L1 ∩ L2 = ∅ (maximizes capacity, effective size = L1 + L2)

**Example**:
```
L1: 16 KB, 2-way, 1 cycle hit, 4% miss rate
L2: 256 KB, 8-way, 10 cycle hit, 30% local miss rate
Memory: 200 cycle access

Global L2 miss rate = 0.04 × 0.30 = 0.012 (1.2%)

AMAT = 1 + 0.04 × (10 + 0.30 × 200)
     = 1 + 0.04 × 70
     = 1 + 2.8
     = 3.8 cycles

Compare to single-level 16 KB cache:
AMAT = 1 + 0.04 × 200 = 9 cycles

Multi-level is 2.4× better!
```

### **Optimization 5: Read Misses Take Priority over Writes**

**Why**: Processor must wait for read, but can buffer writes

**Techniques**:
1. **Read from write buffer**: If read misses on block in write buffer, read from buffer (don't wait for write to complete)

2. **Priority to read misses**: When both read miss and write buffer full, service read first

3. **Dirty block replacement**: When replacing dirty block on read miss:
   - Bad: Write old block to memory, then read new block
   - Good: Copy old block to write buffer, read new block immediately, write old block later

**Performance Impact**:
```
Without optimization:
Read miss with dirty eviction = Write time + Read time

With optimization:
Read miss with dirty eviction ≈ Read time
(Write happens in background)
```

### **Optimization 6: Avoid Address Translation (Virtual Caches)**

**Problem**: Translating virtual → physical address before cache lookup increases hit time

**Solution**: Use virtual address to index/tag cache

**Virtual Cache Benefits**:
- ✓ Faster hit time (no translation needed for cache access)
- ✗ Cache flush on context switch (different processes use same virtual addresses)
- ✗ Aliasing problem (two virtual addresses map to same physical address)

**Hybrid Solution**: Virtually indexed, physically tagged (VIPT)
- Use virtual address for index (fast)
- Use physical address for tag (avoids aliasing)
- Works if: Page offset bits ≥ (Index bits + Offset bits)

---

## 8. REPLACEMENT POLICIES

### **Least Recently Used (LRU)**
- Replace block not used for longest time
- Best performance for most programs
- **Hardware cost**: Need timestamp or counter for each block
  - 2-way: 1 bit (MRU bit)
  - 4-way: 5 bits (4! = 24 permutations)

### **Random**
- Replace random block in set
- Simpler hardware
- Performance close to LRU for large caches

### **First-In-First-Out (FIFO)**
- Replace oldest block in set
- Simpler than LRU
- Usually worse than LRU

**Performance Comparison** (from slides):
```
Cache   LRU    Random  FIFO
16KB    114.1  117.3   115.5  (misses per 1000 instructions)
64KB    103.4  104.3   103.9
256KB   92.2   92.1    92.5

Conclusion: For larger caches, policy matters less
```

---

## 9. WRITE STRATEGIES

### **Write-Through**

**On Write Hit**: Update both cache AND memory

**Pros**:
- ✓ Simple
- ✓ Memory always consistent
- ✓ Easy to recover from crashes

**Cons**:
- ✗ Every write goes to memory (slow)
- ✗ High memory bandwidth

**Optimization**: Use write buffer
```
Without buffer:
CPI = Base_CPI + 0.10 × 100 = 11 (10% stores, 100 cycle mem access)

With buffer:
CPI ≈ Base_CPI (writes happen in background)
Only stall if buffer full
```

### **Write-Back**

**On Write Hit**: Update cache only (set dirty bit)

**On Replacement**: If dirty bit set, write back to memory

**Pros**:
- ✓ Much lower memory bandwidth
- ✓ Multiple writes to same block → only one memory write

**Cons**:
- ✗ More complex
- ✗ Memory inconsistent with cache

**Write Allocation**:
- **Write-allocate**: Fetch block on write miss (typical for write-back)
- **No-write-allocate**: Don't fetch block (typical for write-through)

---

## 10. VIRTUAL MEMORY BASICS

### **Address Translation**
```
Virtual Address = [Virtual Page Number | Page Offset]
Physical Address = [Physical Page Number | Page Offset]

Page offset bits = log₂(Page size)
VPN bits = Virtual address bits - Page offset bits
```

**Example**:
```
32-bit virtual address
4 KB pages → Page offset = 12 bits
VPN = 32 - 12 = 20 bits → 2^20 = 1M virtual pages
```

### **Page Table**
- Maps virtual page number → physical page number
- Stored in main memory
- Very large (one entry per virtual page)

**Page Table Entry (PTE)** contains:
- Physical page number
- Valid bit (is page in memory?)
- Dirty bit (has page been modified?)
- Reference bit (for LRU replacement)
- Protection bits (read/write/execute permissions)

### **Translation Lookaside Buffer (TLB)**

**Problem**: Page table in memory → 2 memory accesses per load/store!

**Solution**: Cache for page table entries

**TLB Characteristics**:
- Small (16-512 entries typical)
- Fully-associative or highly set-associative
- Fast (parallel with L1 cache access)

**TLB Access**:
```
1. Check TLB for virtual page number
2. If TLB hit: Get physical page number from TLB
3. If TLB miss: Access page table in memory, update TLB
4. Concatenate physical page number with page offset
```

**Effective Access Time**:
```
EAT = Memory_access + Miss_rate_TLB × Page_table_access

Example:
TLB hit rate = 95%
Memory access = 100 ns
Page table access = 100 ns

EAT = 100 + 0.05 × 100 = 105 ns

Without TLB:
EAT = 100 + 100 = 200 ns

TLB provides 1.9× speedup!
```

---

## 11. WORKED EXAMPLE PROBLEMS

### **Problem 1: Cache Tag Calculation**

**Given**:
- 64 KB cache
- 4-way set-associative
- 64-byte blocks
- 32-bit addresses

**Find**: Tag bits, Index bits, Offset bits

**Solution**:
```
Step 1: Block offset
Block size = 64 bytes → Offset = log₂(64) = 6 bits

Step 2: Number of sets
Total blocks = 64 KB / 64 bytes = 1024 blocks
Sets = 1024 / 4 = 256 sets
Index = log₂(256) = 8 bits

Step 3: Tag bits
Tag = 32 - 8 - 6 = 18 bits
```

### **Problem 2: AMAT Calculation**

**Given**:
- L1: 32 KB, 1 cycle hit, 3% miss rate
- L2: 1 MB, 15 cycle hit, 10% local miss rate
- Memory: 150 cycles

**Find**: AMAT

**Solution**:
```
Global L2 miss rate = 0.03 × 0.10 = 0.003 (0.3%)

AMAT = 1 + 0.03 × (15 + 0.10 × 150)
     = 1 + 0.03 × (15 + 15)
     = 1 + 0.03 × 30
     = 1 + 0.9
     = 1.9 cycles
```

### **Problem 3: Virtual-to-Physical Translation**

**Given**:
- Page size = 1024 bytes (1 KB)
- Page table:

```
VPN  PPN
 0    3
 1    2
 2    1
 3    0
```

**Find**: Physical address for virtual address 1234

**Solution**:
```
Step 1: Find VPN and offset
1234 / 1024 → quotient = 1 (VPN), remainder = 210 (offset)

Step 2: Look up PPN
VPN 1 → PPN 2 (from table)

Step 3: Calculate physical address
Physical address = PPN × Page size + Offset
                 = 2 × 1024 + 210
                 = 2048 + 210
                 = 2258
```

### **Problem 4: Cache Access Simulation**

**Given**:
- 8-block direct-mapped cache
- 1 word/block
- Access sequence: 22, 22, 26, 16, 3, 16, 18

**Find**: Hit/miss for each access, final hit rate

**Solution**:
```
Address breakdown: [Tag | Index (3 bits) | Offset (2 bits)]

22 = 10110₂ → Index = 110 (6) → MISS (compulsory)
22 = 10110₂ → Index = 110 (6) → HIT
26 = 11010₂ → Index = 010 (2) → MISS (compulsory)
16 = 10000₂ → Index = 000 (0) → MISS (compulsory)
 3 = 00011₂ → Index = 011 (3) → MISS (compulsory)
16 = 10000₂ → Index = 000 (0) → HIT
18 = 10010₂ → Index = 010 (2) → MISS (replaces 26)

Hits = 2, Misses = 5
Hit rate = 2/7 = 28.57%
```

---

## 12. EXAM STRATEGY FOR CACHE QUESTIONS

### **What to Memorize**:
1. ✓ Cache size equation
2. ✓ Tag/Index/Offset formulas for all three cache types
3. ✓ AMAT formula (including multi-level)
4. ✓ Local vs global miss rate
5. ✓ Three types of misses

### **Common Calculation Types**:
1. **Tag/Index/Offset bits** (appears on EVERY exam)
2. **AMAT calculations** (single and multi-level)
3. **Cache access simulation** (table-filling)
4. **Miss rate analysis** (which optimization helps?)
5. **Virtual-to-physical translation**

### **Step-by-Step Problem Solving**:

**For Tag/Index/Offset**:
```
Step 1: Offset bits = log₂(Block size in bytes)
Step 2: Number of sets = Total blocks / Associativity
        Index bits = log₂(Number of sets)
Step 3: Tag bits = Address bits - Index bits - Offset bits
```

**For AMAT**:
```
Step 1: Write down formula: Hit time + Miss rate × Miss penalty
Step 2: For multi-level, expand miss penalty recursively
Step 3: Calculate from inside out
Step 4: CHECK UNITS (cycles vs ns)
```

**For Cache Simulation**:
```
Step 1: Determine address breakdown (tag/index/offset bits)
Step 2: Create cache table with columns: Index, Valid, Tag, Data
Step 3: For each access:
        - Extract index from address
        - Check valid bit and compare tag
        - Update cache on miss
Step 4: Count hits and misses
```

### **Red Flags (Common Mistakes)**:
- ✗ Forgetting that fully-associative has NO index bits
- ✗ Confusing sets with blocks (sets = blocks / associativity)
- ✗ Using wrong log base (always use log₂ for bits!)
- ✗ Confusing local and global miss rates
- ✗ Forgetting to include L1 hit time in multi-level AMAT

### **Cross-Reference to Homework**:
- **HW3 Q3**: Cache tag calculations (direct, set-assoc, fully-assoc)
- **HW4**: Cache coherence and miss types
- Formula sheet sections 4, 6, 7: Cache formulas

---

## 13. SUMMARY CHECKLIST

Before the exam, ensure you can:
- ✓ Calculate tag/index/offset bits for all three cache types
- ✓ Compute AMAT for single and multi-level caches
- ✓ Simulate cache accesses (direct-mapped)
- ✓ Identify the three types of misses
- ✓ Know which optimization reduces which miss type
- ✓ Translate virtual to physical addresses
- ✓ Calculate TLB effective access time
- ✓ Compare write-through vs write-back

**Most Likely Exam Topics** (based on HW emphasis):
1. Tag/Index/Offset calculations (EVERY exam)
2. AMAT calculations
3. Cache access simulation
4. Miss type identification

---

**Remember**: Show ALL work for partial credit! Write formulas first, then substitute values.

**Quick Reference**:
```
Cache Size = Block Size × Sets × Associativity
AMAT = Hit Time + Miss Rate × Miss Penalty
Offset bits = log₂(Block size)
Index bits = log₂(Number of sets)
Tag bits = Address bits - Index bits - Offset bits
```
