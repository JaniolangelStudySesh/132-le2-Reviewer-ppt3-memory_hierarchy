# CMSC 132 — Memory Hierarchy: Summary & Study Guide

---

## 1. The Core Problem

| Problem | Explanation |
|--------|-------------|
| Can't afford large fast memory | No single technology is both big AND fast AND cheap |
| Speed gap | CPU runs at GHz; DRAM takes ~50 ns |
| Solution | **Memory Hierarchy** — appear large AND fast through layering |

---

## 2. Law of Storage (Speed vs Size vs Cost)

| Type | Size | Speed | Cost |
|------|------|-------|------|
| SRAM (small) | 512 B | sub-ns | ~$10K/GB |
| SRAM (large) | KB–MB | ~ns | ~$10K/GB |
| DRAM | GB | ~50 ns | ~$10/GB |
| SSD | TB | ~ms | ~$0.1/GB |
| Hard Disk | TB | ~10 ms | ~$0.1/GB |

> **Rule:** Bigger = Slower. Faster = More Expensive.

---

## 3. Memory Locality

| Type | Definition | Example |
|------|-----------|---------|
| **Temporal Locality** | If you accessed address A, you'll likely access A again soon | Loop variables, counters |
| **Spatial Locality** | If you accessed address A, you'll likely access nearby addresses soon | Arrays, structs |

> **Working Set** — the compact set of memory a program actively uses at any given time. Strong locality = small working set = cache works well.

---

## 4. Memoization (The Cache Principle)

| Scenario | Effect |
|---------|--------|
| **Strong reuse** | Store a few frequently used results → avoid most recomputation ✅ |
| **Poor reuse** | Store many results rarely used again → wasted space + slow lookup ❌ |

---

## 5. Memory Hierarchy Structure

```
[ fast, small ]   ← keep what you actively use here
      ↑↓
[ medium ]
      ↑↓
[ big, slow ]     ← hold what isn't being used
```

**Goal:** With strong locality, the system feels as **fast as the top** and as **large as the bottom**.

---

## 6. Modern Storage Hierarchy (Full Picture)

| Level | Size | Speed | Management |
|-------|------|-------|------------|
| Register File | 10–100 words | sub-ns | Manual (user SW / register spilling) |
| L1 Cache | ~32 KB | ~ns | Automatic (Hardware) |
| L2 Cache | ~512 KB–1 MB | many ns | Automatic (Hardware) |
| L3 Cache | several MB | more ns | Automatic (Hardware) |
| Main Memory (DRAM) | GB | ~100 ns | Automatic (HW + OS) |
| Swap Disk | 100 GB–TB | ~10 ms | Automatic (OS demand paging) |

---

## 7. Average Memory Access Time (AMAT) — KEY FORMULA

### Formula:
$$T_i = t_i + m_i \cdot T_{i+1}$$

| Symbol | Meaning |
|--------|---------|
| $T_i$ | Average access time at level $i$ |
| $t_i$ | Raw (best-case) access time at level $i$ |
| $m_i$ | Miss rate at level $i$ (probability of NOT finding data) |
| $h_i$ | Hit rate at level $i$ ($h_i + m_i = 1$) |
| $T_{i+1}$ | Average access time at the **next** (slower) level |
| $m_i \cdot T_{i+1}$ | The **"miss penalty"** |

### Step-by-Step to Solve AMAT Problems:

1. Identify all levels: $t_1, t_2, t_3$ and miss rates $m_1, m_2$
2. Start from the **bottom level** (main memory): $T_3 = t_3$ (no level below)
3. Compute upward:
   - $T_2 = t_2 + m_2 \cdot T_3$
   - $T_1 = t_1 + m_1 \cdot T_2$

### Intel P4 Example:

| Parameter | Value |
|-----------|-------|
| L1 D-cache ($t_1$) | 4 cycles (int) |
| L2 D-cache ($t_2$) | 18 cycles (int) |
| Main Memory ($t_3$) | ~180 cycles (~50 ns @ 3.6 GHz) |

**If $m_1 = 0.01$, $m_2 = 0.01$:**
$$T_2 = 18 + 0.01 \times 180 = 19.8 \text{ cycles}$$
$$T_1 = 4 + 0.01 \times 19.8 = 4.198 \approx 4.2 \text{ cycles}$$

**If $m_1 = 0.1$, $m_2 = 0.1$:**
$$T_2 = 18 + 0.1 \times 180 = 36$$
$$T_1 = 4 + 0.1 \times 36 = 7.6 \text{ cycles}$$

> **Key insight:** Even a small miss rate at L2 ($m_2 = 0.50$) causes $T_2$ to explode (108 cycles).

---

## 8. How to Optimize AMAT

| Strategy | How it helps | Tradeoff |
|---------|-------------|----------|
| **Increase cache capacity** ($C_i$↑) | Lowers $m_i$ | Increases $t_i$ (bigger = slower) |
| **Smarter replacement policy** | Anticipate what you won't need (evict it) | Hardware complexity |
| **Prefetching** | Anticipate what you will need (fetch early) | Wasted bandwidth if wrong |
| **Add intermediate levels** | Lowers $T_{i+1}$ (the miss penalty) | Cost |
| **Use faster next-level memory** | Reduces $t_{i+1}$ | Increases cost, reduces capacity |

---

## 9. DRAM vs SRAM Design

| | DRAM | SRAM |
|--|------|------|
| Optimized for | Capacity-per-dollar | Low latency at given capacity |
| Access time vs capacity | Essentially the same regardless of size | Tunable: $t = O(\sqrt{\text{capacity}})$ |
| Use in hierarchy | Main memory | Caches (L1, L2, L3) |

> **Memory hierarchy bridges the CPU–DRAM speed gap:**
> - If $T_\text{cpu} \approx T_\text{DRAM}$ → no hierarchy needed
> - If $T_\text{cpu} \ll T_\text{DRAM}$ → need one or more SRAM cache levels

---

## 10. Quick Exam Tips

| Tip | Details |
|-----|---------|
| Always compute from **bottom up** | Start with the slowest level, work toward L1 |
| Hit rate + miss rate = 1 | $h_i = 1 - m_i$ |
| Miss penalty compounds | A bad $m_2$ hurts $T_1$ even with great $m_1$ |
| $T_i \approx t_i$ is NOT the goal | Goal is desired $T_1$ within cost — not zero miss rate |
| Virtual memory acts like a cache | Between DRAM and disk (demand paging) |
