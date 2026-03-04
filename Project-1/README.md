# CS 351 Project 1 - Remote GitHub Development and Performance Monitoring

**Name:** Evode Manirahari  
**Course:** CS 351 - Computer Architecture  
**School:** Sonoma State University

---

## What This Project Does

This project looks at four different C++ programs that all do the same thing - they build a linked list and compute a hash of all the data in the nodes (like a simple blockchain simulation). The difference between each program is how they allocate memory for the nodes:

- **alloca.cpp** - uses stack allocation with recursion to keep the stack frames alive
- **list.cpp** - uses std::list and std::vector, the most "C++ way" of doing it
- **malloc.cpp** - uses C-style malloc() with placement new
- **new.cpp** - uses operator new with a manual linked list (classic CS student approach)

I ran a bunch of trials with different settings and compared their performance.

---

## Experiment Data

### Trial 1: No optimization (-g), NUM_BLOCKS=100,000, MIN_BYTES=3, MAX_BYTES=100

| Program    | Min   | Avg    | Max   |
|------------|-------|--------|-------|
| alloca.out | 0.04s | 0.048s | 0.05s |
| list.out   | 0.13s | 0.136s | 0.14s |
| malloc.out | 0.03s | 0.039s | 0.04s |
| new.out    | 0.12s | 0.126s | 0.13s |

### Trial 2: Optimized (-O2 -g2), NUM_BLOCKS=100,000, MIN_BYTES=3, MAX_BYTES=100

| Program    | Min   | Avg    | Max   |
|------------|-------|--------|-------|
| alloca.out | 0.02s | 0.020s | 0.02s |
| list.out   | 0.02s | 0.029s | 0.03s |
| malloc.out | 0.01s | 0.019s | 0.02s |
| new.out    | 0.01s | 0.026s | 0.03s |

### Trial 3: Small node size (-O2 -g2), NUM_BLOCKS=100,000, MIN_BYTES=10, MAX_BYTES=10

| Program    | Min   | Avg    | Max   |
|------------|-------|--------|-------|
| alloca.out | 0.00s | 0.009s | 0.01s |
| list.out   | 0.01s | 0.018s | 0.02s |
| malloc.out | 0.00s | 0.012s | 0.02s |
| new.out    | 0.02s | 0.020s | 0.02s |

### Trial 4: Large node size (-O2 -g2), NUM_BLOCKS=100,000, MIN_BYTES=1000, MAX_BYTES=5000

| Program    | Min   | Avg    | Max   |
|------------|-------|--------|-------|
| alloca.out | 0.61s | 0.619s | 0.63s |
| list.out   | 0.68s | 0.688s | 0.70s |
| malloc.out | 0.59s | 0.598s | 0.61s |
| new.out    | 0.67s | 0.675s | 0.70s |

### Trial 5: Short chain (-O2 -g2), NUM_BLOCKS=10,000

| Program    | Avg    |
|------------|--------|
| alloca.out | 0.000s |
| list.out   | 0.006s |
| malloc.out | 0.000s |
| new.out    | 0.004s |

### Trial 6: Long chain (-O2 -g2), NUM_BLOCKS=1,000,000

| Program    | Min   | Avg    | Max   |
|------------|-------|--------|-------|
| alloca.out | 0.13s | 0.138s | 0.14s |
| list.out   | 0.20s | 0.208s | 0.21s |
| malloc.out | 0.15s | 0.157s | 0.16s |
| new.out    | 0.19s | 0.201s | 0.21s |

### Heap Breaks (brk() calls via strace)

| Program    | 10,000 blocks | 100,000 blocks | 1,000,000 blocks |
|------------|---------------|----------------|------------------|
| alloca.out | 69            | 69             | 69               |
| list.out   | 78            | 156            | 933              |
| malloc.out | 76            | 137            | 751              |
| new.out    | 78            | 156            | 933              |

---

## Questions

### 1. Which program is fastest? Is it always the fastest?

From my trials, **malloc.out** was the fastest most of the time. In the unoptimized build it averaged 0.039s while alloca was at 0.048s and list was way behind at 0.136s. With -O2 it was still the lowest at 0.019s average.

That said, it wasn't always the fastest. At small chain lengths like 10,000 blocks, both alloca and malloc showed 0.000s - they were basically tied and too fast to measure precisely. At large node sizes malloc still led (0.598s) but alloca was close behind (0.619s). So generally malloc wins but at small scales the differences are too tiny to really matter.

### 2. Which program is slowest? Is it always the slowest?

**list.out** was the slowest in every single experiment I ran. It was never competitive. Even after optimization it lagged behind - in the unoptimized trials it was more than 3x slower than malloc. I think this makes sense because std::list has to maintain extra internal bookkeeping for the doubly-linked list structure, and std::vector inside each node adds its own overhead on top of that. The memory layout is also probably not as cache-friendly since everything is scattered around the heap.

