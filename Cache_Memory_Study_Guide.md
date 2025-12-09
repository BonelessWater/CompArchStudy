# Cache Memory Hierarchy - Study Guide

## Key Formulas

**Average Memory Access Time = Hit Time + (Miss Rate × Miss Penalty)**

**CPU Time = (CPU execution cycles + Memory stall cycles) × Cycle time**

**Memory Stalls = (Instructions/Program) × (Misses/Instruction) × Miss Penalty**

**Cache Size = Block Size × Number of Sets × Set Associativity**

---

## Principle of Locality

### Temporal Locality
- Items accessed recently are likely to be accessed again soon
- Examples: loop instructions, induction variables

### Spatial Locality
- Items near those accessed now are likely to be accessed soon
- Examples: sequential instruction access, array data

---

## Memory Hierarchy Levels

| Level | Size | Access Time | Bandwidth | Managed By |
|-------|------|-------------|-----------|------------|
| **Registers** | <4 KiB | 0.1-0.2 ns | 1-10 million MiB/s | Compiler |
| **Cache (L1-L3)** | 32 KiB - 8 MiB | 0.5-10 ns | 20,000-50,000 MiB/s | Hardware |
| **Main Memory** | <1 TB | 30-150 ns | 10,000-30,000 MiB/s | OS |
| **Disk** | >1 TB | 5,000,000 ns | 100-1000 MiB/s | OS |

**Performance Gap**: CPU improves ~55%/year, memory ~7%/year

---

## Three Types of Cache Misses

1. **Compulsory (Cold Start)**
   - First access to a block not in cache (unless prefetched)
   - Can only be reduced by prefetching

2. **Capacity**
   - Working set of blocks too large for cache
   - Can be reduced by larger cache

3. **Conflict**
   - Blocks evicted early due to limited set associativity
   - Can be reduced by higher associativity

---

## Cache Organization: Three Main Types

### 1. Direct-Mapped Cache (1-way)
- **Placement**: Block can go in only ONE location
- **Formula**: Frame # = (Block address) mod (# of blocks in cache)
- **Identification**: Index bits select row, then compare tag
- **Advantages**: Fastest hit time, simplest hardware
- **Disadvantages**: Highest conflict miss rate

### 2. Fully Associative Cache
- **Placement**: Block can go ANYWHERE in cache
- **Identification**: Compare tag with ALL entries in parallel
- **Advantages**: No conflict misses, lowest miss rate
- **Disadvantages**: Slowest, most expensive (comparators for every entry)

### 3. N-way Set Associative Cache
- **Placement**: Block maps to a SET, can go anywhere within that set
- **Formula**: Set # = (Block address) mod (# of sets)
- **Identification**: Index selects set, compare tag with N entries in parallel
- **Advantages**: Balance between speed and miss rate
- **Disadvantages**: More complex than direct-mapped
- **Note**: Direct-mapped = 1-way, Fully associative = all blocks in 1 set

---

## Address Breakdown

For a cache with:
- Block size = 2^(offset bits)
- Number of sets = 2^(index bits)

**Virtual/Physical Address = Tag | Index | Offset**

- **Offset bits**: Select byte within block
- **Index bits**: Select set (or block for direct-mapped)
- **Tag bits**: Remaining address bits for comparison

**Tag bits = (Total address bits) - (Index bits) - (Offset bits)**

---

## Cache Design Examples

### Direct-Mapped Example (8 blocks, word size 4 bytes)
Address 48 = 0000...0011 0000₂
- Offset: 2 bits (00) → byte within word
- Index: 3 bits (100) → selects block 4
- Tag: remaining bits (including leftmost "1" from 1100)

### 2-Way Set Associative (8 blocks → 4 sets)
Address 48 = 0000...0011 0000₂
- Offset: 2 bits (00)
- Index: 2 bits (00) → selects Set 0
- Tag: store "11" from "1100"
- Data can go in either way of Set 0

---

## Replacement Policies

Used when all frames in a set are full:

1. **Random**
   - Simple, fast, fairly effective
   - No state bits required

2. **Least Recently Used (LRU)**
   - Replace block not used for longest time
   - Best performance but requires tracking bits
   - For 4-way: 5 bits needed (4! = 24 permutations)

3. **First In First Out (FIFO)**
   - Replace oldest block
   - Simple counter implementation
   - Generally better than random for small caches

**Performance**: LRU ≥ FIFO > Random (differences minor for large caches)

---

## Write Policies

