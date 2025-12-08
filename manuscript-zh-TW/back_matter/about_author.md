# 關於作者

**Danny Jiang** 是一位 system software engineer 和 technical lead，在 embedded systems、firmware development 和 performance optimization 領域有超過 20 年的經驗。目前在 **SiFive** 擔任 Benchmarking/Application Engineer，Danny 的職業生涯建立在與領先的半導體和處理器公司合作，包括 **MIPS**（在 Imagination Technologies、MIPS LLC 和 Wave Computing 旗下）、**Broadcom**、**Western Digital**、**Andes Technology** 和 **Silicon Integrated Systems (SiS)**。

在整個職業生涯中，Danny 為數百萬顆晶片的開發和部署做出了貢獻，涵蓋多個領域——從 **RISC-V** 和 **MIPS** processors 到 **SSD controllers**、**Bluetooth/IoT chipsets** 和 **x86 chipset BIOS**。他的專業知識涵蓋整個 system software stack，從低階 bootloaders 和 device drivers 到 ASIC/FPGA validation 和 system integration。

## 專業領域

Danny 專精於：

- **Processor Architecture**: RISC-V, MIPS, ARM, x86

- **System Software**: Bootloaders, firmware, device drivers, RTOS porting

- **Performance Engineering**: Benchmarking, profiling, cache optimization, hardware-aware programming

- **Embedded Systems**: IoT, SSD, wireless connectivity, real-time systems

- **Validation & Verification**: ASIC/FPGA bring-up, silicon validation, system integration

- **Technical Writing**: Documentation, training materials, technical books

**聯絡 Danny**:

- **Email**: djiang.tw@gmail.com
- **LinkedIn**: [linkedin.com/in/danny-jiang-26359644](https://www.linkedin.com/in/danny-jiang-26359644/)
- **GitHub**: https://github.com/djiangtw

**其他作品**:

- See RISC-V Run: Fundamentals
- Data Structures in Practice（本書）
- 對 RISC-V 和 embedded systems 的各種 open-source 貢獻

## 致謝

作者要感謝：

- **劉炳宏教授** 的啟發性討論，促成了這本書。我們關於教科書 data structures 與真實世界 performance 之間差距的對話，是這個專案的催化劑。

- **Open-source community** 創造了讓這本書成為可能的工具——perf、Valgrind、GCC、LLVM 和無數其他工具。

- **Performance engineering 先驅**，包括 Brendan Gregg、Fedor Pikus、Ulrich Drepper 和 Agner Fog，他們的工作塑造了這個領域並影響了這本書。

- 在 SiFive、MIPS、Andes、Broadcom、Western Digital 和 SiS 的 **同事和導師們**，分享他們的專業知識並提供本書例子的真實世界經驗。

- **Early reviewers** 對草稿章節提供寶貴 feedback，幫助改善技術準確性和清晰度。

- **家人和朋友們** 在寫作過程中的堅定支持和耐心。

## 關於本書

**「Data Structures in Practice」** 解決了 computer science 教育中的一個關鍵差距：教科書 data structures 與其在現代硬體上的真實世界 performance 之間的脫節。本書結合了：

- **Hardware-aware perspective**，基於實際 cache behavior、memory hierarchy 和 performance measurements
- **實用見解**，來自 20+ 年的 embedded systems 和 system software 開發
- **嚴格的 benchmarking**，所有 performance 主張都有實際 measurements 支持
- **真實世界 case studies**，來自 bootloaders、device drivers 和 firmware development

本書分為 5 個部分，涵蓋 foundations（memory hierarchy, benchmarking）、basic data structures（arrays, linked lists, hash tables）、trees and hierarchies（BSTs, B-trees, tries, heaps）、advanced topics（lock-free structures, strings, graphs, probabilistic structures）和 case studies（bootloader, device driver, firmware）。六個全面的附錄提供 benchmark framework reference、hardware reference、tool reference、further reading 和 hands-on exercises。

本書專注於 **實用 performance**——理解為什麼 O(log n) algorithm 可以勝過 O(1) algorithm、何時用 arrays 而不是 linked lists，以及如何設計與硬體協作而非對抗的 data structures。

本書採用 **CC BY 4.0** 授權，反映作者對 open knowledge sharing 和 accessible technical education 的承諾。

**December 2025**

