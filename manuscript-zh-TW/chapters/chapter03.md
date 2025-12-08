# Chapter 3: Benchmarking and Profiling

**Part I: Foundations**

---

> "In God we trust. All others must bring data."
> — W. Edwards Deming

## 測量問題

在第二章學習了 memory hierarchy 之後，你可能很想優化你的程式碼。但有個問題：**你怎麼知道你的優化真的有效？**

我是用痛苦的方式學到這個教訓的。

我當時正在優化一個 bootloader 的 hash table 實作。基於我對 cache 行為的理解，我重寫了 hash function，讓它「更 cache-friendly」。我很有信心它會更快。

我跑了程式碼。感覺更快了。我 commit 了這個改動。

一週後，一個同事跑了 benchmarks，發現我的「優化」讓程式碼**慢了 15%**。我優化了錯誤的東西，而且我沒有資料來證明我的假設。

**教訓**：永遠不要相信你的直覺。永遠要測量。

這一章是關於如何正確測量。我們會建立一個完整的 benchmarking framework，並學習如何有效使用 profiling 工具。

## High-Precision Timing

第一個挑戰：你如何準確測量時間？

**不好的做法**：使用 `time()`

```c
time_t start = time(NULL);
run_test();
time_t end = time(NULL);
printf("Time: %ld seconds\n", end - start);
```

**問題**：1 秒的解析度。對快速操作來說沒用。

**更好的做法**：使用 `clock_gettime()`

```c
struct timespec start, end;
clock_gettime(CLOCK_MONOTONIC, &start);
run_test();
clock_gettime(CLOCK_MONOTONIC, &end);

long ns = (end.tv_sec - start.tv_sec) * 1000000000L +
          (end.tv_nsec - start.tv_nsec);
printf("Time: %ld ns\n", ns);
```

**優點**：
- Nanosecond 解析度
- `CLOCK_MONOTONIC`：不受系統時間變更影響
- 可移植（POSIX）

**最好的做法**：使用 CPU cycle counters

在 RISC-V 上：
```c
static inline uint64_t rdcycle(void) {
    uint64_t cycles;
    asm volatile ("rdcycle %0" : "=r" (cycles));
    return cycles;
}

uint64_t start = rdcycle();
run_test();
uint64_t end = rdcycle();
printf("Cycles: %lu\n", end - start);
```

在 x86_64 上：
```c
static inline uint64_t rdtsc(void) {
    uint32_t lo, hi;
    asm volatile ("rdtsc" : "=a" (lo), "=d" (hi));
    return ((uint64_t)hi << 32) | lo;
}
```

在 ARM64 上：
```c
static inline uint64_t rdcycle(void) {
    uint64_t val;
    asm volatile("mrs %0, pmccntr_el0" : "=r"(val));
    return val;
}
```

**優點**：
- 最高精度（1 cycle）
- 直接的硬體測量
- 沒有 system call overhead

**缺點**：
- 架構特定
- 可能受 frequency scaling 影響
- 可能需要 kernel 配置（ARM）

## 統計分析

單次測量是不夠的。你需要多次測量並做統計分析。

**為什麼？**

1. **Noise**：系統有背景活動
2. **Cache 狀態**：第一次執行可能是 cold cache
3. **Branch prediction**：第一次執行可能 miss predictions
4. **Frequency scaling**：CPU 頻率可能變化

**基本統計**：

```c
#define ITERATIONS 1000

uint64_t measurements[ITERATIONS];
for (int i = 0; i < ITERATIONS; i++) {
    uint64_t start = rdcycle();
    run_test();
    uint64_t end = rdcycle();
    measurements[i] = end - start;
}

// 計算統計數據
uint64_t min = measurements[0];
uint64_t max = measurements[0];
uint64_t sum = 0;

for (int i = 0; i < ITERATIONS; i++) {
    if (measurements[i] < min) min = measurements[i];
    if (measurements[i] > max) max = measurements[i];
    sum += measurements[i];
}

uint64_t mean = sum / ITERATIONS;
printf("Min: %lu, Max: %lu, Mean: %lu\n", min, max, mean);
```

