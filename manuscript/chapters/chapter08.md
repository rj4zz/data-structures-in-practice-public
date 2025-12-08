# Chapter 8: Dynamic Arrays and Memory Management

**Part II: Basic Data Structures**

---

> "Premature optimization is the root of all evil, but so is premature pessimization."
> — Andrei Alexandrescu

## The Reallocation Problem

Dynamic arrays (vectors in C++, ArrayList in Java) are one of the most useful data structures. They combine the cache-friendliness of arrays with the flexibility of dynamic sizing.

But there's a hidden cost: **reallocation**.

I was working on a log aggregator for an embedded system. The system collected log messages in a dynamic array and periodically flushed them to flash storage. Simple, right?

The performance was terrible. The system was spending 60% of its time in `realloc()`.

The problem? The array was growing one element at a time:

```c
typedef struct {
    char **messages;
    int size;
    int capacity;
} log_buffer_t;

void add_message(log_buffer_t *buf, const char *msg) {
    if (buf->size >= buf->capacity) {
        buf->capacity++;  // Grow by 1!
        buf->messages = realloc(buf->messages, 
                               buf->capacity * sizeof(char*));
    }
    buf->messages[buf->size++] = strdup(msg);
}
```

**For 1000 messages**: 1000 reallocations, each copying the entire array.

**Total copies**: 1 + 2 + 3 + ... + 1000 = 500,500 elements copied!

The fix was simple: **grow exponentially**, not linearly.

```c
void add_message(log_buffer_t *buf, const char *msg) {
    if (buf->size >= buf->capacity) {
        buf->capacity = buf->capacity ? buf->capacity * 2 : 16;
        buf->messages = realloc(buf->messages, 
                               buf->capacity * sizeof(char*));
    }
    buf->messages[buf->size++] = strdup(msg);
}
```

**For 1000 messages**: 7 reallocations (16, 32, 64, 128, 256, 512, 1024).

**Total copies**: ~2000 elements (vs 500,500).

**Result**: 250× fewer copies, 60× faster.

**Visualizing exponential growth**:

```
Initial state:
┌──────────────────┐
│ Size: 0, Cap: 0  │
└──────────────────┘

Add 1st element → Allocate initial capacity
┌──────────────────┐
│ Size: 1, Cap: 16 │ ✅ Allocated
└──────────────────┘

Add elements 2-16 → No reallocation
┌───────────────────┐
│ Size: 16, Cap: 16 │ ⚠️ Full
└───────────────────┘

Add 17th element → Realloc (16 → 32)
┌───────────────────┐
│ Size: 17, Cap: 32 │ ✅ Reallocated (copy 16 elements)
└───────────────────┘

Add elements 18-32 → No reallocation
┌───────────────────┐
│ Size: 32, Cap: 32 │ ⚠️ Full
└───────────────────┘

Add 33rd element → Realloc (32 → 64)
┌───────────────────┐
│ Size: 33, Cap: 64 │ ✅ Reallocated (copy 32 elements)
└───────────────────┘

Continue...
┌────────────────────┐
│ Size: 64, Cap: 64  │ → Realloc to 128
│ Size: 128, Cap: 128│ → Realloc to 256
│ Size: 256, Cap: 256│ → Realloc to 512
│ Size: 512, Cap: 512│ → Realloc to 1024
└────────────────────┘

Final state (1000 elements):
┌──────────────────────┐
│ Size: 1000, Cap: 1024│ ✅ Only ~10 reallocations total
└──────────────────────┘

Total reallocations: 10 (vs 1000 if growing by 1 each time)
Total copies: ~2000 elements (vs 500,500)
```

**Real-world applications of this strategy**:

This "allocate extra space to avoid frequent expensive operations" pattern appears in many systems:

