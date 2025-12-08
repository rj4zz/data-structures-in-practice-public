# Chapter 3: Benchmarking and Profiling

**Part I: Foundations**

---

> "In God we trust. All others must bring data."
> — W. Edwards Deming

## The Measurement Problem

After learning about memory hierarchy in Chapter 2, you might be eager to optimize your code. But there's a problem: **how do you know if your optimization actually worked?**

I learned this lesson the hard way.

I was optimizing a hash table implementation for a bootloader. Based on my understanding of cache behavior, I rewrote the hash function to be "more cache-friendly." I was confident it would be faster.

I ran the code. It felt faster. I committed the change.

A week later, a colleague ran benchmarks and found that my "optimization" had made the code **15% slower**. I had optimized for the wrong thing, and I had no data to prove my assumptions.

**The lesson**: Never trust your intuition. Always measure.

This chapter is about how to measure correctly. We'll build a comprehensive benchmarking framework and learn to use profiling tools effectively.

## High-Precision Timing

The first challenge: how do you measure time accurately?

**Bad approach**: Using `time()`

```c
time_t start = time(NULL);
run_test();
time_t end = time(NULL);
printf("Time: %ld seconds\n", end - start);
```

**Problem**: 1-second resolution. Useless for fast operations.

**Better approach**: Using `clock_gettime()`

```c
struct timespec start, end;
clock_gettime(CLOCK_MONOTONIC, &start);
run_test();
clock_gettime(CLOCK_MONOTONIC, &end);

long ns = (end.tv_sec - start.tv_sec) * 1000000000L +
          (end.tv_nsec - start.tv_nsec);
printf("Time: %ld ns\n", ns);
```

**Advantages**:
- Nanosecond resolution
- `CLOCK_MONOTONIC`: Not affected by system time changes
- Portable (POSIX)

**Best approach**: Using CPU cycle counters

On RISC-V:
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

On x86_64:
```c
static inline uint64_t rdtsc(void) {
    uint32_t lo, hi;
    asm volatile ("rdtsc" : "=a" (lo), "=d" (hi));
    return ((uint64_t)hi << 32) | lo;
}
```

On ARM64:
```c
static inline uint64_t rdcycle(void) {
    uint64_t val;
    asm volatile("mrs %0, pmccntr_el0" : "=r"(val));
    return val;
}
```

**Advantages**:
- Highest precision (1 cycle)
- Direct hardware measurement
- No system call overhead

**Disadvantages**:
- Architecture-specific
- Can be affected by frequency scaling
- May require kernel configuration (ARM)

## Statistical Analysis

A single measurement is meaningless. You need **multiple runs** and **statistical analysis**.

**Why?**
- Cache state varies between runs
- OS interrupts affect timing
- Branch prediction varies
- Memory allocator behavior varies

**Minimum approach**: Run multiple times and report min/max/mean

```c
#define ITERATIONS 1000

uint64_t times[ITERATIONS];
for (int i = 0; i < ITERATIONS; i++) {
    uint64_t start = rdcycle();
    run_test();
    uint64_t end = rdcycle();
    times[i] = end - start;
}

// Calculate statistics
uint64_t min = times[0], max = times[0], sum = 0;
for (int i = 0; i < ITERATIONS; i++) {
    if (times[i] < min) min = times[i];
    if (times[i] > max) max = times[i];
    sum += times[i];
}
uint64_t mean = sum / ITERATIONS;

printf("Min: %lu cycles\n", min);
printf("Max: %lu cycles\n", max);
printf("Mean: %lu cycles\n", mean);
```

**Better approach**: Add median and standard deviation

```c
// Sort for median
qsort(times, ITERATIONS, sizeof(uint64_t), compare_uint64);
uint64_t median = times[ITERATIONS / 2];

// Calculate standard deviation
double variance = 0;
for (int i = 0; i < ITERATIONS; i++) {
    double diff = (double)times[i] - (double)mean;
    variance += diff * diff;
}
double stddev = sqrt(variance / ITERATIONS);

printf("Median: %lu cycles\n", median);
printf("Std dev: %.2f cycles\n", stddev);
```

**What to report?**
- **Minimum**: Best-case performance (warm cache)
- **Median**: Typical performance (more robust than mean)
- **Standard deviation**: Variability (lower is better)
- **Maximum**: Worst-case (important for real-time systems)

## Benchmark Framework Design

