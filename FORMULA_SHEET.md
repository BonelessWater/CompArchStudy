# COMPUTER ARCHITECTURE FORMULA SHEET
## Final Exam - Fall 2025

---

## 1. PERFORMANCE METRICS

### CPU Time & Performance
```
CPU Time = IC × CPI × Clock Period
CPU Time = (IC × CPI) / Clock Rate
Performance = 1 / Execution Time
Speedup = Performance_new / Performance_old = Time_old / Time_new
```

### CPI Calculations
```
CPI_average = Σ(CPI_i × Frequency_i)
             = (CPI_A × %A) + (CPI_B × %B) + (CPI_C × %C) + ...

where Frequency_i = percentage of instruction class i
```

### Amdahl's Law (CRITICAL!)
```
Speedup_overall = 1 / [(1 - f_enhanced) + (f_enhanced / Speedup_enhanced)]

where:
  f_enhanced = fraction of execution time affected by enhancement
  Speedup_enhanced = speedup of the enhanced portion
```

### Performance Comparison
```
Arithmetic Mean = (1/n) × Σ(Time_i)
Weighted Arithmetic Mean = Σ(Weight_i × Time_i)
Geometric Mean = (Π Time_i)^(1/n)
```

---

## 2. RELIABILITY

### MTTF (Mean Time To Failure)
```
MTTF_system (series) = 1 / Σ(1/MTTF_i)
                     = 1 / (1/MTTF_1 + 1/MTTF_2 + ... + 1/MTTF_n)
```

### Availability
```
Availability = MTTF / (MTTF + MTTR)

where MTTR = Mean Time To Repair
```

---

## 3. PIPELINING

### Clock Period
```
Clock Period = max(all stage delays) + overhead
             = max(T_IF, T_ID, T_EX, T_MEM, T_WB) + overhead

overhead = latch delay + clock skew
```

### Pipeline Performance
```
Time_unpipelined = n × (time for all stages)
Time_pipelined = (k + n - 1) × clock period

where:
  k = number of pipeline stages
  n = number of instructions

Speedup = Time_unpipelined / Time_pipelined
        ≈ k (in ideal case with many instructions)
```

### Pipeline Efficiency
```
Efficiency = Speedup / Number_of_stages
           = Speedup / k
```

### Throughput
```
Throughput = Number_of_instructions / Total_time
           = n / [(k + n - 1) × clock_period]

Ideal Throughput = 1 / clock_period  (one instruction per cycle)
```

---

## 4. CACHE MEMORY

### Cache Organization
```
Block offset bits = log₂(block size in bytes)
Number of blocks = Cache size / Block size
```

### Direct-Mapped Cache
```
Index bits = log₂(number of blocks)
Tag bits = Address width - Index bits - Block offset bits
```

### Set-Associative Cache
```
Number of sets = Number of blocks / Associativity
Index bits = log₂(number of sets)
Tag bits = Address width - Index bits - Block offset bits
```

### Fully-Associative Cache
```
Index bits = 0
Tag bits = Address width - Block offset bits
```

### Cache Performance
```
Average Memory Access Time (AMAT) = Hit time + (Miss rate × Miss penalty)

Hit Rate = Number of hits / Total accesses
Miss Rate = Number of misses / Total accesses = 1 - Hit Rate
```

---

## 5. MEMORY HIERARCHY

### Main Memory Bandwidth & Latency
```
Bandwidth = Bytes transferred / Time
Access Time = Time from address presentation to data ready
Cycle Time = Minimum time between successive requests
```

### DDR DRAM Naming
```
Data Rate (MT/s) = 2 × I/O Clock Rate
Peak Bandwidth (MB/s) = Data Rate × 8 (for 64-bit bus)
DIMM name uses rounded bandwidth value
```

---

## 6. VIRTUAL MEMORY

### Page Table
```
Virtual Address = Virtual Page Number | Page Offset
Physical Address = Physical Page Number | Page Offset

Page offset bits = log₂(page size)
VPN bits = Virtual address width - Page offset bits
```