### Write-Through
- **On hit**: Update both cache AND memory
- **Advantages**: Memory always consistent, simpler
- **Disadvantages**: Slower writes
- **Solution**: Write buffer (CPU continues, buffer handles memory write)

### Write-Back
- **On hit**: Update only cache, mark block as "dirty"
- **On replacement**: Write dirty block back to memory
- **Advantages**: Faster writes, less memory traffic
- **Disadvantages**: More complex, memory inconsistent temporarily

### Write Allocation (on Write Miss)
- **Write-allocate**: Fetch block into cache, then write
- **No-write-allocate (write-around)**: Write directly to memory, don't fetch
- **Typical**: Write-back uses write-allocate, write-through can use either

---

## Six Basic Cache Optimizations

### Reducing Hit Time
1. **Give Reads Priority Over Writes**
   - Read from write buffer if block present
   - Avoid waiting for write to complete

2. **Avoid Address Translation During Indexing**
   - Use virtual addresses for cache (Virtual Cache)
   - Trade-off: aliasing issues, requires process IDs

### Reducing Miss Penalty
3. **Multilevel Caches**
   - Small fast L1, larger slower L2, even larger L3
   - **Local miss rate**: Misses at one level / Accesses to that level
   - **Global miss rate**: Miss rate(L1) × Miss rate(L2)
   - Average access time = Hit(L1) + MissRate(L1) × [Hit(L2) + MissRate(L2) × Penalty(L2)]

### Reducing Miss Rate
4. **Larger Block Size**
   - **Reduces**: Compulsory misses (spatial locality)
   - **Increases**: Miss penalty (longer transfers), capacity misses (fewer blocks fit)
   - **May increase**: Conflict misses (fewer sets)
   - **Optimal**: 32-64 bytes for most applications

5. **Larger Cache Size**
   - **Reduces**: Capacity and conflict misses
   - **No effect on**: Compulsory misses
   - **Increases**: Hit time, cost

6. **Higher Associativity**
   - **Reduces**: Conflict misses only
   - **No effect on**: Compulsory or capacity misses
   - **Slightly increases**: Hit time (more comparators)
   - **Trade-off**: 1-way often faster overall due to cycle time

---

## Multi-Level Cache Terminology

**Inclusion Property**:
- **Inclusive**: L1 ⊆ L2 (simplifies coherence, effective size = L2)
- **Exclusive**: L1 ∩ L2 = ∅ (effective size = L1 + L2)
- **Backward invalidation**: Enforce inclusion on L2 replacement

**Miss Rate Types**:
- **Local miss rate**: Misses at level / Accesses to that level
- **Global miss rate**: Misses at level / Total CPU memory accesses
  - Global L2 = MissRate(L1) × LocalMissRate(L2)

---

## Cache Performance Examples

### Example 1: Direct-Mapped vs 2-Way
Given:
- Ideal CPI = 2.0
- Memory refs/instruction = 1.5
- Cache size = 64KB
- Miss penalty = 75ns
- Direct-mapped: cycle = 1ns, miss rate = 1.4%
- 2-way: cycle = 1.25ns, miss rate = 1.0%

**Direct-mapped**:
- Miss cycles = 1.5 × 0.014 × 75 = 1.575 cycles/inst
- CPI = 2.0 + 1.575 = 3.575
- Time = 3.575 × 1ns = 3.575 ns

**2-way**:
- Miss cycles = 1.5 × 0.010 × 60 = 0.900 cycles/inst (75ns / 1.25ns)
- CPI = 2.0 + 0.900 = 2.900
- Time = 2.900 × 1.25ns = 3.625 ns

**Direct-mapped is faster** despite higher miss rate!

### Example 2: Out-of-Order with Overlap
If 30% of miss penalty overlaps with useful work:
- Effective penalty = 75ns × 0.7 = 52.5ns
- Changes the calculation significantly

---

## Split vs Unified Caches

### Split Cache (Separate I-cache and D-cache)
**Advantages**:
- Doubles memory bandwidth
- Each optimized for its access pattern
- Instructions: lower miss rate, read-only
- Data: higher miss rate, read-write

**Disadvantages**:
- More complex
- Cannot dynamically share space between instructions and data

**Typical Miss Rates** (SPEC2000):
- I-cache: 0.4%
- D-cache: 11.4%
- Weighted average: 3.2%

---

## Virtual Memory Basics

### Why Virtual Memory?
- **Memory sharing**: Multiple processes share physical memory
- **Protection**: Isolate processes from each other
- **Relocation**: Programs can run anywhere in physical memory
- **Simplicity**: Each process sees full address space

### Paging vs Segmentation

