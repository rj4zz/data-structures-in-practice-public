# Chapter 1: The Performance Gap

**Part I: Foundations**

---

> "In theory, theory and practice are the same. In practice, they are not."
> — 多位電腦科學家

## 謎團

凌晨兩點，我盯著眼前的 profiling 數據，完全無法理解。

當時我正在開發一個 RISC-V SoC 的 bootloader，遇到了性能問題。Bootloader 需要從一個表格中查詢裝置配置資訊——大約 500 個項目，每個項目包含一個 32-bit 的裝置 ID 和一個指向配置資料的 pointer。聽起來很簡單。

我的同事用 hash table 實作了這個功能。「O(1) 查詢，」他很有信心地說，「沒有比這更快的了。」

但 bootloader 很慢。慢到無法接受。我們的目標是 100ms 開機時間，但實際上慢了三倍。

我嘗試了一個明顯的優化：把 hash table 換成對排序過的 array 做 binary search。Binary search 是 O(log n)，理論上比 O(1) 還差。教科書是這麼說的。我的演算法教授肯定會皺眉。

結果呢？**Bootloader 快了 40%。**

O(log n) 怎麼可能贏過 O(1)？到底發生了什麼事？

## 調查

我啟動了 `perf`，Linux 的性能分析工具，跑了兩個版本：

```bash
# Hash table 版本
$ perf stat -e cache-references,cache-misses ./bootloader_hash
  Performance counter stats:
    1,247,832  cache-references
      892,441  cache-misses  (71.5% miss rate)

# Binary search 版本
$ perf stat -e cache-references,cache-misses ./bootloader_binsearch
  Performance counter stats:
      423,156  cache-references
       89,234  cache-misses  (21.1% miss rate)
```

答案就在這裡。Hash table 的 **cache miss rate 是 71.5%**。Binary search 只有 **21.1%**。

在這個系統上，每次 cache miss 大約要花 100 個 CPU cycles。Hash table 大部分時間都在等記憶體。

O(1) 的 hash table 做的*操作*比較少，但每個操作都很*昂貴*。O(log n) 的 binary search 做的操作比較多，但每個操作都很*便宜*。

**硬體推翻了演算法。**

## 為什麼這很重要

這本書要談的就是這個落差——教科書教的東西，和你的程式碼在真實硬體上執行時實際發生的事情之間的落差。

傳統的資料結構課程教你用 Big-O complexity 思考：
- Arrays: O(1) 存取，O(n) 插入
- Linked lists: O(1) 插入，O(n) 存取
- Hash tables: O(1) 平均情況
- Binary search trees: O(log n) 操作

這些是有用的抽象概念。它們幫助我們推理大規模演算法的行為。但它們是*不完整的*。

它們假設所有記憶體存取的成本都一樣。它們假設操作是獨立發生的。它們假設一個理想化的、不存在的電腦。

真實的電腦有：
- **Memory hierarchies**: Registers、L1 cache、L2 cache、L3 cache、DRAM
- **Latency gaps**: 1 cycle vs 100+ cycles
- **Cache lines**: 一次抓取 64 bytes
- **Prefetchers**: 硬體會猜測你接下來需要什麼
- **Limited bandwidth**: 你無法一次抓取所有東西

如果你在做嵌入式系統，還有更多限制：
- **Tiny caches**: 8KB 到 64KB 很常見
- **No L3 cache**: 很多 MCU 只有 L1 或 L2
- **Slow memory**: DRAM 可能是 100MHz，不是 3GHz
- **Real-time requirements**: 最壞情況很重要，不是平均情況

## 真實的性能模型

這是現代電腦更好的心智模型：

**Time = Operations × (Computation Cost + Memory Cost)**

其中：
- **Computation Cost**: 實際的 ALU 操作（通常很便宜）
- **Memory Cost**: Cache misses、DRAM 存取（通常很昂貴）

對很多演算法來說，Memory Cost 佔主導地位。

讓我們用一個典型的嵌入式 RISC-V 系統的真實數據來量化：

| 操作 | Latency | 相對成本 |
|------|---------|----------|
| Register access | 1 cycle | 1× |
| L1 cache hit | 3-4 cycles | 3× |
| L2 cache hit | 12-15 cycles | 12× |
| L3 cache hit | 40-50 cycles | 40× |
| DRAM access | 100-200 cycles | 100× |

一次 cache miss 的成本可以等於 **100 次 register 操作**。

這意味著：
- 有良好 cache 行為的 O(n) 演算法可以贏過 cache 行為差的 O(log n) 演算法
- O(1) 的 hash table 可以輸給 O(log n) 的 binary search
- 能放進 cache 的「慢」演算法可以贏過放不進 cache 的「快」演算法

