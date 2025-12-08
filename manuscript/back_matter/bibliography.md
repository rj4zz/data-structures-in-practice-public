# Bibliography and References

This bibliography lists the key resources referenced throughout the book, organized by category.

---

## Books

### Computer Architecture

**Computer Architecture: A Quantitative Approach (6th Edition)**  
*John L. Hennessy and David A. Patterson*  
Morgan Kaufmann, 2017  
The definitive reference on computer architecture, covering memory hierarchy, cache design, and performance analysis.

**Modern Processor Design: Fundamentals of Superscalar Processors**  
*John Paul Shen and Mikko H. Lipasti*  
Waveland Press, 2013  
Comprehensive coverage of modern processor microarchitecture, including cache design and memory systems.

### Performance Optimization

**Systems Performance: Enterprise and the Cloud (2nd Edition)**  
*Brendan Gregg*  
Addison-Wesley, 2020  
Comprehensive guide to performance analysis and optimization, covering profiling tools and methodologies.

**The Art of Writing Efficient Programs**  
*Fedor G. Pikus*  
Packt Publishing, 2021  
Practical guide to writing high-performance C++ code, with extensive coverage of cache optimization.

**Optimizing Software in C++**  
*Agner Fog*  
Free online resource, 2023  
https://www.agner.org/optimize/  
Detailed manual on optimizing C++ code for x86/x64 processors.

### Data Structures and Algorithms

**Introduction to Algorithms (4th Edition)**  
*Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest, and Clifford Stein*  
MIT Press, 2022  
The standard textbook on algorithms and data structures.

**The Art of Computer Programming, Volume 3: Sorting and Searching (2nd Edition)**  
*Donald E. Knuth*  
Addison-Wesley, 1998  
Comprehensive treatment of sorting, searching, and fundamental data structures.

**Data-Oriented Design**  
*Richard Fabian*  
Self-published, 2018  
Practical guide to designing software for cache efficiency and performance.

### Embedded Systems

**Embedded Systems Architecture (2nd Edition)**  
*Tammy Noergaard*  
Newnes, 2012  
Comprehensive coverage of embedded systems design, including memory management and real-time considerations.

**Programming Embedded Systems (2nd Edition)**  
*Michael Barr and Anthony Massa*  
O'Reilly Media, 2006  
Practical guide to embedded systems programming, covering bootloaders, drivers, and firmware.

### Concurrent Programming

**The Art of Multiprocessor Programming (2nd Edition)**  
*Maurice Herlihy and Nir Shavit*  
Morgan Kaufmann, 2020  
Comprehensive coverage of concurrent data structures and lock-free algorithms.

**C++ Concurrency in Action (2nd Edition)**  
*Anthony Williams*  
Manning Publications, 2019  
Practical guide to concurrent programming in C++, including lock-free data structures.

---

## Seminal Papers

### Cache-Conscious Data Structures

**"Cache-Conscious Data Structures"**  
*Jun Rao and Kenneth A. Ross*  
SIGMOD 1999  
Foundational paper on designing data structures for cache efficiency.

**"Cache Performance of Traversals and Random Accesses"**  
*Trishul M. Chilimbi, Mark D. Hill, and James R. Larus*  
ASPLOS 1999  
Analysis of cache behavior for different access patterns.

**"Cache-Oblivious Algorithms"**  
*Matteo Frigo, Charles E. Leiserson, Harald Prokop, and Sridhar Ramachandran*  
FOCS 1999  
Introduction to cache-oblivious algorithm design.

### Memory Systems

**"Hitting the Memory Wall: Implications of the Obvious"**  
*William A. Wulf and Sally A. McKee*  
ACM SIGARCH Computer Architecture News, 1995  
Classic paper on the growing gap between processor and memory performance.

**"What Every Programmer Should Know About Memory"**  
*Ulrich Drepper*  
Red Hat, Inc., 2007  
Comprehensive guide to memory hierarchy and cache behavior.

### Lock-Free Data Structures

**"Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms"**  
*Maged M. Michael and Michael L. Scott*  
PODC 1996  
The Michael-Scott lock-free queue algorithm.

**"Hazard Pointers: Safe Memory Reclamation for Lock-Free Objects"**  
*Maged M. Michael*  
IEEE TPDS, 2004  
Solution to memory reclamation in lock-free data structures.

### Memory Allocation

**"The Memory Fragmentation Problem: Solved?"**  
*Paul R. Wilson, Mark S. Johnstone, Michael Neely, and David Boles*  
ISMM 1995  
Comprehensive survey of memory allocation and fragmentation.

**"Hoard: A Scalable Memory Allocator for Multithreaded Applications"**
*Emery D. Berger, Kathryn S. McKinley, Robert D. Blumofe, and Paul R. Wilson*
ASPLOS 2000
Scalable memory allocator design.

