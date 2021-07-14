# Cache and Cache synchronized with DB

## what is cache
- cache hit
- cache miss

## why we need cache
- amdehl's law
- reduce the QPS on DB

## where is cache
- hardware
    + CPU L1, L2, L3 cache
    + TLB: Translation lookaside buffer, cache of recent translations of virtual memory to physical memory; it is part of MMU
    + disk cache: DRAM and flash memory caches frequently accessed data from disk
- software
    + virtual memory (not cache actually), requires hardware support MMU in CPU
    + mmap (in linux) (not cache actually), build virtual addresses mapped to physical addresses, skip the page cache, only once data copy from disk to memory, faster than normal file visit, which needs twice data copy: from disk to page cache (kernel), from page cache to memory (user)

## how does cache work
- rely on the locality of reference (cache memory as hardware)
    + temporal locality: same resources are accessed repeatly in short time
    + spatial locality: accessing resources are near each other

## cache algorithm?