| Feature | Paging | Segmentation |
|---------|--------|--------------|
| **Size** | Fixed (e.g., 4KB, 8KB) | Variable |
| **Addressing** | Page # + Offset (concatenate) | Segment # + Offset (add) |
| **Fragmentation** | Internal only | External |
| **Placement** | Fully associative | Fully associative |
| **Replacement** | LRU (minimize page faults) | LRU |
| **Write Policy** | Always write-back (disk is slow) | Always write-back |

### Translation Lookaside Buffer (TLB)

**Purpose**: Cache for address translations (avoid repeated page table lookups)

**Operation**:
1. Check TLB for virtual page number
2. If TLB hit: Get physical page number immediately
3. If TLB miss: Access page table in memory, update TLB

**Structure**:
- Tag = Virtual page number
- Data = Physical page number + protection bits
- Small but highly effective (exploits locality)

**Example (AMD Opteron)**:
- 40 entries in data TLB
- Fully associative or set associative
- Avoids two memory accesses per data access

---

## Address Translation Example

**Given**:
- Page size = 1024 bytes = 1KB
- Virtual address = 1234

**Calculation**:
- 1234 ÷ 1024 = quotient 1, remainder 210
- Virtual page = 1, Offset = 210
- Look up page table: Virtual page 1 → Physical page 1
- Physical address = (1 × 1024) + 210 = 1234

**Given**:
- Virtual address = 3456

**Calculation**:
- 3456 ÷ 1024 = quotient 3, remainder 384
- Virtual page = 3, Offset = 384
- Look up page table: Virtual page 3 → Physical page 0
- Physical address = (0 × 1024) + 384 = 384

---

## Memory Hierarchy Design Example

**System Configuration**:
- 8 KB pages
- 256-entry TLB (direct-mapped)
- L1 cache: 8KB (direct-mapped), 64-byte blocks
- L2 cache: 4MB (direct-mapped), 64-byte blocks
- Virtual address: 64 bits
- Physical address: 41 bits

**Address Breakdown**:
- Page offset: 13 bits (8KB = 2^13)
- Block offset: 6 bits (64 bytes = 2^6)
- TLB index: 8 bits (256 entries = 2^8)
- Virtual page number: 64 - 13 = 51 bits
- L1 index: log₂(8KB/64B) = 7 bits
- L2 index: log₂(4MB/64B) = 16 bits

---

## Protection in Virtual Memory

### Mechanism
- **Base register**: Start of valid address range
- **Bound register**: End of valid address range
- **Check**: base ≤ address ≤ bound

### Modes
- **User mode**: Limited privileges
- **Kernel/Supervisor mode**: Full access

### Page Table Protection Bits
- Read/Write/Execute permissions
- Valid/Invalid bit
- Dirty bit (modified)
- Process ID (for TLB)

---

## Important Performance Considerations

1. **Block size must balance**:
   - Too small: More compulsory misses
   - Too large: Fewer blocks, more capacity/conflict misses, higher penalty

2. **Associativity trade-off**:
   - Higher associativity reduces conflicts
   - But increases hit time and energy
   - Often 1-way is fastest overall (lower cycle time)

3. **Write buffer essential**:
   - Hides write latency
   - Enables write merging
   - Must check buffer on reads (read-after-write hazard)

4. **Multi-level caches**:
   - L1 optimized for speed (small, fast, split I/D)
   - L2 optimized for miss rate (larger, unified)
   - Use global miss rate for overall performance

5. **TLB critical for performance**:
   - Virtual memory adds translation overhead
   - TLB exploits locality of translations
   - Page size affects TLB reach (coverage)

---

## Quick Reference: AMD Opteron Data Cache

- **Size**: 64 KiB
- **Associativity**: 2-way set associative
- **Block size**: 64 bytes
- **Sets**: 512 (9-bit index)
- **Operation**: 4-step read hit process
  1. Index selects set
  2. Two tags compared in parallel
  3. Block offset selects 8 bytes
  4. Data written to memory bus

---

## Key Takeaways

1. **Locality enables caching** - without it, caches wouldn't work
2. **AMAT formula drives all optimizations** - improve hit time, miss rate, or miss penalty
3. **Associativity is about conflict misses only** - doesn't help compulsory or capacity
4. **Write policies involve tradeoffs** - write-through (simple) vs write-back (fast)
5. **Multi-level caches are universal** - balance speed and capacity
6. **TLB is essential** - virtual memory would be impractical without it
7. **Direct-mapped can be fastest** - despite higher miss rate, cycle time matters
8. **Block size has optimal value** - depends on cache size and access pattern
