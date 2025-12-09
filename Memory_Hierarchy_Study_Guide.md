# Memory Hierarchy Design - Study Guide

## Key Formula
**Average Memory Access Time = Hit Time + (Miss Rate × Miss Penalty)**

---

## Four Types of Cache Misses

1. **Compulsory** - First access to a block (not in cache unless prefetched)
2. **Capacity** - Working set too large to fit in cache
3. **Conflict** - Too many blocks map to same limited set of frames
4. **Coherence** - True and false sharing misses

---

## Six Basic Cache Optimizations

### Reducing Hit Time
- **Giving Reads Priority over Writes**
- **Avoiding Address Translation during Cache Indexing**

### Reducing Miss Penalty
- **Multilevel Caches**

### Reducing Miss Rate
- **Larger Block Size** (reduces compulsory misses)
- **Larger Cache Size** (reduces capacity misses)
- **Higher Associativity** (reduces conflict misses)

---

## 11 Advanced Cache Optimizations

### Reducing Hit Time
1. **Small and Simple Caches**
   - Add smaller L0 cache between L1 and CPU
   - Direct-mapped to avoid multiple tag comparisons
   - Tag comparison after data fetch

2. **Way Prediction**
   - Predict which block in set will be accessed next
   - Fast data access + low power
   - Single tag match if prediction correct

3. **Trace Caches**
   - Store dynamic instruction sequences (including branches)
   - Breaks spatial locality constraints
   - Same instruction may be stored multiple times

### Increasing Cache Bandwidth
4. **Pipelined Caches**
   - Multiple clock cycles for hit (1 for Pentium, 2 for P3, 4 for P4)
   - Increases bandwidth, not necessarily latency
   - Higher branch misprediction penalty

5. **Multibanked Caches**
   - Adjacent words in different banks
   - Parallel access overlaps latencies
   - Allows multiple simultaneous misses

6. **Nonblocking Caches (Hit Under Miss)**
   - Process cache lookups while miss is being handled
   - Useful for dynamically scheduled CPUs
   - Can support multiple outstanding misses

### Reducing Miss Penalty
7. **Critical Word First / Early Restart**
   - **Early Restart**: Resume CPU as soon as requested word fetched
   - **Critical Word First**: Fetch requested word first, then rest of block
   - Most beneficial for large block sizes

8. **Merging Write Buffers**
   - Multiple writes to same block merged before memory write
   - Reduces write stalls
   - Must maintain memory consistency

### Reducing Miss Rate
9. **Compiler Optimizations**
   - **Loop Interchange**: Change loop order to improve spatial locality
   - **Blocking**: Access submatrices to fit in cache
   - **Merging Arrays**: Combine arrays accessed together
   - **Loop Fusion**: Combine loops accessing same data

### Reducing Miss Penalty/Rate via Parallelism
10. **Hardware Prefetching**
    - Speculatively fetch consecutive blocks when memory idle
    - Use stream buffer to avoid cache pollution
    - Low-priority, uses otherwise-unused bandwidth

11. **Compiler-Controlled Prefetching**
    - Insert special load instructions before data needed
    - Can be register or cache prefetching
    - Risk: extra conflict misses, delaying valid accesses

---

## Compiler Optimization Examples

### Loop Interchange
```
BEFORE (bad - stride of 100):
for (j = 0; j < 100; j++)
    for (i = 0; i < 5000; i++)
        x[i][j] = 2 * x[i][j]

AFTER (good - sequential access):
for (i = 0; i < 5000; i++)
    for (j = 0; j < 100; j++)
        x[i][j] = 2 * x[i][j]
```

### Blocking (for matrix operations)
- Access submatrices (blocks) that fit in cache
- Improves both spatial and temporal locality
- y matrix gets spatial locality, z matrix gets temporal locality

---

## Memory Technology

### DRAM vs SRAM
- **DRAM**: 1 transistor/bit, needs refresh, 4-8x larger, 8-16x slower, 8-16x cheaper
- **SRAM**: 4-6 transistors/bit, no refresh needed, faster, more expensive

### DRAM Variations
- **SDRAM**: Synchronous, clock-synchronized
- **DDR**: Double Data Rate (both clock edges)
- **DDR4**: 1.2V operation (vs 1.35-1.5V for DDR3)

### DRAM Organization
- **RAS (Row Access Strobe)**: First half of address
- **CAS (Column Access Strobe)**: Second half of address
- Modern DRAMs organized in banks (up to 16 for DDR4)
- Commands: ACT (Activate), PRE (Precharge)

### Flash Memory
- Non-volatile, electrically erasable
- Fast read (near DRAM speeds), slow write (10-100x slower than DRAM)
- No power needed to maintain data
- Types: NOR and NAND flash

---

## Virtual Machines

### Purpose
- **Improve Protection**: Smaller code base than OS
- **Manage Software**: Run multiple OS versions/legacy systems
- **Manage Hardware**: Server consolidation, load balancing

### Virtual Machine Monitor (VMM/Hypervisor)
- Software supporting VMs (~10,000 lines of code)
- Maps virtual resources to physical resources
- Much smaller than traditional OS

### VM Overhead
- **Processor-bound**: Near zero overhead (runs at native speed)
- **I/O-intensive**: Can have high overhead (many system calls)
- **I/O-bound**: Low overhead (processor virtualization hidden)

### Paravirtualization (Xen Example)
- Small modifications to guest OS for efficiency
- VMM at privilege level 0, guest OS at level 1, apps at level 3
- Xen mapped into upper 64MB of address space (avoid TLB flush)

---

## ARM Cortex-A53 Memory Hierarchy

### L1 Caches
- **I-cache**: 32KB, 2-way set associative, 64-byte blocks
- **D-cache**: 32KB, 4-way set associative, 64-byte blocks

### TLBs
- **L1 TLB**: Fully associative, 10 entries, 64KB page
- **L2 TLB**: 512 entries, 4-way set associative

### L2 Cache
- 1MB, 16-way set associative, 64-byte blocks

### Key Observations
- Larger memory footprints → higher miss rates
- L2 miss penalty 5x higher than L1, so L2 misses contribute significantly
- mcf is known as a "cache buster"

---

## Intel Core i7 6700

Multi-level cache hierarchy with:
- L1 instruction and data caches
- L2 unified cache
- L3 shared cache
- Advanced prefetching and prediction mechanisms

---

## Important Concepts to Remember

1. **Memory hierarchy exists due to processor-memory performance gap**
2. **Three optimization categories**: Reduce hit time, reduce miss penalty, reduce miss rate
3. **Compiler optimizations require no new hardware**
4. **Non-blocking caches enable hit-under-miss and miss-under-miss**
5. **Prefetching is speculative and energy-inefficient**
6. **DDR transfers on both clock edges**
7. **Virtual machines provide isolation with minimal overhead for CPU-bound workloads**
8. **Higher associativity reduces conflict misses but increases hit time and energy**
9. **Blocking/tiling optimizations exploit cache capacity more effectively**
10. **Stream buffers prevent prefetch pollution of active cache**

---

## Quick Reference: Memory Access Components

**Access Time** = Delay from initiation to completion  
**Cycle Time** = Minimum interval between separate requests  
**Bandwidth** = Bytes read/written per unit time  
**Latency** = Described by access time for reads