- **String builders** (Java `StringBuilder`, C# `StringBuilder`): Grow exponentially to avoid O(n²) string concatenation
- **Network buffers** (TCP receive buffers): Pre-allocate larger buffers to reduce system calls
- **Memory allocators** (malloc implementations): Use size classes (16, 32, 64, 128...) to reduce fragmentation
- **Database transaction logs**: Pre-allocate log space in chunks to avoid frequent disk I/O
- **Sparse matrices** (scientific computing): Allocate extra capacity in compressed row storage to allow efficient insertions
- **File systems** (ext4, XFS): Pre-allocate blocks for growing files to reduce fragmentation

The key insight: **Trading space for time** by over-allocating reduces the amortized cost of growth from O(n) to O(1).

## Dynamic Array Implementation

Here's a complete dynamic array implementation:

```c
typedef struct {
    int *data;
    size_t size;      // Number of elements
    size_t capacity;  // Allocated space
} vector_t;

void vector_init(vector_t *v) {
    v->data = NULL;
    v->size = 0;
    v->capacity = 0;
}

void vector_free(vector_t *v) {
    free(v->data);
    v->data = NULL;
    v->size = 0;
    v->capacity = 0;
}

void vector_push(vector_t *v, int value) {
    if (v->size >= v->capacity) {
        size_t new_capacity = v->capacity ? v->capacity * 2 : 16;
        int *new_data = realloc(v->data, new_capacity * sizeof(int));
        if (!new_data) {
            // Handle allocation failure
            return;
        }
        v->data = new_data;
        v->capacity = new_capacity;
    }
    v->data[v->size++] = value;
}

int vector_pop(vector_t *v) {
    if (v->size > 0) {
        return v->data[--v->size];
    }
    return -1;  // Error
}

int vector_get(vector_t *v, size_t index) {
    if (index < v->size) {
        return v->data[index];
    }
    return -1;  // Error
}
```

**Key design choices**:
- Initial capacity: 16 (avoid tiny allocations)
- Growth factor: 2× (exponential growth)
- `realloc()`: May avoid copy if space available

## Growth Factor Analysis

The growth factor affects both memory usage and performance.

**Common growth factors**:
- **1.5×**: Used by some implementations (e.g., Facebook's folly)
- **2×**: Most common (C++ std::vector, Python list)
- **φ (1.618)**: Golden ratio, theoretical optimum

**Trade-offs**:

**2× growth**:
- Pros: Simple, fast (bit shift), good amortized performance
- Cons: Can waste up to 50% memory

**1.5× growth**:
- Pros: Less memory waste (~33%), better memory reuse
- Cons: More reallocations, slightly slower

**Benchmark** (growing to 1M elements):

```
Growth factor 1.5×:
  Reallocations: 34
  Peak memory: 1.5 MB
  Time: 12 ms

Growth factor 2×:
  Reallocations: 20
  Peak memory: 2 MB
  Time: 8 ms
```

**2× is faster** (fewer reallocations) but uses more memory.

**Recommendation**: Use 2× unless memory is very tight.

## Shrinking: When to Deallocate

Should you shrink the array when elements are removed?

**Naive approach**: Shrink on every pop

```c
void vector_pop(vector_t *v) {
    if (v->size > 0) {
        v->size--;
        if (v->size < v->capacity / 2) {
            v->capacity /= 2;
            v->data = realloc(v->data, v->capacity * sizeof(int));
        }
    }
}
```

**Problem**: Thrashing if you push/pop around the threshold

```
Push to 1024 → capacity 1024
Pop to 512   → shrink to 512
Push to 513  → grow to 1024
Pop to 512   → shrink to 512
...
```

**Better approach**: Hysteresis (shrink at 1/4, not 1/2)

```c
void vector_pop(vector_t *v) {
    if (v->size > 0) {
        v->size--;
        if (v->size < v->capacity / 4 && v->capacity > 16) {
            v->capacity /= 2;
            v->data = realloc(v->data, v->capacity * sizeof(int));
        }
    }
}
```

**Now**: Must pop to 256 before shrinking from 1024.

**Even better**: Don't shrink automatically, provide explicit `vector_shrink_to_fit()`.

```c
void vector_shrink_to_fit(vector_t *v) {
    if (v->size < v->capacity) {
        v->data = realloc(v->data, v->size * sizeof(int));
        v->capacity = v->size;
    }
}
```

**Recommendation**: Don't auto-shrink unless memory is critical.

## Reserve and Capacity

If you know the final size in advance, **reserve** space upfront:

```c
void vector_reserve(vector_t *v, size_t capacity) {
    if (capacity > v->capacity) {
        int *new_data = realloc(v->data, capacity * sizeof(int));
        if (new_data) {
            v->data = new_data;
            v->capacity = capacity;
        }
    }
}

// Usage
vector_t v;
vector_init(&v);
vector_reserve(&v, 1000);  // Allocate once

for (int i = 0; i < 1000; i++) {
    vector_push(&v, i);  // No reallocation!
}
```

**Benchmark** (1000 elements):

```
Without reserve:
  Reallocations: 7
  Time: 45 μs

With reserve:
  Reallocations: 1
  Time: 12 μs
```

**3.75× faster** by avoiding reallocations.

**Guideline**: If you know the size, always reserve.

## Small Vector Optimization (SVO)

For small vectors, the overhead of heap allocation dominates.

**Small Vector Optimization**: Store small arrays inline, only allocate for large arrays.

```c
#define SMALL_SIZE 16

typedef struct {
    int small_data[SMALL_SIZE];  // Inline storage
    int *data;                    // Heap storage (if needed)
    size_t size;
    size_t capacity;
} small_vector_t;

void small_vector_init(small_vector_t *v) {
    v->data = v->small_data;  // Start with inline storage
    v->size = 0;
    v->capacity = SMALL_SIZE;
}

void small_vector_push(small_vector_t *v, int value) {
    if (v->size >= v->capacity) {
        size_t new_capacity = v->capacity * 2;
        int *new_data = malloc(new_capacity * sizeof(int));

        // Copy from inline or heap storage
        memcpy(new_data, v->data, v->size * sizeof(int));

        // Free old heap storage (if any)
        if (v->data != v->small_data) {
            free(v->data);
        }

        v->data = new_data;
        v->capacity = new_capacity;
    }
    v->data[v->size++] = value;
}

void small_vector_free(small_vector_t *v) {
    if (v->data != v->small_data) {
        free(v->data);
    }
}
```

**Benefits**:
- No allocation for small vectors (≤ 16 elements)
- Better cache locality (data inline with struct)
- Faster for common case

**Cost**:
- Larger struct size (64 bytes vs 16 bytes)
- One extra copy when transitioning to heap

**Benchmark** (average size: 8 elements):

```
Regular vector:
  Allocations: 1 per vector
  Time: 850 ns per vector

Small vector:
  Allocations: 0 (inline)
  Time: 120 ns per vector
```

**7× faster** for small vectors.

**Recommendation**: Use SVO for vectors that are usually small (< 16-32 elements).

## Memory Allocator Considerations

`realloc()` performance depends on the allocator.

**Best case**: Allocator can expand in place (no copy)

```c
// Allocate 1 KB
void *ptr = malloc(1024);

// Expand to 2 KB (may expand in place)
ptr = realloc(ptr, 2048);  // No copy if space available
```

**Worst case**: Must allocate new block and copy

```c
// Allocate 1 KB
void *ptr = malloc(1024);

// Another allocation uses adjacent space
void *other = malloc(1024);

// Now realloc must copy
ptr = realloc(ptr, 2048);  // Must allocate new block and copy
```

**Embedded systems**: Often use simple allocators that can't expand in place.

**Solution**: Use custom allocator or memory pool

```c
typedef struct {
    char pool[64 * 1024];  // 64 KB pool
    size_t used;
} memory_pool_t;

memory_pool_t global_pool = {0};

void *pool_alloc(size_t size) {
    if (global_pool.used + size > sizeof(global_pool.pool)) {
        return NULL;  // Out of memory
    }
    void *ptr = &global_pool.pool[global_pool.used];
    global_pool.used += size;
    return ptr;
}

// Can't free individual allocations, but can reset entire pool
void pool_reset(void) {
    global_pool.used = 0;
}
```

**Use case**: Temporary vectors that are freed together.

## Insertion and Deletion

Inserting or deleting in the middle requires shifting elements.

**Insert at index**:

```c
void vector_insert(vector_t *v, size_t index, int value) {
    if (index > v->size) return;

    // Ensure capacity
    if (v->size >= v->capacity) {
        size_t new_capacity = v->capacity ? v->capacity * 2 : 16;
        v->data = realloc(v->data, new_capacity * sizeof(int));
        v->capacity = new_capacity;
    }

    // Shift elements right
    memmove(&v->data[index + 1], &v->data[index],
            (v->size - index) * sizeof(int));

    v->data[index] = value;
    v->size++;
}
```

**Delete at index**:

```c
void vector_delete(vector_t *v, size_t index) {
    if (index >= v->size) return;

    // Shift elements left
    memmove(&v->data[index], &v->data[index + 1],
            (v->size - index - 1) * sizeof(int));

    v->size--;
}
```

**Performance**:
- Insert/delete at end: O(1)
- Insert/delete at beginning: O(n) (must shift all elements)
- Insert/delete in middle: O(n)

**Benchmark** (1000 elements):

```
Insert at end:       50 ns
Insert at beginning: 12,000 ns  (240× slower)
Insert in middle:    6,000 ns   (120× slower)
```

**Guideline**: If you need frequent insertions/deletions in the middle, consider a different data structure (e.g., linked list, gap buffer).

## Embedded Systems: Fixed-Capacity Vectors

On embedded systems, you often can't afford dynamic allocation.

**Fixed-capacity vector**:

```c
#define MAX_CAPACITY 256

typedef struct {
    int data[MAX_CAPACITY];
    size_t size;
} fixed_vector_t;

void fixed_vector_init(fixed_vector_t *v) {
    v->size = 0;
}

int fixed_vector_push(fixed_vector_t *v, int value) {
    if (v->size >= MAX_CAPACITY) {
        return -1;  // Full
    }
    v->data[v->size++] = value;
    return 0;
}
```

**Benefits**:
- No allocation
- Predictable memory usage
- Fast (no reallocation)
- Simple

**Cost**:
- Fixed maximum size
- May waste memory if not full

**Recommendation**: Use fixed-capacity vectors on embedded systems unless you truly need dynamic sizing.

## Real-World Example: Log Buffer Optimization

Back to my log aggregator. Here's the complete optimization:

**Before** (grow by 1):

```c
typedef struct {
    char **messages;
    int size;
    int capacity;
} log_buffer_t;

void add_message(log_buffer_t *buf, const char *msg) {
    if (buf->size >= buf->capacity) {
        buf->capacity++;
        buf->messages = realloc(buf->messages,
                               buf->capacity * sizeof(char*));
    }
    buf->messages[buf->size++] = strdup(msg);
}
```

**Problems**:
- 1000 reallocations for 1000 messages
- 500,500 elements copied
- Terrible performance

**After** (exponential growth + reserve):

```c
typedef struct {
    char **messages;
    int size;
    int capacity;
} log_buffer_t;

void log_buffer_init(log_buffer_t *buf, int expected_size) {
    buf->messages = malloc(expected_size * sizeof(char*));
    buf->size = 0;
    buf->capacity = expected_size;
}

void add_message(log_buffer_t *buf, const char *msg) {
    if (buf->size >= buf->capacity) {
        buf->capacity *= 2;
        buf->messages = realloc(buf->messages,
                               buf->capacity * sizeof(char*));
    }
    buf->messages[buf->size++] = strdup(msg);
}
```

**Improvements**:
- Reserve expected size upfront
- Exponential growth if exceeded
- 7 reallocations (vs 1000)
- 2000 elements copied (vs 500,500)

**Result**: 60× faster, from 60% CPU to < 1% CPU.

## Gap Buffer: Efficient Text Editing

For text editors, you need efficient insertion/deletion at the cursor position.

**Problem with dynamic array**: Inserting at cursor requires shifting all text after cursor.

**Solution**: Gap buffer (used by Emacs)

```c
typedef struct {
    char *buffer;
    size_t gap_start;  // Start of gap
    size_t gap_end;    // End of gap
    size_t capacity;
} gap_buffer_t;

void gap_buffer_init(gap_buffer_t *gb, size_t capacity) {
    gb->buffer = malloc(capacity);
    gb->gap_start = 0;
    gb->gap_end = capacity;
    gb->capacity = capacity;
}

void gap_buffer_insert(gap_buffer_t *gb, char c) {
    if (gb->gap_start >= gb->gap_end) {
        // Grow buffer (double size)
        size_t new_capacity = gb->capacity * 2;
        char *new_buffer = malloc(new_capacity);

        // Copy before gap
        memcpy(new_buffer, gb->buffer, gb->gap_start);

        // Copy after gap
        size_t after_gap = gb->capacity - gb->gap_end;
        memcpy(new_buffer + new_capacity - after_gap,
               gb->buffer + gb->gap_end, after_gap);

        free(gb->buffer);
        gb->buffer = new_buffer;
        gb->gap_end = new_capacity - after_gap;
        gb->capacity = new_capacity;
    }

    gb->buffer[gb->gap_start++] = c;
}

void gap_buffer_move_cursor(gap_buffer_t *gb, int new_pos) {
    if (new_pos < gb->gap_start) {
        // Move gap left
        size_t move = gb->gap_start - new_pos;
        memmove(&gb->buffer[gb->gap_end - move],
                &gb->buffer[new_pos], move);
        gb->gap_start = new_pos;
        gb->gap_end -= move;
    } else if (new_pos > gb->gap_start) {
        // Move gap right
        size_t move = new_pos - gb->gap_start;
        memmove(&gb->buffer[gb->gap_start],
                &gb->buffer[gb->gap_end], move);
        gb->gap_start += move;
        gb->gap_end += move;
    }
}
```

**How it works**:

```
Initial (capacity 10):
[_, _, _, _, _, _, _, _, _, _]
 ^gap_start              ^gap_end

Insert "abc":
[a, b, c, _, _, _, _, _, _, _]
          ^gap_start     ^gap_end

Move cursor to 1:
[a, _, _, _, _, _, _, b, c, _]
    ^gap_start        ^gap_end

Insert "x":
[a, x, _, _, _, _, _, b, c, _]
       ^gap_start     ^gap_end
```

**Benefits**:
- O(1) insertion at cursor (just move gap_start)
- O(1) deletion at cursor
- Only pay for cursor movement (amortized O(1) for sequential editing)

**Benchmark** (1000 insertions at cursor):

```
Dynamic array: 12,000 μs  (shift on every insert)
Gap buffer:       120 μs  (100× faster)
```

## Summary

The reallocation problem was solved by understanding growth strategies. The log aggregator spending 60% of its time in `realloc()` was growing one element at a time, causing 500,500 element copies for just 1000 messages. Switching to exponential growth (2× capacity) reduced reallocations from 1000 to just 10, making the system dramatically faster.

**Key insights**:
- Exponential growth (2×) for amortized O(1) append
- Reserve space if size known
- Don't auto-shrink (use explicit shrink_to_fit)
- Small vector optimization for common case
- Fixed-capacity for embedded systems

**Growth strategies**:
- 2× growth: Fewer reallocations, more memory
- 1.5× growth: More reallocations, less memory
- **Recommendation**: Use 2× unless memory critical

**Optimizations**:
- Reserve: Avoid reallocations if size known
- Small vector optimization: Inline storage for small arrays
- Memory pools: Avoid allocator overhead
- Gap buffer: Efficient text editing

**Embedded considerations**:
- Fixed-capacity vectors (no allocation)
- Predictable memory usage
- Simple allocators can't expand in place
- Consider memory pools

**When to use dynamic arrays**:
- Need variable size
- Mostly append operations
- Random access required
- Cache-friendly sequential access

**When NOT to use**:
- Frequent insertions/deletions in middle → gap buffer or rope
- Fixed size known → static array
- Embedded with tight memory → fixed-capacity vector

**Next steps**: We've covered the fundamental data structures (arrays, lists, stacks, queues, hash tables, dynamic arrays). In Part III, we'll explore trees and hierarchical structures, where cache behavior becomes even more critical.