**更好的做法**：加上 median 和 standard deviation

```c
// 排序以計算 median
qsort(measurements, ITERATIONS, sizeof(uint64_t), compare_uint64);
uint64_t median = measurements[ITERATIONS / 2];

// 計算 standard deviation
double variance = 0;
for (int i = 0; i < ITERATIONS; i++) {
    double diff = (double)measurements[i] - (double)mean;
    variance += diff * diff;
}
double stddev = sqrt(variance / ITERATIONS);

printf("Median: %lu cycles\n", median);
printf("Std dev: %.2f cycles\n", stddev);
```

**應該報告什麼？**
- **Minimum**：最佳情況性能（warm cache）
- **Median**：典型性能（比 mean 更穩健）
- **Standard deviation**：變異性（越低越好）
- **Maximum**：最壞情況（對 real-time 系統很重要）

## Benchmark Framework 設計

讓我們建立一個可重複使用的 framework。這是介面：

```c
typedef struct {
    const char *name;
    void (*setup)(void);
    void (*run)(void);
    void (*teardown)(void);
} benchmark_t;

void benchmark_run(benchmark_t *bench, int iterations);
```

**實作**：

```c
void benchmark_run(benchmark_t *bench, int iterations) {
    uint64_t *times = malloc(iterations * sizeof(uint64_t));

    printf("Running benchmark: %s\n", bench->name);

    // Warmup run
    if (bench->setup) bench->setup();
    bench->run();
    if (bench->teardown) bench->teardown();

    // 實際測量
    for (int i = 0; i < iterations; i++) {
        if (bench->setup) bench->setup();

        uint64_t start = rdcycle();
        bench->run();
        uint64_t end = rdcycle();

        if (bench->teardown) bench->teardown();

        times[i] = end - start;
    }

    // 計算並報告統計數據
    report_statistics(bench->name, times, iterations);

    free(times);
}
```

**使用範例**：

```c
// 測試資料
int array[1000];

void setup_array(void) {
    for (int i = 0; i < 1000; i++) {
        array[i] = i;
    }
}

void test_sequential_access(void) {
    volatile int sum = 0;
    for (int i = 0; i < 1000; i++) {
        sum += array[i];
    }
}

benchmark_t bench = {
    .name = "Sequential Array Access",
    .setup = setup_array,
    .run = test_sequential_access,
    .teardown = NULL
};

benchmark_run(&bench, 1000);
```

## 使用 perf 做 Cache 分析

Timing 告訴你**花了多久**，但不告訴你**為什麼**。為此，你需要 cache 分析。

**Linux perf** 是性能分析的標準工具：

```bash
# 基本的 cache 統計
$ perf stat -e cache-references,cache-misses ./program
  Performance counter stats:
    1,234,567 cache-references
       12,345 cache-misses              #    1.00% of all cache refs
```

**有用的 events**：
- `cache-references`: 總 cache 存取次數
- `cache-misses`: Cache misses（所有層級）
- `L1-dcache-loads`: L1 data cache loads
- `L1-dcache-load-misses`: L1 data cache load misses
- `LLC-loads`: Last-level cache loads
- `LLC-load-misses`: Last-level cache load misses

**詳細分析**：

```bash
# L1 cache 分析
$ perf stat -e L1-dcache-loads,L1-dcache-load-misses ./program
  Performance counter stats:
    10,000,000 L1-dcache-loads
       100,000 L1-dcache-load-misses    #    1.00% miss rate

# 所有 cache 層級
$ perf stat -e cache-references,cache-misses,\
L1-dcache-loads,L1-dcache-load-misses,\
LLC-loads,LLC-load-misses ./program
```

**比較不同實作**：

```bash
# Array 版本
$ perf stat -e cache-misses ./array_test
    1,234 cache-misses

# Linked list 版本
$ perf stat -e cache-misses ./list_test
   45,678 cache-misses              # 37 倍的 misses！
```

## 整合 perf 到 Benchmarks

我們可以把 perf 測量整合到 benchmark framework：