## 我們的第一個 Benchmark: Array vs Linked List

讓我們用一個簡單的實驗來具體說明。我們會比較兩種方式來加總 100,000 個整數：

1. **Array**: 連續記憶體，對 cache 很友善
2. **Linked list**: 分散的記憶體，cache 的惡夢

兩者都是 O(n)。教科書說它們的性能應該差不多。讓我們看看實際上會發生什麼。

這是 array 版本：

```c
// Array: 連續記憶體
int array[100000];
for (int i = 0; i < 100000; i++) {
    array[i] = i;
}

// 加總所有元素
long long sum = 0;
for (int i = 0; i < 100000; i++) {
    sum += array[i];
}
```

這是 linked list 版本：

```c
// Linked list: 分散的記憶體
typedef struct node {
    int value;
    struct node *next;
} node_t;

node_t *head = NULL;
for (int i = 0; i < 100000; i++) {
    node_t *node = malloc(sizeof(node_t));
    node->value = i;
    node->next = head;
    head = node;
}

// 加總所有元素
long long sum = 0;
node_t *curr = head;
while (curr) {
    sum += curr->value;
    curr = curr->next;
}
```

使用我們的 benchmark 框架（我們會在第三章詳細探討），這是結果：

```
=== Array Sequential Sum ===
Mean time:      70,147 ns
Median time:    71,724 ns
Total cycles:   17,557,410

=== Linked List Sequential Sum ===
Mean time:      179,169 ns
Median time:    160,527 ns
Total cycles:   44,740,656

Array is 2.55× faster than Linked List
```

同樣的演算法（循序加總），同樣的 O(n) complexity，但 array **快了 2.5 倍**。

為什麼？讓我們看看 cache 行為：

**Array 的存取模式**：
```
Memory:  [0][1][2][3][4][5][6][7][8][9]...
         ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑
Access:  循序、可預測
Cache:   一次抓取 64 bytes（16 個 ints）
Result:  ~94% cache hit rate
```

**Linked list 的存取模式**：
```
Memory:  [node] ... [node] ... [node] ... [node]
         ↑          ↑          ↑          ↑
Access:  隨機、無法預測（跟著 pointers 走）
Cache:   每個 node 可能在不同的 cache line
Result:  ~70% cache miss rate
```

Array 受益於 **spatial locality**——當你存取 `array[0]` 時，CPU 會抓取整個 cache line（64 bytes），其中包含 `array[0]` 到 `array[15]`。接下來的 15 次存取都是免費的。

Linked list 受害於 **pointer chasing**——每個 node 都是由 `malloc()` 分別配置的，隨機散佈在記憶體中。每次存取都可能需要抓取新的 cache line。

## Memory Hierarchy

要理解為什麼 cache 這麼重要，我們需要理解 memory hierarchy。

現代電腦不是入門課程教的簡單「CPU + RAM」模型。它們更像這樣：

```
CPU Core
  ↓ 1 cycle
Registers (32-64 registers, ~256 bytes)
  ↓ 3-4 cycles
L1 Cache (32-64 KB, split I/D)
  ↓ 12-15 cycles
L2 Cache (256 KB - 1 MB, unified)
  ↓ 40-50 cycles
L3 Cache (4-32 MB, shared) [not on all systems]
  ↓ 100-200 cycles
DRAM (GB scale)
  ↓ 10,000+ cycles
SSD/Flash
```

每一層都：
- 比下一層**更快**但**更小**
- 每 byte 的成本**更高**
- 離 CPU **更近**

速度差距是巨大的。在 1 GHz 的 RISC-V 處理器上：
- L1 cache: 3-4 nanoseconds
- DRAM: 100-200 nanoseconds
- 這是 **50 倍的差距**

打個比方，如果 L1 cache 存取是 1 秒，DRAM 存取就是 **50 秒**。這是快速回應和去泡杯咖啡的差別。

## Cache Lines: 基本單位

這是一個關鍵洞察：**CPU 不會抓取單一 bytes。它們抓取 cache lines。**

一個 cache line 通常是 64 bytes。當你存取一個 byte 時，CPU 會抓取包含那個 byte 的整個 64-byte 區塊。

這有深遠的影響：

**好處**：如果你存取附近的資料，它已經在 cache 裡了（spatial locality）
```c
// 很好：循序存取
for (int i = 0; i < n; i++) {
    sum += array[i];  // 下一個元素很可能在同一個 cache line
}
```