**"TLSF: A New Dynamic Memory Allocator for Real-Time Systems"**
*Miguel Masmano, Ismael Ripoll, Alfons Crespo, and Jorge Real*
ECRTS 2004
Two-Level Segregated Fit allocator for real-time systems.

### Hash Tables and Search Structures

**"Space/Time Trade-offs in Hash Coding with Allowable Errors"**
*Burton H. Bloom*
Communications of the ACM, 1970
Original Bloom filter paper.

**"Cuckoo Filter: Practically Better Than Bloom"**
*Bin Fan, Dave G. Andersen, Michael Kaminsky, and Michael D. Mitzenmacher*
CoNEXT 2014
Improved probabilistic data structure supporting deletions.

**"The Adaptive Radix Tree: ARTful Indexing for Main-Memory Databases"**
*Viktor Leis, Alfons Kemper, and Thomas Neumann*
ICDE 2013
Cache-efficient trie structure for in-memory databases.

### String Processing

**"Fast String Searching"**
*Robert S. Boyer and J Strother Moore*
Communications of the ACM, 1977
The Boyer-Moore string search algorithm.

**"Hyperscan: A Fast Multi-pattern Regex Matcher for Modern CPUs"**
*Xiang Wang, Yang Hong, Harry Chang, KyoungSoo Park, Geoff Langdale, Jiayu Hu, and Heqing Zhu*
NSDI 2019
SIMD-optimized pattern matching.

---

## Online Resources

### Documentation

**Intel 64 and IA-32 Architectures Optimization Reference Manual**
Intel Corporation
https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html
Comprehensive optimization guide for x86/x64 processors.

**ARM Cortex-A Series Programmer's Guide**
ARM Limited
https://developer.arm.com/documentation/
Programming guide for ARM Cortex-A processors, including cache architecture.

**RISC-V Specifications**
RISC-V International
https://riscv.org/technical/specifications/
Official RISC-V ISA and platform specifications.

### Blogs and Articles

**Brendan Gregg's Blog**
https://www.brendangregg.com/
Performance analysis, profiling tools, and flamegraphs.

**Easyperf Blog**
https://easyperf.net/blog/
Performance analysis and optimization techniques.

**Mechanical Sympathy**
https://mechanical-sympathy.blogspot.com/
Hardware and software working together efficiently.

**Agner Fog's Optimization Resources**
https://www.agner.org/optimize/
Comprehensive optimization manuals and instruction tables.

### Video Courses and Talks

**Performance Ninja Class**
https://github.com/dendibakh/perf-ninja
Hands-on performance optimization exercises.

**CppCon Talks**
https://www.youtube.com/user/CppCon
Conference talks on C++ performance and optimization.

---

## Tools and Software

### Profiling Tools

**perf**
Linux profiling tool with hardware counter support
https://perf.wiki.kernel.org/

**Valgrind**
Memory debugging and profiling suite
https://valgrind.org/

**Intel VTune Profiler**
Advanced profiling for x86 processors
https://www.intel.com/content/www/us/en/developer/tools/oneapi/vtune-profiler.html

### Benchmarking Libraries

**Google Benchmark**
Microbenchmarking library for C++
https://github.com/google/benchmark

**Criterion**
Statistics-driven benchmarking library for C
https://github.com/Snaipe/Criterion

### Data Structure Libraries

**Abseil (Google)**
C++ library with optimized containers
https://abseil.io/

**Folly (Facebook)**
C++ library with high-performance data structures
https://github.com/facebook/folly

**jemalloc**
Scalable memory allocator
http://jemalloc.net/

**mimalloc (Microsoft)**
Compact general-purpose allocator
https://github.com/microsoft/mimalloc

### Source Code Examples

**Linux Kernel**
https://github.com/torvalds/linux
- `include/linux/list.h` - Intrusive doubly-linked list
- `lib/rbtree.c` - Red-black tree implementation
- `lib/prio_heap.c` - Binary heap implementation

**FreeRTOS**
https://github.com/FreeRTOS/FreeRTOS-Kernel
- `tasks.c` - Task scheduler
- `queue.c` - Queue implementation
- `list.c` - List implementation

**Redis**
https://github.com/redis/redis
- Rax (radix tree) implementation
- Bloom filter module

---

## Specifications and Standards

**RISC-V ISA Specifications**
RISC-V International
https://riscv.org/technical/specifications/

**Device Tree Specification**
Devicetree.org
https://www.devicetree.org/

**RISC-V SBI Specification**
RISC-V International
https://github.com/riscv-non-isa/riscv-sbi-doc

---

## Note on References

For detailed chapter-specific resources, including papers, books, and online materials organized by topic, please refer to **Appendix D: Further Reading**.

All URLs were verified as of December 2025. Due to the nature of online resources, some links may change over time. For updated links and errata, please visit the book's repository.

---

**Last Updated**: December 2025

