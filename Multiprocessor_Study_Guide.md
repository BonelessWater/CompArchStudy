# Multiprocessors and Thread-Level Parallelism - Study Guide

## Flynn's Taxonomy of Parallel Architectures

| Type | Description | Examples |
|------|-------------|----------|
| **SISD** | Single Instruction, Single Data | Uniprocessors |
| **MISD** | Multiple Instruction, Single Data | No commercial prototypes; successive refinement |
| **SIMD** | Single Instruction, Multiple Data | Illiac-IV, CM-2; Simple programming, low overhead |
| **MIMD** | Multiple Instruction, Multiple Data | Sun Enterprise, Cray T3D, SGI Origin; Most flexible |

**MIMD in Practice**: Designs with ≤ 128 processors using off-the-shelf microprocessors

---

## Levels of Parallelism

1. **Bit-Level Parallelism** (1970-1985)
   - 4-bit → 8-bit → 16-bit → 32-bit microprocessors

2. **Instruction-Level Parallelism** (1985-today)
   - Pipelining, superscalar, VLIW, out-of-order execution

3. **Thread-Level Parallelism** (Modern)
   - Server parallelism, multicore architectures
   - Requires n threads for n processors

---

## Two Main MIMD Types

### 1. Centralized Shared-Memory Multiprocessors (SMP/UMA)

**Characteristics**:
- Small number of processors share centralized memory
- **SMP**: Symmetric shared-memory multiprocessors
- **UMA**: Uniform Memory Access (same access time from all processors)
- Multiple buses/switches and memory banks
- Symmetric relationship between processors and memory

**Limitations**:
- Processor performance growth makes centralized memory less attractive
- Memory bandwidth becomes bottleneck

### 2. Distributed-Memory Multiprocessors

**Characteristics**:
- Memory distributed across processors
- **Benefits**:
  - Cost-effective memory bandwidth scaling
  - Reduced local memory access time
- **Challenges**:
  - Complex inter-processor communication
  - Higher communication latency

**Two Approaches**:

**a) DSM - Distributed Shared Memory (NUMA)**
- **NUMA**: Non-Uniform Memory Access
- Shared address space (same physical address = same location)
- Access time depends on data location

**b) Multicomputers**
- Logically disjoint address spaces
- Message passing for communication

---

## Communication Models

### Shared Memory
- Processors communicate via shared address space
- **Advantages**: Easy programming, hardware-controlled caching, low overhead
- **Best for**: Uniprocessors, small-scale multiprocessors

### Message Passing
- Processors have private memories, use RPC (Remote Procedure Calls)
- **Advantages**: Less hardware, easier design, focuses on costly non-local ops
- **Types**: Synchronous or asynchronous
- **Best for**: Large-scale systems

---

## Challenges of Parallel Processing

### 1. Limited Parallelism
- **Amdahl's Law Impact**: 80× speedup with 100 processors requires 99.75% parallelism
- **Workload Types**:
  - Commercial/multiprogramming/OS workload
  - Scientific/technical applications

### 2. High Communication Costs
- **Bandwidth**: Resources occupied during communication
- **Latency**: Sender overhead + transmission time + receiver overhead
- **Solution**: Communication latency hiding (overlap with computation)

---

## Cache Coherence

### The Problem
```
Initial: X = 0 (in memory)
Core 1: Writes X = 1 (updates cache)
Core 2: Reads X (gets 0 from its cache)
```
**Result**: Inconsistent views of memory

### Coherence Definition

**Informal**: "Any read must return the most recent write"

**Better**: "Any write must eventually be seen by a read"

**Two Rules**:
1. If P1 writes X and P2 reads it, P2 sees P1's write if read/write are "sufficiently far apart"
2. Writes to a single location are serialized (order preserved)

---

## Cache Coherence Solutions

### 1. Snooping Protocol (Snoopy Bus)
- **Mechanism**: Broadcast all requests to all processors
- **Operation**: Processors "snoop" to see if they have copy
- **Requirements**: Bus (natural broadcast medium)
- **Best for**: Small-scale machines (dominates market)

### 2. Directory-Based Protocol
- **Mechanism**: Centralized tracking of sharing information
- **Operation**: Point-to-point requests via network
- **Scalability**: Better than snooping (distributed directory)
- **Best for**: Large-scale systems
- **History**: Existed before snooping protocols

---

## Snooping Protocol States (Basic)

### Three States

1. **Invalid (I)**
   - Block contains no valid data