**壞處**：如果你存取分散的資料，每次抓取都浪費了 63 bytes
```c
// 很糟：隨機存取
for (int i = 0; i < n; i++) {
    sum += array[random()];  // 每次存取都可能 miss cache
}
```

**更糟**：如果你的資料結構佈局不好，你為用不到的資料付出代價
```c
// Linked list node: 16 bytes (4-byte value + 8-byte pointer + padding)
// Cache line: 64 bytes
// 浪費: 48 bytes (75% 的 cache line 沒用到！)
```

## Prefetching: 硬體試圖幫忙

現代 CPU 有硬體 prefetchers，會試圖預測你接下來會存取什麼。它們擅長偵測簡單的模式：

**循序存取**：Prefetcher 很喜歡
```c
for (int i = 0; i < n; i++) {
    process(array[i]);  // Prefetcher: "我看到模式了！提前抓取！"
}
```

**Strided access**：Prefetcher 可以處理
```c
for (int i = 0; i < n; i += 2) {
    process(array[i]);  // Prefetcher: "Stride 是 2，了解！"
}
```

**Pointer chasing**：Prefetcher 放棄
```c
while (node) {
    process(node->value);
    node = node->next;  // Prefetcher: "不知道下一個是什麼..."
}
```

這就是為什麼 linked lists 這麼慢——prefetcher 幫不上忙。每次 pointer dereference 都是個驚喜。

## 嵌入式系統：更嚴苛的限制

如果你在做嵌入式系統，情況更極端：

**典型的嵌入式 RISC-V MCU**：
- L1 cache: 16-32 KB（桌面系統是 32-64 KB）
- L2 cache: 128-256 KB（桌面系統是 256 KB - 1 MB）
- L3 cache: 沒有（桌面系統是 4-32 MB）
- DRAM: 100 MHz（桌面系統是 3 GHz）

有了 16 KB 的 L1 cache，你的整個 working set 需要塞進 **16 KB**，否則就會 thrash cache。

比較一下：
- 100,000 個整數（array）：400 KB → 放不進 L1
- 100,000 個 linked list nodes：1.6 MB → 連 L2 都放不進

這就是為什麼嵌入式系統開發者對資料結構的大小和佈局這麼在意。每個 byte 都很重要。

## Real-Time 考量

在嵌入式系統中，我們常常在意**最壞情況**的性能，不是平均情況。

考慮一個以 1 kHz 執行的 real-time control loop（1ms 週期）：
- 最好情況：所有資料都在 L1 cache → 50 microseconds
- 最壞情況：所有資料都在 DRAM → 500 microseconds

如果你的演算法有無法預測的 cache 行為，你就無法保證 real-time deadlines。

這就是為什麼 real-time 系統常常偏好：
- **Static allocation**: 可預測的記憶體佈局
- **Fixed-size data structures**: 不會動態調整大小
- **Simple algorithms**: 可預測的 cache 行為

即使它們在平均情況的 Big-O 分析中「比較慢」。

## 你會在這本書學到什麼

這本書會教你用硬體現實的角度思考資料結構：

**Part I: Foundations**
- Memory hierarchy 如何運作（Chapter 2）
- 如何測量和分析性能（Chapter 3）

**Part II: Basic Data Structures**
- Arrays: 對 cache 友善的基礎（Chapter 4）
- Linked lists: 何時以及如何使用（Chapter 5）
- Stacks、queues 和 ring buffers（Chapter 6）
- Hash tables: Cache-conscious 設計（Chapter 7）
- Dynamic arrays 和記憶體管理（Chapter 8）

**Part III: Trees and Hierarchies**
- Binary search trees: Cache 行為（Chapter 9）
- B-trees: Cache-conscious trees（Chapter 10）
- Tries 和 radix trees（Chapter 11）
- Heaps 和 priority queues（Chapter 12）

**Part IV: Advanced Topics**
- Lock-free data structures（Chapter 13）
- String processing（Chapter 14）
- Graphs 和 networks（Chapter 15）
- Probabilistic structures（Chapter 16）

**Part V: Case Studies**
- Bootloader data structures（Chapter 17）
- Device driver queues（Chapter 18）
- Firmware memory management（Chapter 19）
- Benchmark case studies（Chapter 20）

每一章都會包含：
- 來自嵌入式系統的**真實案例**
- 顯示實際性能的 **Benchmarks**
- 使用 profiling 工具的 **Cache 分析**
- 給你自己程式碼的**設計指南**

## 前置需求和設定

要從這本書獲得最大收穫，你應該：