```c
typedef struct {
    uint64_t cycles;
    uint64_t cache_references;
    uint64_t cache_misses;
    uint64_t l1_loads;
    uint64_t l1_misses;
} perf_counters_t;

void benchmark_run_with_perf(benchmark_t *bench, int iterations) {
    // 設定 perf counters
    int fd_cache_ref = perf_event_open(PERF_COUNT_HW_CACHE_REFERENCES);
    int fd_cache_miss = perf_event_open(PERF_COUNT_HW_CACHE_MISSES);

    // 執行 benchmark
    perf_counters_t counters = {0};

    for (int i = 0; i < iterations; i++) {
        if (bench->setup) bench->setup();

        // 讀取起始 counters
        uint64_t start_ref = read_counter(fd_cache_ref);
        uint64_t start_miss = read_counter(fd_cache_miss);
        uint64_t start_cycles = rdcycle();

        bench->run();

        // 讀取結束 counters
        uint64_t end_cycles = rdcycle();
        uint64_t end_miss = read_counter(fd_cache_miss);
        uint64_t end_ref = read_counter(fd_cache_ref);

        if (bench->teardown) bench->teardown();

        counters.cycles += end_cycles - start_cycles;
        counters.cache_references += end_ref - start_ref;
        counters.cache_misses += end_miss - start_miss;
    }

    // 報告結果
    printf("Benchmark: %s\n", bench->name);
    printf("  Cycles: %lu\n", counters.cycles / iterations);
    printf("  Cache refs: %lu\n", counters.cache_references / iterations);
    printf("  Cache misses: %lu (%.2f%%)\n",
           counters.cache_misses / iterations,
           100.0 * counters.cache_misses / counters.cache_references);
}
```

## 常見陷阱

**1. Compiler optimizations**

Compiler 可能會把你的 benchmark 優化掉：

```c
// 不好：Compiler 會把這個優化掉
void test(void) {
    int sum = 0;
    for (int i = 0; i < 1000; i++) {
        sum += array[i];
    }
    // sum 從未被使用，整個 loop 被移除！
}

// 好：使用 volatile 或 return value
void test(void) {
    volatile int sum = 0;  // 防止優化
    for (int i = 0; i < 1000; i++) {
        sum += array[i];
    }
}
```

**2. Cold vs warm cache**

第一次執行總是比較慢（cold cache）：

```c
// 第一次執行：cold cache
run_test();  // 10,000 cycles

// 第二次執行：warm cache
run_test();  // 1,000 cycles
```

**解決方案**：永遠做 warmup run，或報告 cold 和 warm 兩種性能。

**3. Measurement overhead**

Timing 程式碼本身也要花時間：

```c
uint64_t start = rdcycle();  // ~10 cycles
uint64_t end = rdcycle();    // ~10 cycles
printf("Overhead: %lu\n", end - start);  // ~20 cycles
```

**解決方案**：測量 overhead 並減去它，或確保測試執行夠久，讓 overhead 可以忽略。

**4. System noise**

OS interrupts、其他 processes、frequency scaling 都會增加 noise。

**解決方案**：
- 執行很多次 iterations
- 報告 median（對 outliers 穩健）
- 停用 frequency scaling：`cpupower frequency-set -g performance`
- Pin 到 CPU core：`taskset -c 0 ./program`
- 提高 priority：`nice -n -20 ./program`

## 嵌入式系統考量

在嵌入式系統上做 benchmarking 有獨特的挑戰：

**1. 有限的 profiling 工具**

很多嵌入式系統沒有 `perf` 或類似工具。

**解決方案**：直接透過 memory-mapped registers 使用 hardware performance counters。

**範例**（RISC-V）：
```c
// 啟用 performance counters（machine mode）
#define CSR_MCOUNTEREN 0x306
#define CSR_MCOUNTINHIBIT 0x320

void enable_perf_counters(void) {
    // 允許 user mode 讀取 counters
    asm volatile ("csrw %0, %1" :: "i"(CSR_MCOUNTEREN), "r"(0x7));
    // 啟用所有 counters
    asm volatile ("csrw %0, %1" :: "i"(CSR_MCOUNTINHIBIT), "r"(0x0));
}
```

**2. 沒有 operating system**