2. **Shared (S) - Read-only**
   - Block present in ≥1 caches
   - Memory is up-to-date
   - Can be read

3. **Exclusive (E) / Modified (M) - Read/write**
   - Cache has only copy
   - Writeable, dirty
   - Memory is out-of-date

### State Transitions

**From Invalid**:
- CPU read → Place read miss on bus → Shared
- CPU write → Place write miss on bus → Exclusive

**From Shared**:
- CPU read → Read hit (stay in Shared)
- CPU write → Place write miss on bus → Exclusive
- Write miss from another CPU → Invalid

**From Exclusive**:
- CPU read/write → Hit (stay in Exclusive)
- Read/write miss from another CPU → Write back block → Shared/Invalid

---

## MESI Protocol (4-State)

Adds **Exclusive (clean)** state to basic protocol

### States: Modified, Exclusive, Shared, Invalid

**E (Exclusive Clean)**:
- Only copy, but clean (matches memory)
- Can write without bus transaction
- No write miss needed for private data

**Transitions**:
- Read miss with no other copies → **Exclusive** (not Shared)
- Write in Exclusive state → **Modified** (no bus traffic!)
- Remote read from Modified → Write back, go to **Shared**

**Benefit**: Reduces write misses for private data

---

## Advanced Coherence Protocols

### MOESI Protocol (5-State)
- **M**: Modified (dirty, exclusive)
- **O**: Owned (dirty, but shared) - reduces delay when modified block loaded by others
- **E**: Exclusive (clean, exclusive)
- **S**: Shared (clean, shared)
- **I**: Invalid

**Owned State Benefit**: Can share dirty data without writing back to memory

---

## Four Types of Cache Misses (The 4 C's)

### 1. Compulsory
- First access to block (cold start)

### 2. Capacity
- Working set too large for cache

### 3. Conflict
- Too many blocks map to same set

### 4. Coherence Misses (New in Multiprocessors)

**a) True Sharing Miss**
- Multiple processors actually access same data
- Example: P1 writes X, P2 reads X (legitimate sharing)

**b) False Sharing Miss**
- Different processors access different words in same block
- Example:
  ```
  Block contains x1 and x2
  Time    P1          P2
  1       write x1    —
  2       —           read x2     (miss due to invalidation)
  3       write x1    —
  4       —           write x2    (miss due to invalidation)
  5       read x2     —           (miss due to invalidation)
  ```

---

## Performance Impact by Workload

### Commercial Workload (OLTP)
- **L3 access is bottleneck**
- Increasing L3 size: reduces misses but increases idle time
- Memory access components shift with cache size
- **Adding processors**: Increases true sharing misses (NOT recommended)
- **Larger blocks**: Reduce uniprocessor and true sharing misses, but DOUBLE false sharing

### Multiprogramming/OS Workload
- **Kernel characteristics**:
  - Kernel initializes all pages → High compulsory misses
  - Kernel shares data → Nontrivial coherence misses
- **Cache size impact**: Capacity misses reduce; coherence misses increase
- **Block size impact**: Compulsory misses drop drastically
- **Traffic paradox**: Miss rate drops 4×, but traffic increases 2× (block size grows 8×)

### Scientific Workload
- **Processor count**: Miss rate varies non-obviously
- **Cache size**: Usually drops miss rate (coherence dampens effect)
- **Block size**: Usually drops miss rate

---

## Directory-Based Coherence

### Architecture Components

**Three Node Types**:
1. **Local Node**: Where request originates
2. **Home Node**: Where memory location and directory entry reside
3. **Remote Node**: Has copy of requested block

### Directory States

1. **Uncached**: No processor has block
2. **Shared**: ≥1 processor has block, memory up-to-date
3. **Exclusive**: One owner has block, memory out-of-date

**Tracking**: Bit vector indicating which processors have copies (Sharers)

### Message Types

| Message | Source | Destination | Contents | Function |
|---------|--------|-------------|----------|----------|
| Read miss | Local | Home | P, A | Request data |
| Write miss | Local | Home | P, A | Request exclusive access |
| Invalidate | Home | Remote | A | Invalidate block |
| Fetch | Home | Remote | A | Request data from owner |
| Data value reply | Home/Remote | Local | D | Return requested data |
| Data write back | Remote | Home | A, D | Write dirty block back |

**Hops**: 3-4 hops typical for a transaction

---

## Synchronization

### Why Synchronize?
- Implement mutual exclusion
- Coordinate shared data access
- Conditional synchronization among processes

