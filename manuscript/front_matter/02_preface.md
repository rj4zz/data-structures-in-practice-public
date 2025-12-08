# Preface

## Why This Book Exists

I've spent over 20 years writing system software—bootloaders, device drivers, firmware, and embedded systems. During that time, I've learned that the data structures taught in textbooks often fail to deliver expected performance when running on real hardware.

The problem isn't that the textbooks are wrong. Big-O complexity analysis is correct and important. The problem is that it's incomplete. Modern computers have complex memory hierarchies where a single cache miss can cost as much as 100 register operations. In this environment, an O(log n) algorithm with good cache behavior can easily outperform an O(1) algorithm with poor cache behavior.

This book bridges that gap. It teaches data structures from a hardware-aware perspective, showing you how to design and implement data structures that perform well on real silicon, not just in theoretical analysis.

## Who This Book Is For

This book is written for:

- **System software engineers** who need to understand how data structures interact with hardware
- **Embedded systems developers** working with constrained resources and real-time requirements
- **Performance-conscious programmers** who want to understand why their code is slow

You should be comfortable with:
- C programming (pointers, structs, memory management)
- Basic data structures (arrays, linked lists, trees)
- Basic algorithms (sorting, searching)
- Command-line tools and compilation

You don't need:
- Advanced algorithms knowledge
- Computer architecture expertise (we'll teach what you need)
- Assembly language (we'll introduce it when necessary)

## About the Stories

This book uses narrative-driven examples to make technical concepts concrete and memorable. Each chapter opens with a story that illustrates a real-world problem, then investigates the solution using actual measurements and profiling data.

The scenarios in this book fall into two categories:

**Real Cases**: Many stories are based on actual work experience in embedded systems and system software development. Technical details—performance numbers, cache behavior, hardware constraints—are authentic. Some scenarios have been generalized to protect proprietary information, but the technical substance remains accurate.

**Mock Scenarios**: Some examples are constructed specifically to illustrate technical points. While not from actual projects, these scenarios are grounded in realistic engineering situations and plausible technical constraints. They represent problems that commonly occur in embedded systems and system software development.

**Important**: Whether real or mock, all scenarios avoid fabricated specifics like locations, customer names, or overly dramatic timelines. The focus is always on technical truth and realistic engineering contexts.

**All benchmark results, performance measurements, and hardware behavior are based on actual testing or documented specifications.** When you see numbers in this book, they come from real measurements on real hardware.

## How to Read This Book

**Sequential reading**: The book is designed to be read front-to-back. Early chapters establish foundations (memory hierarchy, benchmarking) that later chapters build upon.

**Reference reading**: Each chapter is also self-contained enough to serve as a reference. If you need to understand hash tables or lock-free queues, you can jump directly to that chapter.

**Code examples**: All code examples are available in the book's repository. They're designed to be compiled and run on standard Linux systems. Many examples also work on embedded systems with minimal modification.

**Benchmarks**: The book includes a complete benchmarking framework. You can reproduce all measurements and experiment with variations.

## What You'll Learn

By the end of this book, you'll understand:

- How memory hierarchy affects data structure performance
- When to use arrays vs. linked lists (hint: almost always arrays)
- How to design cache-friendly data structures
- Why hash tables are often slower than binary search
- How to implement lock-free data structures correctly
- How to measure and profile data structure performance
- How to choose the right data structure for embedded systems

More importantly, you'll learn to **measure, don't assume**. Every optimization claim in this book is backed by actual benchmark results.

## Acknowledgments

This book would not exist without the inspiration and support of many people.

First and foremost, I want to thank **Professor Bing-Hong Liu** for the insightful discussions that sparked the idea for this book. Our conversations about the gap between textbook data structures and real-world performance planted the seed that grew into this project. His encouragement to bridge theory and practice has been invaluable.

I'm grateful to the **open-source community** for creating the tools that made this book possible—perf, Valgrind, GCC, LLVM, and countless others. The transparency of open-source software allows us to understand performance at the deepest levels.

Thank you to the engineers who have shared their knowledge through blogs, papers, and conference talks. The work of **Brendan Gregg** on performance analysis, **Fedor Pikus** on C++ optimization, **Ulrich Drepper** on memory systems, and many others has shaped my understanding and influenced this book.

I'm indebted to my colleagues at **SiFive**, **MIPS**, **Andes Technology**, **Broadcom**, **Western Digital**, and **SiS** for the real-world experiences that inform the examples in this book. The problems we solved together—from bootloader optimization to firmware debugging—are the foundation of the practical insights here.

Thank you to the **early reviewers** who provided feedback on draft chapters. Your suggestions improved both the technical accuracy and clarity of the material.

Finally, thank you to my **family** for their patience and support during the many evenings and weekends spent writing. This book is as much yours as it is mine.

## About the Author

Danny Jiang has over 20 years of experience in system software engineering, specializing in embedded systems, bootloaders, device drivers, and firmware development. He has worked on RISC-V, ARM, and x86 architectures, from tiny microcontrollers to application processors.

---

**Let's begin.**