Bare-metal 系統沒有 `clock_gettime()`。

**解決方案**：使用 hardware timers 或 cycle counters。

```c
// 使用 SoC timer（範例）
#define TIMER_BASE 0x10000000
#define TIMER_MTIME (*(volatile uint64_t*)(TIMER_BASE + 0x00))

uint64_t get_time_us(void) {
    return TIMER_MTIME;  // 假設 1 MHz timer
}
```

**3. Real-time 限制**

在 real-time 系統中，**最壞情況**比平均情況更重要。

**解決方案**：報告 maximum time 和 99th percentile。

```c
// 排序 times
qsort(times, iterations, sizeof(uint64_t), compare);

uint64_t min = times[0];
uint64_t max = times[iterations - 1];
uint64_t p50 = times[iterations / 2];
uint64_t p99 = times[(iterations * 99) / 100];

printf("Min: %lu cycles\n", min);
printf("P50: %lu cycles\n", p50);
printf("P99: %lu cycles\n", p99);
printf("Max: %lu cycles\n", max);
```

**4. 有限的記憶體**

無法儲存數千次測量。

**解決方案**：使用 online algorithms（running statistics）。

```c
typedef struct {
    uint64_t count;
    uint64_t min;
    uint64_t max;
    double mean;
    double m2;  // 用於 variance 計算
} running_stats_t;

void update_stats(running_stats_t *stats, uint64_t value) {
    stats->count++;

    if (value < stats->min) stats->min = value;
    if (value > stats->max) stats->max = value;

    // Welford's online algorithm for mean and variance
    double delta = value - stats->mean;
    stats->mean += delta / stats->count;
    double delta2 = value - stats->mean;
    stats->m2 += delta * delta2;
}

double get_stddev(running_stats_t *stats) {
    return sqrt(stats->m2 / stats->count);
}
```

## 實際範例：Array vs Linked List

讓我們把所有東西整合在一起，用一個完整的 benchmark 比較 arrays 和 linked lists：

```c
#define SIZE 1000
#define ITERATIONS 1000

// Array 實作
int array[SIZE];

void setup_array(void) {
    for (int i = 0; i < SIZE; i++) {
        array[i] = i;
    }
}

void test_array_sequential(void) {
    volatile int sum = 0;
    for (int i = 0; i < SIZE; i++) {
        sum += array[i];
    }
}

// Linked list 實作
typedef struct node {
    int value;
    struct node *next;
} node_t;

node_t *list_head = NULL;

void setup_list(void) {
    list_head = NULL;
    for (int i = SIZE - 1; i >= 0; i--) {
        node_t *node = malloc(sizeof(node_t));
        node->value = i;
        node->next = list_head;
        list_head = node;
    }
}

void test_list_sequential(void) {
    volatile int sum = 0;
    node_t *curr = list_head;
    while (curr) {
        sum += curr->value;
        curr = curr->next;
    }
}

void teardown_list(void) {
    node_t *curr = list_head;
    while (curr) {
        node_t *next = curr->next;
        free(curr);
        curr = next;
    }
}

// 執行 benchmarks
int main(void) {
    benchmark_t benchmarks[] = {
        {
            .name = "Array Sequential",
            .setup = setup_array,
            .run = test_array_sequential,
            .teardown = NULL
        },
        {
            .name = "List Sequential",
            .setup = setup_list,
            .run = test_list_sequential,
            .teardown = teardown_list
        }
    };

    for (int i = 0; i < 2; i++) {
        benchmark_run_with_perf(&benchmarks[i], ITERATIONS);
    }

    return 0;
}
```

**預期輸出**：

```
Benchmark: Array Sequential
  Cycles: 1,234
  Cache refs: 250
  Cache misses: 16 (6.40%)

Benchmark: List Sequential
  Cycles: 4,567
  Cache refs: 1,000
  Cache misses: 950 (95.00%)
```

**分析**：
- Array: 快 3.7 倍
- Array: Cache misses 少 15.8 倍
- List: 95% cache miss rate（幾乎每次存取都 miss）

---

## Summary