### 3. Was there a trend in execution time based on node data size?

Yes, definitely. When the nodes were small (10 bytes each), everything ran really fast - under 0.020s for all four programs. When I bumped up to large nodes (1000-5000 bytes), the times shot up to 0.6-0.7 seconds. That is roughly a 30-60x increase for the same number of blocks.

The reason makes sense - larger nodes mean there is more data to initialize and then more bytes to iterate over when computing the hash. The std::accumulate loop has to touch every single byte in every node, so once you have big nodes that loop dominates everything. Allocation cost stops mattering as much compared to just processing all that data.

### 4. Was there a trend in execution time based on chain length?

Yes, it scales pretty linearly with chain length which is what I expected. Going from 10,000 to 100,000 blocks (10x more) gave roughly 10x more runtime. From 100,000 to 1,000,000 it was about 7-10x more. Since building the list and hashing it are both O(n) operations this makes sense - more nodes means proportionally more work.

### 5. Consider heap breaks - what is noticeable? Does stack size affect the heap?

The most interesting thing I noticed is that **alloca.out always had 69 brk() calls no matter how many blocks I used** - 10,000, 100,000, or 1,000,000 blocks all produced the same 69 breaks. That is because alloca uses the stack for node data, not the heap, so the heap never needs to grow to store nodes. The 69 calls are just from the program's normal startup and other bookkeeping.

The other three programs all showed increasing break counts as NUM_BLOCKS went up - at 1,000,000 blocks, list and new needed 933 breaks while malloc needed 751. malloc needing fewer breaks makes sense because malloc's internal allocator is better at managing heap growth - it requests larger chunks from the OS at a time compared to operator new.

Increasing the stack size does not affect heap behavior at all. They are completely separate memory regions. The stack grows down from the top, the heap grows up from the data segment. alloca's constant break count is the clearest proof of this separation.

### 6. Node diagram (malloc.cpp or alloca.cpp, 6 bytes of data)
```
head
 |
 v
+-------------------------------+       +-------------------------------+
|         Node 1                |       |         Node 2                |
|-------------------------------|       |-------------------------------|
| next     (8 bytes) -----------+-----> | next     (8 bytes) -----------> nullptr  <- tail
| numBytes (8 bytes) = 6        |       | numBytes (8 bytes) = 6        |
| bytes*   (8 bytes) ---+       |       | bytes*   (8 bytes) ---+       |
+------------------------+------+       +------------------------+------+
                         |                                       |
                         v                                       v
               +------------------+                   +------------------+
               |  6-byte block    |                   |  6-byte block    |
               |------------------|                   |------------------|
               | [0] data byte    | <- bytes ptr      | [0] data byte    | <- bytes ptr
               | [1] data byte    |    points here    | [1] data byte    |    points here
               | [2] data byte    |                   | [2] data byte    |
               | [3] data byte    |                   | [3] data byte    |
               | [4] data byte    |                   | [4] data byte    |
               | [5] data byte    |                   | [5] data byte    |
               +------------------+                   +------------------+
               (heap for malloc,                      (heap for malloc,
                stack for alloca)                      stack for alloca)

Node struct overhead = 3 fields x 8 bytes = 24 bytes
Data block = 6 bytes allocated separately
Total per node = 30 bytes
```

### 7. For each program, were any allocation/initialization/processing tasks the same?

**Processing (hashing)** is identical across all four programs. They all use the same std::accumulate loop to hash the bytes in each node, so once the list is built the traversal and hashing cost is the same for everyone.

**Initialization** is also the same - every program calls the same Node constructor which fills the data with values from getNumBytesForBlock().

**Allocation** is where they differ. list.cpp hands everything off to std::list and std::vector. new.cpp uses operator new. malloc.cpp uses malloc() then placement new to construct the Node into that memory. alloca.cpp uses alloca() to grab stack memory then placement new. So the where does memory come from step is different, but what happens after allocation is the same.

### 8. As node data size increases, does allocation significance increase or decrease?

It **decreases**. When nodes are small, the allocation call is a big chunk of the work you do per node. But when nodes are large, you have to spend way more time filling and hashing all those bytes, and the single allocation call becomes a tiny fraction of the total work.

I can see this in my data - with small nodes (10 bytes), malloc (0.012s avg) vs new (0.020s avg) is a 40% difference. With large nodes (1000-5000 bytes), malloc (0.598s) vs new (0.675s) is only about 13% difference. The allocation difference did not disappear but it matters a lot less when you are spending most of your time processing data.

---

## Build Notes

All programs were verified to produce identical hash output using make test before any benchmarking.
Stack size was set to unlimited with ulimit -s unlimited before running alloca trials.