### Atomic Primitives

**1. Atomic Exchange**
```
Swap register with memory atomically
0 = lock free, 1 = lock taken
```

**2. Test-and-Set**
```
Test value and set if passes test (atomic)
```

**3. Fetch-and-Increment**
```
Return memory value and increment atomically
Can implement counting semaphores
```

**4. Load-Linked (LL) + Store-Conditional (SC)**
- **LL**: Returns initial value
- **SC**: Returns 1 if succeeds (no intervening stores), 0 otherwise

**Atomic Swap with LL/SC**:
```assembly
try:    mov   R3, R4          ; move exchange value
        ll    R2, 0(R1)       ; load linked
        sc    R3, 0(R1)       ; store conditional
        beqz  R3, try         ; branch if store fails
        mov   R4, R2          ; put loaded value in R4
```

**Fetch-and-Increment with LL/SC**:
```assembly
try:    ll    R2, 0(R1)       ; load linked
        addi  R3, R2, #1      ; increment
        sc    R3, 0(R1)       ; store conditional
        beqz  R3, try         ; branch if store fails
```

---

## Spin Locks

### Naive Spin Lock
```assembly
        movi  R2, #1
lockit: exch  R2, 0(R1)        ; atomic exchange
        bnez  R2, lockit       ; already locked? retry
```
**Problem**: Continuous writes invalidate all cached copies → excessive bus traffic

### Test-and-Test-and-Set
```assembly
try:    movi  R2, #1
lockit: lw    R3, 0(R1)        ; read lock variable
        bnez  R3, lockit       ; not free? spin on cache copy
        exch  R2, 0(R1)        ; try to acquire
        bnez  R2, try          ; failed? retry
```
**Benefit**: Spin on local cache copy (no bus traffic until lock changes)

### Advanced: Exponential Backoff
```assembly
        DADDUI  R3, R0, #1
lockit: LL      R2, 0(R1)
        BNEZ    R2, lockit
        DADDUI  R2, R2, #1
        SC      R2, 0(R1)
        BNEZ    R2, gotit
        DSLL    R3, R3, #1     ; double delay
        PAUSE   R3             ; delay
        J       lockit
gotit:  ; use protected data
```

### Hardware Solutions
- **Queuing Lock**: Controller picks next processor from pending requests
- **Barrier Synchronization**: Popular for parallel programs

---

## Memory Consistency Models

### The Problem
```
P1: A = 0;              P2: B = 0;
    ...                     ...
    A = 1;                  B = 1;
L1: if (B == 0) ...     L2: if (A == 0) ...
```
**Question**: Can both L1 and L2 be true simultaneously?
- Without consistency model: YES (if invalidations delayed)
- Should be: NO (impossible)

**Notation**:
- R(var)val: Read variable var returns value val
- W(var)val: Write val to variable var

---

## Consistency Model Types

### 1. Strict Consistency
**Definition**: Any read returns the most recent write

**Valid**:
```
P1: W(x)1
P2:          R(x)1  R(x)1
```

**Invalid**:
```
P1: W(x)1
P2:          R(x)0  R(x)1  ← Can't see old value after write!
```

### 2. Sequential Consistency (Most Common)
**Definition**: Execution appears as if:
- Each processor's accesses kept in order
- Accesses among processors interleaved

**Maintains**: All orderings (RW, RR, WR, WW)

**Example**:
```
P1: W(x)1
P2:          R(x)1        R(x)2
P3:          R(x)1        R(x)2
P4:                 W(x)2
```
Can be reordered to sequential execution

### 3. Relaxed Consistency Models

**Processor Consistency / Total Store Ordering**:
- Relax W→R ordering

**Partial Store Order**:
- Relax W→W ordering

**Weak Ordering**:
- Relax R→W and R→R

**Release Consistency** (Weakest):
- Uses acquire/release operations
- Most flexible implementation

---

## Ordering Requirements by Model

| Model | R→R | R→W | W→R | W→W | Special |
|-------|-----|-----|-----|-----|---------|
| **Sequential** | Yes | Yes | Yes | Yes | — |
| **Processor (TSO)** | Yes | Yes | No | Yes | — |
| **Partial Store** | Yes | Yes | No | No | — |
| **Weak** | No | No | No | No | Sync ops order all |
| **Release** | No | No | No | No | Acquire/Release |

**Key**: Weaker models = more implementation flexibility, but require explicit synchronization

---