測量問題透過建立嚴謹的 benchmarking framework 解決了。那個「感覺更快」的優化結果慢了 15%——直覺失敗了，但資料沒有。Framework 透過 high-precision timing、統計分析和 cache profiling 揭露了真相。

High-precision timing 是基礎。`time()` 的 1 秒解析度對快速操作來說沒用。`clock_gettime()` 提供 nanosecond 解析度和 monotonic clock，但最好的方法是直接使用 CPU cycle counters（RISC-V 的 `rdcycle`、x86 的 `rdtsc`、ARM 的 `pmccntr_el0`），提供 1-cycle 精度，沒有 system call overhead。但單次測量是沒意義的——cache 狀態、OS interrupts、branch prediction 和 memory allocator 行為都會變化。你需要多次執行並報告統計數據：minimum（best-case，warm cache）、median（典型性能，對 outliers 穩健）、standard deviation（變異性）和 maximum（worst-case，對 real-time 系統很重要）。

Benchmark framework 設計讓測量可重複使用。Framework 定義了 `benchmark_t` 結構，包含 `setup`、`run` 和 `teardown` 函數，加上 `benchmark_run()` 函數來執行測量。Framework 做 warmup run（避免 cold cache 偏差）、多次 iterations、計算統計數據，並報告結果。這讓比較不同實作變得簡單——只要定義 benchmark 結構並執行它。

Cache 分析解釋了為什麼。Timing 告訴你花了多久，但 cache profiling 告訴你為什麼。Linux `perf` 提供 hardware performance counters：`cache-references`、`cache-misses`、`L1-dcache-loads`、`L1-dcache-load-misses`、`LLC-loads`、`LLC-load-misses`。比較 array 和 linked list 的 sequential access 顯示 array 有 6.4% cache miss rate，而 linked list 有 95% cache miss rate——這解釋了 3.7 倍的性能差距。我們可以把 perf counters 整合到 benchmark framework，用 `perf_event_open()` 和 `read()` 來讀取 counters。

常見陷阱會破壞測量。Compiler optimizations 可能會把你的 benchmark 優化掉——使用 `volatile` 或 return value 來防止。Cold vs warm cache 會讓第一次執行比後續執行慢 10 倍——永遠做 warmup run。Measurement overhead（`rdcycle()` 本身要花 ~10 cycles）會影響短測試——測量 overhead 並減去它，或確保測試夠長。System noise（OS interrupts、其他 processes、frequency scaling）會增加變異性——執行很多 iterations、報告 median、停用 frequency scaling、pin 到 CPU core、提高 priority。

嵌入式系統有獨特的挑戰。很多嵌入式系統沒有 `perf`——直接透過 memory-mapped registers 或 CSRs 使用 hardware performance counters（RISC-V 的 `mcounteren` 和 `mcountinhibit`）。Bare-metal 系統沒有 `clock_gettime()`——使用 hardware timers 或 cycle counters。Real-time 系統在意 worst-case 性能——報告 maximum 和 99th percentile，不只是 mean。有限的記憶體無法儲存數千次測量——使用 online algorithms（Welford's algorithm for running mean and variance）。

實際範例證明了這些技術。比較 array 和 linked list 的 sequential access（1000 個元素，1000 次 iterations）顯示 array 快 3.7 倍（1,234 vs 4,567 cycles）、cache misses 少 15.8 倍（16 vs 950 misses）、cache miss rate 低很多（6.4% vs 95%）。Linked list 幾乎每次存取都 miss cache，因為 pointer chasing 和差的 spatial locality。Array 受益於 cache lines（一次抓取 64 bytes）和 prefetcher（偵測 sequential access）。

教訓很明確：永遠不要相信你的直覺。永遠要測量。使用 high-precision timing、統計分析和 cache profiling 來理解性能。建立可重複使用的 benchmark framework 來讓測量變得簡單和一致。避免常見陷阱（compiler optimizations、cold cache、measurement overhead、system noise）。在嵌入式系統上，使用 hardware counters、worst-case 分析和 online statistics。在下一章，我們會用我們的 benchmarking framework 深入探討 arrays，並學習如何最大化 cache locality 和性能。