### TLB (Translation Lookaside Buffer)
```
Effective Access Time = Hit time_TLB + (Miss rate_TLB × Miss penalty_TLB)
```

---

## 7. INSTRUCTION-LEVEL PARALLELISM

### Tomasulo's Algorithm Timing
```
Execute start = max(Issue time + 1,
                    All source operands ready time,
                    Functional unit available time)

Execute end = Execute start + Latency - 1

Write-CDB = Execute end + 1 (if CDB available)

Commit = Write-CDB + 1 (in-order, if earlier instructions committed)
```

### Multi-Issue Processors
```
IPC (Instructions Per Cycle) = Total instructions / Total cycles
CPI = Total cycles / Total instructions = 1 / IPC
```

---

## 8. CACHE COHERENCE

### MESI States (for each cache line)
- **M (Modified)**: Dirty, exclusive copy
- **E (Exclusive)**: Clean, exclusive copy
- **S (Shared)**: Clean, possibly shared
- **I (Invalid)**: Not valid

---

## 9. CRITICAL FORMULAS FOR EXAM

### HW1 Style Problems
```
1. Amdahl's Law Speedup
2. MTTF_system = 1/(1/MTTF_1 + 1/MTTF_2 + ... + 1/MTTF_n)
3. Availability = MTTF/(MTTF + MTTR)
4. Performance comparisons using arithmetic/geometric/weighted means
5. CPI calculation with instruction mix
```

### HW2 Style Problems
```
1. Pipeline: Clock period = max(stage delays) + overhead
2. Speedup = Time_unpipelined / Time_pipelined
3. Efficiency = Speedup / k
4. Throughput = Instructions / Time
5. Hazard timing: RAW, WAR, WAW, Structural
```

### HW3 Style Problems
```
1. Cache tag bits calculation for different architectures
2. Hit rate = Hits / Total accesses
3. Tomasulo scheduling with latencies
4. LRU vs MRU replacement policy
```

### HW4 Style Problems
```
1. Cache coherence protocol state transitions (Snoopy Bus & Directory)
2. Miss type identification: Compulsory, Capacity, Conflict, Coherence
3. MESI state tracking
```

---

## 10. QUICK REFERENCE TABLE

| Concept | Formula | Notes |
|---------|---------|-------|
| CPU Time | IC × CPI / Clock Rate | Basic performance |
| Amdahl's Law | 1/[(1-f) + f/S] | MOST IMPORTANT |
| MTTF (series) | 1/Σ(1/MTTF_i) | Reliability |
| Pipeline Speedup | Ideal ≈ k stages | With many instructions |
| Cache Index | log₂(sets) | Set-associative |
| Cache Tag | Addr - Index - Offset | All cache types |
| AMAT | Hit time + MR × MP | Cache performance |

---

## 11. COMMON MISTAKES TO AVOID

1. **Amdahl's Law**: Remember f is the FRACTION affected, not the speedup
2. **MTTF**: Use reciprocals: 1/(1/MTTF₁ + 1/MTTF₂ + ...)
3. **Cache**: Index bits change with associativity, tag bits adjust accordingly
4. **Pipeline**: Remember overhead (latch + clock skew) in clock period
5. **Hazards**: RAW is most common; execution completes AFTER latency cycles
6. **Tomasulo**: Execution starts when ALL operands ready + FU available
7. **Coherence**: Track state changes for BOTH caches in multiprocessor

---

## 12. EXAM DAY CHECKLIST

✓ Know Amdahl's Law cold
✓ Cache calculations: tag/index/offset bits
✓ Pipeline timing with hazards
✓ Tomasulo scheduling rules
✓ MESI state transitions
✓ Miss type identification
✓ MTTF and Availability formulas
✓ CPI calculations with instruction mix

---

**REMEMBER**: Show ALL intermediate steps for partial credit!