**知道**：
- C 程式設計（pointers、structs、記憶體管理）
- 基本資料結構（arrays、linked lists、trees）
- 基本演算法（sorting、searching）

**有**：
- Linux 系統（建議 Ubuntu/Debian）
- GCC compiler
- 基本的 command-line 技能

**選擇性但有幫助**：
- RISC-V 開發板或 QEMU
- 嵌入式系統經驗
- 熟悉 assembly language

所有程式碼範例和 benchmarks 都可以在這裡找到：
`github.com/dannyjiang/ds-in-practice`（placeholder）

## 前方的路

在下一章，我們會深入探討 memory hierarchy。你會學到：
- Caches 在硬體層級如何運作
- Cache lines、sets 和 ways 是什麼意思
- 如何預測 cache 行為
- 如何測量 cache 性能

然後在第三章，我們會建立一個完整的 benchmarking 框架——就是本書所有測量使用的那個。

在 Part I 結束時，你會有工具和知識來分析任何資料結構的真實性能。

讓我們開始吧。

---

## Summary

凌晨兩點的謎團解開了。O(log n) 的 binary search 比 O(1) 的 hash table 快 40%，因為 cache 行為比演算法 complexity 更重要。Hash table 的 71.5% cache miss rate 對比 binary search 的 21.1% 解釋了一切。硬體推翻了演算法。

傳統的資料結構課程教你用 Big-O complexity 思考：arrays 是 O(1) 存取，linked lists 是 O(1) 插入，hash tables 是 O(1) 平均情況，binary search trees 是 O(log n) 操作。這些是有用的抽象概念，幫助我們推理大規模演算法的行為。但它們是不完整的——它們假設所有記憶體存取的成本都一樣，假設操作是獨立發生的，假設一個理想化的、不存在的電腦。

真實的電腦有 memory hierarchies（registers、L1 cache、L2 cache、L3 cache、DRAM），有巨大的 latency gaps（1 cycle vs 100+ cycles），有 cache lines（一次抓取 64 bytes），有 prefetchers（硬體會猜測你接下來需要什麼），還有 limited bandwidth（你無法一次抓取所有東西）。對很多演算法來說，Memory Cost 佔主導地位。一次 cache miss 的成本可以等於 100 次 register 操作。

這意味著有良好 cache 行為的 O(n) 演算法可以贏過 cache 行為差的 O(log n) 演算法，O(1) 的 hash table 可以輸給 O(log n) 的 binary search，能放進 cache 的「慢」演算法可以贏過放不進 cache 的「快」演算法。我們的第一個 benchmark 證明了這一點：同樣是 O(n) 的循序加總，array 比 linked list 快 2.5 倍，因為 array 受益於 spatial locality（94% cache hit rate），而 linked list 受害於 pointer chasing（70% cache miss rate）。

如果你在做嵌入式系統，情況更極端。典型的嵌入式 RISC-V MCU 只有 16-32 KB 的 L1 cache（桌面系統是 32-64 KB），128-256 KB 的 L2 cache（桌面系統是 256 KB - 1 MB），沒有 L3 cache（桌面系統是 4-32 MB），DRAM 只有 100 MHz（桌面系統是 3 GHz）。有了 16 KB 的 L1 cache，你的整個 working set 需要塞進 16 KB，否則就會 thrash cache。這就是為什麼嵌入式系統開發者對資料結構的大小和佈局這麼在意——每個 byte 都很重要。

在 real-time 系統中，我們常常在意最壞情況的性能，不是平均情況。如果你的演算法有無法預測的 cache 行為，你就無法保證 real-time deadlines。這就是為什麼 real-time 系統常常偏好 static allocation（可預測的記憶體佈局）、fixed-size data structures（不會動態調整大小）和 simple algorithms（可預測的 cache 行為），即使它們在平均情況的 Big-O 分析中「比較慢」。

這本書會教你用硬體現實的角度思考資料結構。在 Part I，你會學習 memory hierarchy 如何運作以及如何測量和分析性能。在 Part II-IV，你會學習如何設計 cache-conscious 的 arrays、linked lists、hash tables、trees 和 advanced data structures。在 Part V，你會看到來自 bootloader、device driver 和 firmware 的真實案例。每一章都會包含真實案例、實際性能的 benchmarks、使用 profiling 工具的 cache 分析，以及給你自己程式碼的設計指南。

The performance gap 是真實的。教科書說 O(1) hash table 贏過 O(log n) binary search，但現實中 cache 行為可以逆轉這個結果。教訓很簡單：測量，不要假設。在下一章，我們會詳細探討 memory hierarchy，學習 caches 在硬體層級如何運作。