Let's build a reusable framework. Here's the interface:

```c
typedef struct {
    const char *name;
    void (*setup)(void);
    void (*run)(void);
    void (*teardown)(void);
} benchmark_t;

void benchmark_run(benchmark_t *bench, int iterations);
```

**Implementation**:

```c
void benchmark_run(benchmark_t *bench, int iterations) {
    uint64_t *times = malloc(iterations * sizeof(uint64_t));

    printf("Running benchmark: %s\n", bench->name);

    // Warmup run
    if (bench->setup) bench->setup();
    bench->run();
    if (bench->teardown) bench->teardown();

    // Actual measurements
    for (int i = 0; i < iterations; i++) {
        if (bench->setup) bench->setup();

        uint64_t start = rdcycle();
        bench->run();
        uint64_t end = rdcycle();

        if (bench->teardown) bench->teardown();

        times[i] = end - start;
    }

    // Calculate and report statistics
    report_statistics(bench->name, times, iterations);

    free(times);
}
```

**Usage example**:

```c
// Test data
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

## Cache Analysis with perf

Timing tells you **how long**, but not **why**. For that, you need cache analysis.

**Linux perf** is the standard tool for performance analysis:

```bash
# Basic cache statistics
$ perf stat -e cache-references,cache-misses ./program
  Performance counter stats:
    1,234,567 cache-references
       12,345 cache-misses              #    1.00% of all cache refs
```

**Useful events**:
- `cache-references`: Total cache accesses
- `cache-misses`: Cache misses (all levels)
- `L1-dcache-loads`: L1 data cache loads
- `L1-dcache-load-misses`: L1 data cache load misses
- `LLC-loads`: Last-level cache loads
- `LLC-load-misses`: Last-level cache load misses

**Detailed analysis**:

```bash
# L1 cache analysis
$ perf stat -e L1-dcache-loads,L1-dcache-load-misses ./program
  Performance counter stats:
    10,000,000 L1-dcache-loads
       100,000 L1-dcache-load-misses    #    1.00% miss rate

# All cache levels
$ perf stat -e cache-references,cache-misses,\
L1-dcache-loads,L1-dcache-load-misses,\
LLC-loads,LLC-load-misses ./program
```

**Comparing implementations**:

```bash
# Array version
$ perf stat -e cache-misses ./array_test
    1,234 cache-misses

# Linked list version
$ perf stat -e cache-misses ./list_test
   45,678 cache-misses              # 37× more misses!
```

## Integrating perf with Benchmarks

We can integrate perf measurements into our benchmark framework:

```c
typedef struct {
    uint64_t cycles;
    uint64_t cache_references;
    uint64_t cache_misses;
    uint64_t l1_loads;
    uint64_t l1_misses;
} perf_counters_t;

void benchmark_run_with_perf(benchmark_t *bench, int iterations) {
    // Setup perf counters
    int fd_cache_ref = perf_event_open(PERF_COUNT_HW_CACHE_REFERENCES);
    int fd_cache_miss = perf_event_open(PERF_COUNT_HW_CACHE_MISSES);

    // Run benchmark
    perf_counters_t counters = {0};

    for (int i = 0; i < iterations; i++) {
        if (bench->setup) bench->setup();

        // Read start counters
        uint64_t start_ref = read_counter(fd_cache_ref);
        uint64_t start_miss = read_counter(fd_cache_miss);
        uint64_t start_cycles = rdcycle();

        bench->run();

        // Read end counters
        uint64_t end_cycles = rdcycle();
        uint64_t end_miss = read_counter(fd_cache_miss);
        uint64_t end_ref = read_counter(fd_cache_ref);

        if (bench->teardown) bench->teardown();

        counters.cycles += end_cycles - start_cycles;
        counters.cache_references += end_ref - start_ref;
        counters.cache_misses += end_miss - start_miss;
    }

    // Report results
    printf("Benchmark: %s\n", bench->name);
    printf("  Cycles: %lu\n", counters.cycles / iterations);
    printf("  Cache refs: %lu\n", counters.cache_references / iterations);
    printf("  Cache misses: %lu (%.2f%%)\n",
           counters.cache_misses / iterations,
           100.0 * counters.cache_misses / counters.cache_references);
}
```

## Common Pitfalls

**1. Compiler optimizations**

The compiler might optimize away your benchmark:

```c
// BAD: Compiler optimizes this away
void test(void) {
    int sum = 0;
    for (int i = 0; i < 1000; i++) {
        sum += array[i];
    }
    // sum is never used, entire loop removed!
}