## Memory Consistency vs Synchronization

### Synchronized Programs
Most programs ARE synchronized:
```
acquire(s)    // lock
write(x)
release(s)    // unlock
```
All shared data access ordered by sync operations → consistency not an issue

### Unsynchronized Programs (Data Races)
- Outcome depends on processor speed
- Nondeterministic behavior
- Generally should be avoided

---

## Implementation Issues

### Write Races
**Problem**: Cannot update cache until bus obtained; another processor might write meanwhile

**Solution**: Two-step process
1. Arbitrate for bus, place miss on bus
2. If miss occurs while waiting, handle it (invalidate), then restart

### Split Transaction Bus
**Problem**: Multiple outstanding transactions → two caches can grab block in Exclusive
**Solution**: Track and prevent multiple misses for one block

### Snooping Cache Implementation
**Challenge**: Every bus transaction checks cache tags; interferes with CPU access

**Solutions**:
1. **Duplicate tags**: L1 tags duplicated for parallel CPU/snoop access
2. **L2 inclusion**: L2 already duplicate; use if L2 includes L1 (L1 ⊆ L2)

---

## Modern Multiprocessor Examples

### IBM Power8
- 8 separate buses between L3 and cores
- Two sets of links for connecting larger systems
- Up to 16 chips in system

### Intel Xeon E7
- Three rings connect processors and L3 cache banks
- QPI (QuickPath Interconnect) for inter-chip links
- Up to 8 chips in system

### Sun Niagara (T1)
**Target**: Commercial server applications with high TLP, low ILP

**Design Philosophy**:
- Multicore + fine-grain multithreading
- Simple pipeline, small L1 caches, shared L2
- Metric: Performance/Watt/Square Foot

**Workload Characteristics**:
- High thread-level parallelism
- High cache miss rates
- Many unpredictable branches
- Frequent load-load dependencies

### Sun Wildfire
- Hybrid snoopy + directory protocol

---

## Key Formulas and Concepts

### Speedup Limitation
For S = speedup with N processors:
```
S ≤ 1 / ((1 - P) + P/N)
```
where P = fraction of program that is parallel

**Example**: 80× speedup with 100 processors → P ≥ 0.9975 (99.75% parallel)

### Communication Latency
```
Latency = Sender_Overhead + Transmission_Time + Receiver_Overhead
```

### Coherence Miss Classification
```
Total_Misses = Compulsory + Capacity + Conflict + Coherence
Coherence_Misses = True_Sharing + False_Sharing
```

---

## Important Trade-offs

1. **Snooping vs Directory**:
   - Snooping: Fast for small systems, doesn't scale
   - Directory: Scales well, more complex, higher latency

2. **MESI vs MSI**:
   - MESI: Extra state, less write traffic for private data
   - MSI: Simpler, more write misses

3. **Block Size**:
   - Larger: Fewer compulsory/true sharing misses
   - Larger: More false sharing, higher miss penalty

4. **Consistency Model**:
   - Stricter: Easier to reason about, harder to implement
   - Relaxed: Better performance, requires explicit synchronization

5. **Shared Memory vs Message Passing**:
   - Shared: Easier programming, needs coherence hardware
   - Message: Explicit communication, simpler hardware

---

## Quick Reference: Protocol Comparison

| Protocol | States | Benefits | Drawbacks |
|----------|--------|----------|-----------|
| **MSI** | 3 | Simple | Write misses for private data |
| **MESI** | 4 | Reduces write traffic | More complex |
| **MOESI** | 5 | Share dirty data | Most complex |

---

## Key Takeaways

1. **Coherence is critical** in shared-memory multiprocessors
2. **Snooping dominates small-scale** (bus-based) systems
3. **Directory protocols scale** to large systems
4. **False sharing is real** and impacts performance
5. **Synchronization overhead** can be bottleneck
6. **Consistency models** trade complexity for performance
7. **4 C's become 4 types**: Add coherence misses to compulsory/capacity/conflict
8. **Most programs are synchronized** → consistency model rarely matters
9. **MESI is common** in modern processors
10. **Communication latency hiding** essential for performance

---

## Exam Tips

- Know state transition diagrams for snooping protocols (MSI, MESI)
- Understand directory protocol message flow (3-4 hops)
- Be able to identify true vs false sharing in code examples
- Know LL/SC implementation of atomic operations
- Understand consistency model ordering requirements
- Remember that snooping uses broadcast, directory uses point-to-point
- Know when each coherence miss type occurs
