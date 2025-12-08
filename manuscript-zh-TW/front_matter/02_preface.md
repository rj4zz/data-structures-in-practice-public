# 序言

## 為什麼寫這本書

我在 system software 領域工作超過 20 年——bootloaders、device drivers、firmware 和 embedded systems。在這段時間裡，我學到教科書教的 data structures 在真實硬體上跑時，往往無法達到預期的 performance。

問題不在於教科書是錯的。Big-O complexity analysis 是正確且重要的。問題在於它不完整。現代電腦有複雜的 memory hierarchies，一次 cache miss 的代價可能相當於 100 次 register operations。在這種環境下，一個有良好 cache behavior 的 O(log n) algorithm 很容易勝過一個 cache behavior 不佳的 O(1) algorithm。

這本書彌補了這個差距。它從 hardware-aware 的角度教 data structures，展示如何設計和實作在真實 silicon 上表現良好的 data structures，而不只是在理論分析中。

## 這本書適合誰

這本書是為以下讀者寫的：

- **System software engineers**，需要理解 data structures 如何與硬體互動
- **Embedded systems developers**，在受限資源和 real-time requirements 下工作
- **Performance-conscious programmers**，想理解為什麼他們的 code 很慢

你應該熟悉：
- C programming（pointers, structs, memory management）
- 基本 data structures（arrays, linked lists, trees）
- 基本 algorithms（sorting, searching）
- Command-line tools 和 compilation

你不需要：
- 進階 algorithms 知識
- Computer architecture 專業知識（我們會教你需要的）
- Assembly language（必要時我們會介紹）

## 關於故事

這本書用 narrative-driven examples 讓技術概念具體且易記。每章開頭都有一個故事，說明真實世界的問題，然後用實際 measurements 和 profiling data 來探討解決方案。

本書的場景分為兩類：

**真實案例**：許多故事基於 embedded systems 和 system software 開發的實際工作經驗。技術細節——performance numbers、cache behavior、hardware constraints——都是真實的。有些場景為了保護專有資訊而被一般化，但技術實質保持準確。

**模擬場景**：有些例子是專門為了說明技術要點而構建的。雖然不是來自實際專案，但這些場景基於現實的工程情境和合理的技術限制。它們代表 embedded systems 和 system software 開發中常見的問題。

**重要**：無論真實或模擬，所有場景都避免虛構的細節，如地點、客戶名稱或過度戲劇化的時間線。焦點始終在技術真實性和現實的工程情境。

**所有 benchmark results、performance measurements 和 hardware behavior 都基於實際測試或文件化的規格。** 當你在本書看到數字時，它們來自真實硬體上的真實測量。

## 如何閱讀這本書

**順序閱讀**：本書設計為從頭到尾閱讀。早期章節建立基礎（memory hierarchy, benchmarking），後面章節在此基礎上建構。

**參考閱讀**：每章也足夠獨立，可以作為參考。如果你需要理解 hash tables 或 lock-free queues，可以直接跳到該章。

**Code examples**：所有 code examples 都在本書的 repository 中。它們設計為可以在標準 Linux systems 上 compile 和執行。許多 examples 也可以在 embedded systems 上稍作修改即可運作。

**Benchmarks**：本書包含完整的 benchmarking framework。你可以重現所有 measurements 並實驗變化。

## 你會學到什麼

讀完這本書，你會理解：

- Memory hierarchy 如何影響 data structure performance
- 何時用 arrays vs. linked lists（提示：幾乎總是 arrays）
- 如何設計 cache-friendly data structures
- 為什麼 hash tables 往往比 binary search 慢
- 如何正確實作 lock-free data structures
- 如何測量和 profile data structure performance
- 如何為 embedded systems 選擇正確的 data structure

更重要的是，你會學到 **measure, don't assume**。本書的每個 optimization 主張都有實際 benchmark results 支持。

## 致謝

這本書的存在要感謝許多人的啟發和支持。

首先，我要感謝 **劉炳宏教授** 的深入討論，激發了這本書的想法。我們關於教科書 data structures 與真實世界 performance 之間差距的對話，種下了成長為這個專案的種子。他鼓勵我彌合理論與實踐的差距，這是無價的。

我感謝 **open-source community** 創造了讓這本書成為可能的工具——perf、Valgrind、GCC、LLVM 和無數其他工具。Open-source software 的透明度讓我們能在最深層次理解 performance。

感謝透過 blogs、papers 和 conference talks 分享知識的工程師們。**Brendan Gregg** 在 performance analysis、**Fedor Pikus** 在 C++ optimization、**Ulrich Drepper** 在 memory systems 以及許多其他人的工作，塑造了我的理解並影響了這本書。

我感謝在 **SiFive**、**MIPS**、**Andes Technology**、**Broadcom**、**Western Digital** 和 **SiS** 的同事們，他們提供了本書例子的真實世界經驗。我們一起解決的問題——從 bootloader optimization 到 firmware debugging——是這裡實用見解的基礎。

感謝 **early reviewers** 對草稿章節提供 feedback。你們的建議改善了材料的技術準確性和清晰度。

最後，感謝我的 **家人** 在許多晚上和週末寫作期間的耐心和支持。這本書和你們一樣屬於我。

## 關於作者

Danny Jiang 在 system software engineering 有超過 20 年的經驗，專精於 embedded systems、bootloaders、device drivers 和 firmware development。他在 RISC-V、ARM 和 x86 architectures 上工作過，從微小的 microcontrollers 到 application processors。

---

**讓我們開始吧。**