// GOOD: Use volatile or return value
void test(void) {
    volatile int sum = 0;  // Prevents optimization
    for (int i = 0; i < 1000; i++) {
        sum += array[i];
    }
}
```

**2. Cold vs warm cache**

First run is always slower (cold cache):

```c
// First run: cold cache
run_test();  // 10,000 cycles

// Second run: warm cache
run_test();  // 1,000 cycles
```

**Solution**: Always do a warmup run, or report both cold and warm performance.

**3. Measurement overhead**

Timing code itself takes time:

```c
uint64_t start = rdcycle();  // ~10 cycles
uint64_t end = rdcycle();    // ~10 cycles
printf("Overhead: %lu\n", end - start);  // ~20 cycles
```

**Solution**: Measure overhead and subtract it, or ensure test runs long enough that overhead is negligible.

**4. System noise**

OS interrupts, other processes, frequency scaling all add noise.

**Solutions**:
- Run many iterations
- Report median (robust to outliers)
- Disable frequency scaling: `cpupower frequency-set -g performance`
- Pin to CPU core: `taskset -c 0 ./program`
- Increase priority: `nice -n -20 ./program`

## Embedded Systems Considerations

Benchmarking on embedded systems has unique challenges:

**1. Limited profiling tools**

Many embedded systems don't have `perf` or similar tools.

**Solution**: Use hardware performance counters directly via memory-mapped registers.

**Example** (RISC-V):
```c
// Enable performance counters (machine mode)
#define CSR_MCOUNTEREN 0x306
#define CSR_MCOUNTINHIBIT 0x320

void enable_perf_counters(void) {
    // Allow user mode to read counters
    asm volatile ("csrw %0, %1" :: "i"(CSR_MCOUNTEREN), "r"(0x7));
    // Enable all counters
    asm volatile ("csrw %0, %1" :: "i"(CSR_MCOUNTINHIBIT), "r"(0x0));
}
```

**2. No operating system**

Bare-metal systems have no `clock_gettime()`.

**Solution**: Use hardware timers or cycle counters.

```c
// Use SoC timer (example)
#define TIMER_BASE 0x10000000
#define TIMER_MTIME (*(volatile uint64_t*)(TIMER_BASE + 0x00))

uint64_t get_time_us(void) {
    return TIMER_MTIME;  // Assuming 1 MHz timer
}
```

**3. Real-time constraints**

In real-time systems, **worst-case** matters more than average.

**Solution**: Report maximum time and 99th percentile.

```c
// Sort times
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

**4. Limited memory**

Can't store thousands of measurements.

**Solution**: Use online algorithms (running statistics).

```c
typedef struct {
    uint64_t count;
    uint64_t min;
    uint64_t max;
    double mean;
    double m2;  // For variance calculation
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

## Practical Example: Array vs Linked List

Let's put it all together with a complete benchmark comparing arrays and linked lists:

```c
#define SIZE 1000
#define ITERATIONS 1000

// Array implementation
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

// Linked list implementation
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

// Run benchmarks
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

**Expected output**:

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

**Analysis**:
- Array: 3.7× faster
- Array: 15.8× fewer cache misses
- List: 95% cache miss rate (almost every access misses)

## Summary

The measurement problem was solved by building a rigorous benchmarking framework. The "optimization" that felt faster turned out to be 15% slower—intuition failed, but data didn't. The framework revealed the truth through high-precision timing, statistical analysis, and cache profiling.

**Measurement techniques**:
- High-precision timing (`clock_gettime()`, cycle counters)
- Statistical analysis (min, median, stddev)
- Cache analysis (`perf`, hardware counters)

**Framework design**:
- Reusable benchmark structure
- Setup/run/teardown phases
- Warmup runs
- Multiple iterations

**Common pitfalls**:
- Compiler optimizations (use `volatile`)
- Cold vs warm cache (warmup runs)
- Measurement overhead (subtract or minimize)
- System noise (many iterations, report median)

**Embedded considerations**:
- Direct hardware counter access
- Worst-case analysis (max, P99)
- Online statistics (limited memory)
- Bare-metal timing

**Next Chapter**: Armed with our benchmarking framework, we'll dive deep into arrays and explore how to maximize cache locality and performance.

