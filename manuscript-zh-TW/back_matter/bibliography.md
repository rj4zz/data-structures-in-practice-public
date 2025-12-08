# 參考文獻

本參考文獻列出本書引用的關鍵資源，按類別組織。

---

## 書籍

### Computer Architecture

**Computer Architecture: A Quantitative Approach (6th Edition)**  
*John L. Hennessy and David A. Patterson*  
Morgan Kaufmann, 2017  
Computer architecture 的權威參考，涵蓋 memory hierarchy、cache design 和 performance analysis。

**Modern Processor Design: Fundamentals of Superscalar Processors**  
*John Paul Shen and Mikko H. Lipasti*  
Waveland Press, 2013  
現代 processor microarchitecture 的全面涵蓋，包括 cache design 和 memory systems。

### Performance Optimization

**Systems Performance: Enterprise and the Cloud (2nd Edition)**  
*Brendan Gregg*  
Addison-Wesley, 2020  
Performance analysis 和 optimization 的全面指南，涵蓋 profiling tools 和 methodologies。

**The Art of Writing Efficient Programs**  
*Fedor G. Pikus*  
Packt Publishing, 2021  
撰寫高效能 C++ code 的實用指南，廣泛涵蓋 cache optimization。

**Optimizing Software in C++**  
*Agner Fog*  
Free online resource, 2023  
https://www.agner.org/optimize/  
優化 x86/x64 processors 的 C++ code 的詳細手冊。

### Data Structures and Algorithms

**Introduction to Algorithms (4th Edition)**  
*Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest, and Clifford Stein*  
MIT Press, 2022  
Algorithms 和 data structures 的標準教科書。

**The Art of Computer Programming, Volume 3: Sorting and Searching (2nd Edition)**  
*Donald E. Knuth*  
Addison-Wesley, 1998  
Sorting、searching 和基本 data structures 的全面處理。

**Data-Oriented Design**  
*Richard Fabian*  
Self-published, 2018  
為 cache efficiency 和 performance 設計 software 的實用指南。

### Embedded Systems

**Embedded Systems Architecture (2nd Edition)**  
*Tammy Noergaard*  
Newnes, 2012  
Embedded systems design 的全面涵蓋，包括 memory management 和 real-time considerations。

**Programming Embedded Systems (2nd Edition)**  
*Michael Barr and Anthony Massa*  
O'Reilly Media, 2006  
Embedded systems programming 的實用指南，涵蓋 bootloaders、drivers 和 firmware。

### Concurrent Programming

**The Art of Multiprocessor Programming (2nd Edition)**  
*Maurice Herlihy and Nir Shavit*  
Morgan Kaufmann, 2020  
Concurrent data structures 和 lock-free algorithms 的全面涵蓋。

**C++ Concurrency in Action (2nd Edition)**  
*Anthony Williams*  
Manning Publications, 2019  
C++ concurrent programming 的實用指南，包括 lock-free data structures。

---

## 重要論文

### Cache-Conscious Data Structures

**"Cache-Conscious Data Structures"**  
*Jun Rao and Kenneth A. Ross*  
SIGMOD 1999  
為 cache efficiency 設計 data structures 的基礎論文。

**"Cache Performance of Traversals and Random Accesses"**  
*Trishul M. Chilimbi, Mark D. Hill, and James R. Larus*  
ASPLOS 1999  
不同 access patterns 的 cache behavior 分析。

**"Cache-Oblivious Algorithms"**  
*Matteo Frigo, Charles E. Leiserson, Harald Prokop, and Sridhar Ramachandran*  
FOCS 1999  
Cache-oblivious algorithm design 的介紹。

### Lock-Free Data Structures

**"Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms"**  
*Maged M. Michael and Michael L. Scott*  
PODC, 1996  
Lock-free queue 的經典論文。

**"Hazard Pointers: Safe Memory Reclamation for Lock-Free Objects"**  
*Maged M. Michael*  
IEEE TPDS, 2004  
Lock-free data structures 的 safe memory reclamation。

### B-Trees and Indexing

**"The Ubiquitous B-Tree"**  
*Douglas Comer*  
ACM Computing Surveys, 1979  
B-trees 的經典介紹。

**"Modern B-Tree Techniques"**  
*Goetz Graefe*  
Foundations and Trends in Databases, 2011  
現代 B-tree implementations 和 optimizations。

---

## 線上資源

### Tutorials and Guides

**What Every Programmer Should Know About Memory**  
*Ulrich Drepper*  
https://people.freebsd.org/~lstewart/articles/cpumemory.pdf  
Memory hierarchies 和 cache optimization 的全面指南。

**Gallery of Processor Cache Effects**  
*Igor Ostrovsky*  
http://igoro.com/archive/gallery-of-processor-cache-effects/  
Cache effects 的互動式展示。

### Documentation

**Intel 64 and IA-32 Architectures Optimization Reference Manual**  
https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html  
Intel processors 的官方 optimization guide。

**ARM Cortex-A Series Programmer's Guide**  
https://developer.arm.com/documentation/  
ARM processors 的官方文檔。

**RISC-V Specifications**  
https://riscv.org/specifications/  
RISC-V ISA 和 extensions 的官方規格。

---

## 工具和函式庫

**Google Benchmark**  
https://github.com/google/benchmark  
C++ microbenchmarking library。

**Folly**  
https://github.com/facebook/folly  
Facebook 的 C++ library，包含 cache-efficient data structures。

**jemalloc**  
https://github.com/jemalloc/jemalloc  
High-performance memory allocator。

---

## Blogs 和文章

**Brendan Gregg's Blog**  
https://www.brendangregg.com/  
Performance analysis 和 profiling 的深入文章。

**Mechanical Sympathy**  
https://mechanical-sympathy.blogspot.com/  
Hardware-aware programming 的討論。

**Agner Fog's Optimization Resources**  
https://www.agner.org/optimize/  
x86/x64 optimization 的全面資源。

---

完整參考文獻見英文版 Appendix D: Further Reading。

