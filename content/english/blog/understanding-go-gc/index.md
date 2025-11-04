---
title: "Understanding Go\'s Garbage Collector"
date: 2025-09-28
---

![gopher-gc-thumbnail.png](gopher-gc-thumbnail.png#halfsize)

I was reading [Efficient Go](https://www.oreilly.com/library/view/efficient-go/9781098105709/) and came across the section on garbage collection (GC). I realized how little I actually knew about such an important topic. Both out of curiosity and for the fun of learning things, I decided to learn a bit more about how it works. So, I looked into many different resources [^resources] and wrote down my understanding to make it stick. This post is the result, and I hope it proves itself to be useful for others as well.

[^resources]: In a way, this post is the result of my overall understanding and summary of the things I learned from resources such as some of the official Go blog posts (like, [A Guide to the Go Garbage Collector](https://go.dev/doc/gc-guide), [Go GC: Prioritizing low latency and simplicity](https://go.dev/blog/go15gc), the ISMM keynote [Getting to Go: The Journey of Go's Garbage Collector](https://go.dev/blog/ismmkeynote) ), the [Memory Efficiency: Mastering Go's Garbage Collector](https://goperf.dev/01-common-patterns/gc/) from the [Go Optimization Guide](https://goperf.dev/), and of course the [Eficient Go](https://www.oreilly.com/library/view/efficient-go/9781098105709/) book. Oh, and also some  Wikipedia entries as well.

# Introduction

Go is a garbage-collected language. This is great for developer velocity. It allows us to spend less time on manual memory management and more on business logic. Unfortunately though, making GC work efficiently is neither simple nor cheap.

The thing is, just because GC hides memory management details from us doesn't mean they don't happen under the hood. They happen, and they are costly. If we don't think about it, we might just generate garbage without realizing the possible runtime costs.

> Think of a garbage collector like a Roomba: Just because you have one does not mean you tell your children not to drop arbitrary pieces of garbage onto the floor.
>
> \- Halvar Flake

So, **By seeing the actual costs involved with GC, we can better appreciate the complexity of the problem it's solving. Thus, we might feel more motivated to write code that creates less garbage.** At least, that was my experience.

For this, in the following sections, I will examine the Go Garbage Collector in more detail. I will examine its trigger policy, how it frees or rearranges memory, and the side effects of these actions. I will also talk about what we can do to help the GC, so that our applications suffer less from latency caused by poorly managed memory.

# The Pacing Problem

Now, even if we had a procedure for cleaning up garbages, unless we trigger it, we are basically are no better of. So, any garbage collector, in a way, requires a mechanism for determining when to trigger the collection process.

Honestly, if I were to implement such a trigger mechanism for the first time, the first idea that would come to my mind would be to simply trigger the GC periodically at a fixed interval. But, even a little bit of thought shows the problems here. A fixed interval does not care whether the program is allocating a lot or very little. What about something like triggering the GC after allocating memory a determined number of times? Likewise, this approach would also be likely to fail. Because it simply ignores the fact that allocations can have very different sizes and lifetimes.

So, **whatever mechanism we come up with, it better should adapt to the program’s behavior.** It needs to monitor how fast memory is being allocated and how quickly old objects become unreachable, then decide when to run the collector accordingly.

> [!TIP] The Pacing Problem
> Running the GC too often or too rarely can both cause serious problems:
>
> **If the GC doesn't run often enough**, memory usage grows too quickly. When it finally runs, it has to clean a much larger area, which takes more CPU time than if collections had been more frequent.
>
> **If the GC runs too often**, it wastes precious CPU cycles by checking for garbage before enough has accumulated. This means extra work with little benefit.
>

It's a bit like cleaning your house: **If you never clean and let garbage pile up, you waste space and also make the next cleaning much harder. If you keep checking a clean room again and again, you're just wasting time.**

Fortunately, Go takes a smarter approach by using a special mechanism to decide when to trigger garbage collection. This mechanism is called the pacer.

# The GC Pacer

> [!NOTE] Go's GC Pacer Source Code
> Those who are curious about how the pacer works in more detail, can read the [runtime/mgcpacer](https://go.dev/src/runtime/mgcpacer.go). It is not that hard to follow and only about 1500 lines of code, comments included.

The main idea behind Go's pacer is to keep garbage collection proportional to the rate of memory allocation. Basically, after each collection, the GC measures the size of the live heap (the memory still in use after collection) and some additional parameters to compute the next target.

Here is a piece of code from [runtime/mgcpacer](https://go.dev/src/runtime/mgcpacer.go#L1233) demonstrating how the heap goal is calculated.

```golang
// Compute the next GC goal, which is when the allocated heap
// has grown by GOGC/100 over where it started the last cycle,
// plus additional runway for non-heap sources of GC work.
gcPercentHeapGoal := ^uint64(0)
if gcPercent := c.gcPercent.Load(); gcPercent >= 0 {
   gcPercentHeapGoal = c.heapMarked + (c.heapMarked+c.lastStackScan.Load()+c.globalsScan.Load())*uint64(gcPercent)/100
}
// Apply the minimum heap size here. It's defined in terms of gcPercent
// and is only updated by functions that call commit.
if gcPercentHeapGoal < c.heapMinimum {
   gcPercentHeapGoal = c.heapMinimum
}
c.gcPercentHeapGoal.Store(gcPercentHeapGoal)
```

Here, `heapMarked` is the number of bytes marked by the previous GC. It is the part of the heap that survived the last collection, and also known as the "live heap". The `lastStackScan` is the number of bytes of stack that were scanned last GC cyclea nd `globalsScan` is the total amount of global variable space that is scannable. There is also one last value to talk about `gcPercent`. It is the growth percentage. It comes from `GOGC` and defaults to `100`, which means the next goal allows roughly a 100 percent growth over the base term.

> [!NOTE] Target Heap Memory
> [A Guide to the Go Garbage Collector](https://go.dev/doc/gc-guide#GOGC), summarizes this calculation of the target heap memory as:
> ```
> Target heap memory = Live heap + (Live heap + GC roots) * GOGC / 100 
> ```
> Here, I find the adding of `GC roots` rather interesting. Why add them into the equation in the first place? Why not use something as simple as the following example?
> 
>```
>Target heap memory = (1 + GOGC/100) * Live Heap
>```
> It turns out that it was already like this until Go v1.18. It seems like the main motivation for this change was to make the GC's pacing model reflect all of the work the collector needs to perform, not just the size of the heap. Those who are further interested in the topic, can look up the [GC Pacer Redesign](https://github.com/golang/proposal/blob/master/design/44167-gc-pacer-redesign.md) proposal that initiated the change.

Regardless of these details, the main idea is still the same: we have a pacer that basically keeps the garbage collector in sync with the program’s allocation behavior. It constantly adjusts when the next collection should happen based on how the heap grows and how much work the previous GC cycle required.

> [!NOTE] The `GOMEMLIMIT` Option
> Go also provides an called `GOMEMLIMIT`. When the process approaches this limit, the pacer logic triggers the GC immediately, without prior checks. It serves as another pacing mechanism, but one focused on memory pressure rather than allocation rate.
>
> However, this option is a tricky one. _Efficient Go_ book especially warns us about it:
> > **When your program allocates and uses more memory than the desired limit with the `GOMEMLIMIT` option set, it will only make things worse.** This is because the GC will run nearly continuously, **consuming around 25% of the CPU time** that could otherwise be used by your program.


At this point, we have discussed how the pacer calculates the next target for heap memory after a collection. But this still leaves the question: **where do we actually compare the current heap memory with the target heap memory to trigger the GC?**

As I understand, there are two main places:

1. **The [runtime/malloc.go](https://go.dev/src/runtime/malloc.go) implementation**: For each large allocation (roughly greater than 32 KB), the GC always checks whether the heap has passed the GC trigger threshold. For small allocations, it performs the check only after enough space has been allocated over time, similar to waiting for sufficient allocation activity to occur. Those interested in the details can examine `mallocgc` and, in particular, its helper functions: `mallocgcTiny`, `mallocgcSmallNoscan`, `mallocgcSmallScanNoHeader`, `mallocgcSmallScanHeader`, and `mallocgcLarge`.

2. **The `forcegchelper` goroutine defined and being run in [runtime/proc.go](https://go.dev/src/runtime/proc.go#L359)**. It basically runs GC if it does not run for a certain period of time defined by the constant `forcegcperiod`.

In addition to these, we can also trigger the GC manually via `runtime.GC()` and `debug.FreeOSMemory()`. I guess they can be helpful in cases where we want to reclaim memory deterministically or prepare for a memory-heavy phase. But in many cases we don't need this, or simply using better patterns is a better idea.

[^gogc-example]: Let's try to understand how `GOGC` is being used to set a new heap target. When the program starts there is no last time for the garbage being collected. So, as a default value, Go GC uses 4MiB. After this, once the live heap crosses 4MiB, first GC gets triggered. For example, if the live heap is 6 MiB after the first collection and `GOGC=100`, the next trigger will be at 12 MiB. If, after that collection, the heap shrinks to 5 MiB, then the following trigger will be set at 10 MiB. This process repeats after each collection cycle.

[^minimal-total-heap-size]: [GOGC](https://go.dev/doc/gc-guide#GOGC) section of the  "A Guide to the Go Garbage Collector" documentation notes that "the Go GC has a minimum total heap size of 4 MiB, so if the GOGC-set target is ever below that, it gets rounded up."

# How The GC Works

As I mentioned in the previous section, Go's garbage collector doesn't give us many things to tweak. Just `GOGC` and `GOMEMLIMIT`. So it's very tempting to ask: why even bother understanding how it works if there's so little we can actually change?

One reason is simple curiosity. But there's also a more practical one. **By understanding how the GC works, we can better appreciate the costs involved. Thus, be more motivated to avoiding patterns that create unnecessary garbage.**

The Efficient Go books summarizes how the Go's GC works as follows:

> The Go GC implementation can be described as the [concurrent, nongenerational, tri‐color mark and sweep collector](https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html) implementation. Whether invoked by the programmer or by the runtime-based `GOGC` or `GOMEMLIMIT` option, the `runtime.GC()` implementation comprises a few phases. The first one is a mark phase that has to:
>
> 1. Perform a “stop the world” (STW) event to inject an essential write barrier (a lock on writing data) into all goroutines. Even though STW is relatively fast (10– 30 microseconds on average), it is pretty impactful—it suspends the execution of all goroutines in our process for that time.
> 2. Try to use 25% of the CPU capacity given to the process to concurrently mark all objects in the heap that are still in use.
> 3. Terminate marking by removing the write barrier from the goroutines. This requires another STW event. After the mark phase, the GC function is generally complete. As interesting as it sounds, the GC doesn't release any memory! Instead, the sweeping phase releases objects that were not marked as in use. It is done lazily: every time a goroutine wants to allocate memory through the Go Allocator, it must perform a sweeping work first, then allocate. This is counted as an allocation latency, even though it is technically a garbage collection functionality—worth noting!

So, I know this is a lot to take in if you're learning about garbage collectors for the first time. What does **“concurrent, nongenerational, tri-color mark-and-sweep collector”** even mean? Let's explain these terms one by one. I'll start with the **tri-color mark-and-sweep** part, it's the core idea behind how the GC identifies and frees unused memory. After that, I will move on to explaining the **concurrent** and **nongenerational** parts.

## Tri-color Mark and Sweep

The **tri-color mark-and-sweep** algorithm is the main technique Go's garbage collector uses to find which parts of memory are still in use. It's called tri-color because it marks objects with three different colors depending on their state:
1. White for objects that haven't been seen yet,
2. Gray for objects that have been discovered but not fully checked,
3. and Black for objects that are verified as reachable.

The main idea is simple. Start from the root objects, follow all the references they point to, and repeat the process for each newly discovered object. In the end, anything that can't be reached from the roots is considered garbage and can be safely freed. This approach is known as tracing garbage collection because it traces all reachable objects starting from the root set.

Forr anyone interested in exploring different garbage collection algorithms visually, I found [Visualizing Garbage Collection Algorithms](https://spin.atomicobject.com/visualizing-garbage-collection-algorithms/) to be a really interesting resource. :)

For example, here's what the mark-and-sweep process looks like from the heap's point of view:

![](https://raw.githubusercontent.com/kenfox/gc-viz/master/docs/MARK_SWEEP_GC.gif#center)

## Concurrent

**Concurrent** means that the GC runs alongside our goroutines most of the time rather than stopping everything. As it's explained in the [Memory Efficiency and Go's Garbage Collector](https://goperf.dev/01-common-patterns/gc/#concurrent) (and visualized below, using the explanation provided there), there are only two parts where "stop the world" occurs:

1. When creating write barriers before the marking phase begins, and
2. When removing those barriers after marking is complete.

![](concurrent.png#75persize)

Here, I think it's important to realize that Go's GC does not stop goroutines to perform the actual marking of unused pointers. It only pauses them briefly to install the write barriers. Another point worth noting is that these write barriers do not block. A barrier might sound like something that halts execution (which was my first impression too), like a mutex, but it's not that. It doesn't stop anything. It just acts like a small guard that notifies the GC whenever a pointer write happens.

> Write barriers ensure correctness while the application mutates objects during concurrent marking. These barriers help track references created or modified mid-scan so the GC doesn't miss them.
>
> [Memory Efficiency: Mastering Go's Garbage Collector](https://goperf.dev/01-common-patterns/gc/)

So, Go's GC doesn't block other goroutines most of the time, except for those tiny moments when it adds or removes the write barriers. I found this trick of introducing barriers pretty clever.

## Nongenerational

**Nongenerational** just means Go is not generational. And what does generational mean? Well, it's basically to treat objects differently depending on how long they've been around and how often they're accessed. Why? Because they can use this information to optimize collection cycles. Most objects die young. So, by focusing on recently created ones more often, the GC can reclaim memory faster without scanning the entire heap each time. This is interesting. But if this approach is so efficient, why doesn't Go do the same? Apparently, the reason is that there was no obvious benefit to it, that is, the benefits were not sufficient enough. [^why-nongenerational]

[^why-nongenerational]: As for why Go is non-generational, [Go Optimization Guide](https://goperf.dev/01-common-patterns/gc/#non-generational) notes that "it hasn't shown clear, consistent benefits in real-world Go programs with the designs tried so far." The ISMM keynote, [Getting to Go: The Journey of Go's Garbage Collector](https://go.dev/blog/ismmkeynote) also explains that while generational collectors can help reduce long stop-the-world pauses, Go's concurrent GC already avoids those and instead focuses on maintaining low, predictable latency.

## Go's GC Philosophy

The more I read and question how Go's garbage collector works, the more I realize how complex the topic actually is. I could certainly go deeper into the technical details. But my goal here is not mastering every internal detail regarding the Go's GC. It's to build a practical understanding and develop an intuitive sense of the bigger picture.

At this point, I think it's sufficient to recognize that the Go team [prioritized low latency and simplicity](https://go.dev/blog/go15gc). These priorities are explicit in Go's own materials and design history. In most cases where they could have traded latency for something else (like raw throughput), they chose to keep latency minimal. Keeping that in mind helps explain many design choices and the philosophy.

# Helping Out The GC

So, now that we have a brief understanding of Go's garbage collector I think it's now a good point for us to talk more about what we can do to help the GC out.

As, _Efficient Go_ points out:

> Produce less garbage! \
> It's easy to overallocate memory in Go. This is why the best way to solve GC bottleneck or other memory efficiency issues is to allocate less.

## Fine Tuning GC Options

{...}

> [!WARNING]
> To me, both `GOGC` and `GOMEMLIMIT` feel like parameters you should tweak only after everything else in your program is in good shape. Probably, code inefficiencies hurt performance more than any gains we'd get from tweaking these settings.
> It's better to treat these settings as fine-tuning tools, not as something your app depends on to perform well.

## 

## Allocate in Stack

In general, stack allocations are faster than heap allocations. This is because the way they work is very simple. When a function starts, it just moves the stack pointer to reserve space. When it ends, it moves the pointer back. No extra bookkeeping is needed. This also means the GC is not involved.

So, we can help the GC by avoiding heap allocations when it is possible to keep them in stack.

Now, when building a binary, the Go compiler decides which values should stay on the stack and which ones must escape to the heap. If the compiler cannot prove that a value can remain on the stack, it makes that value escape to the heap. This process is called [escape analysis](https://en.wikipedia.org/wiki/Escape_analysis).

It is easy to access the debug information for escape analysis.

> As for how to access the information from the Go compiler's escape analysis, the simplest way is through a debug flag supported by the Go compiler that describes all optimizations it applied or did not apply to some package in a text format. This includes whether or not values escape. Try the following command, where [package] is some Go package path.
>
> `$ go build -gcflags=-m=3 [package]`
>
> This information can also be visualized as an overlay in an LSP-capable editor; it is exposed as a code action.
>
> \- [Escape Analysis](https://go.dev/doc/gc-guide#Escape_analysis) section from [A Guide to the Go Garbage Collector](https://go.dev/doc/gc-guide)

After identifying what escapes to the heap and what doesn't, we can start exploring alternative implementations. We can try to figure out if it's possible to achieve the same result without causing heap allocations.

## Memory Pooling

So, if your application creates lots of short-lived objects of the same type, it means you're constantly allocating and freeing memory on the heap. Why not allocate a fixed chunk of memory once for those objects and reuse it as they're created and destroyed? This way, you can reduce GC pressure and avoid unnecessary overhead.

The specific Go feature that provides this is the `sync.Pool` type from the standard library. In a way, it basically keeps allocated but unused items for later reuse, so that we don't need to do allocations again and again.

I think there's one thing to pay attention to here, though. If the objects you create are already on the stack and don't escape to the heap, you're probably better off not using `sync.Pool` at all. Stack allocations are simply faster.

In fact, I quickly tested this idea, with the following benchmark:

```golang
package main

import (
    "sync"
    "testing"
)

type Obj struct {
    A, B, C int
}

// Benchmark 1: Stack allocation (object doesn't escape)
func BenchmarkStackAlloc(b *testing.B) {
    for i := 0; i < b.N; i++ {
        obj := Obj{A: i, B: i + 1, C: i + 2}
        _ = obj.A + obj.B + obj.C // use it so compiler doesn't optimize it away
    }
}

// Benchmark 2: Using sync.Pool
func BenchmarkPoolAlloc(b *testing.B) {
    pool := sync.Pool{
        New: func() any {
            return new(Obj)
        },
    }
    for i := 0; i < b.N; i++ {
        obj := pool.Get().(*Obj)
        obj.A, obj.B, obj.C = i, i+1, i+2
        _ = obj.A + obj.B + obj.C
        pool.Put(obj)
    }
}
```

Here is the results I have by running `go test -bench=. -benchmem`:

```bash
BenchmarkStackAlloc-8           1000000000               0.2543 ns/op          0 B/op          0 allocs/op
BenchmarkPoolAlloc-8            159609063                7.440 ns/op           0 B/op          0 allocs/op
```

So yeah, just make sure you are using `sync.Pool` for objects that actually escape to the heap, and also check that you meet the criterias outlined in [When Should You Use `sync.Pool`](https://goperf.dev/01-common-patterns/object-pooling/#when-should-you-use-syncpool)

## Memory Preallocation

In Go, some data structures like slices adjust their capacity by reallocating memory. For example, we can define a slice without specifying its size and keep appending elements to it. It's possible to write something like this:

```golang
var nums []int
for i := 0; i < 100000; i++ {
    nums = append(nums, i)
}
```

The problem here is that every time the slice runs out of capacity, Go allocates a new underlying array and copies all the existing elements into it. This process keeps repeating as the slice grows. This is unwanted as it means we're constantly allocating and copying memory.

So, it's better to write the same thing like this:

```golang
nums := make([]int, 0, 100000)
for i := 0; i < 100000; i++ {
    nums = append(nums, i)
}
```

This is going to be faster. Because we've already told Go how many elements we expect in nums. It no longer needs to start with a small capacity and reallocate as more elements are appended.

Long story short: **If you know the final size of your data beforehand, preallocate the memory.**

If you're interested in seeing this measured in practice, the Go Optimization Guide explains and [benchmarks](https://goperf.dev/01-common-patterns/mem-prealloc/#benchmarking-impact) this concept pretty clearly in their [Memory Preallocation](https://goperf.dev/01-common-patterns/mem-prealloc) section.

## Reuse Memory

Another effective way to reduce GC pressure is to reuse existing data structures instead of allocating new ones.

The core idea is simple. Whenever you find yourself creating new objects, cloning data, or rebuilding structures, ask whether you can achieve the same result by reusing what you already have.

To make this idea clearer, let's go over a few examples you're likely to run into sooner or later.

### Example 1: String Concatenation

Consider the simple case of string concatenation. When you add two strings using the + operator, Go creates a new string each time. This is because strings are immutable. This means that even a small concatenation results in new memory allocations for the combined result.

Instead, we can reuse memory by writing into a preallocated buffer, such as `strings.Builder`. This both helps reduce the number of new allocations being made as well as latency and the GC pressure.

For example, consider the following two benchmarks:


```golang
package main

import (
    "strings"
    "testing"
)

var sink string

var words = []string{
    "alpha", "bravo", "charlie", "delta", "echo",
    "foxtrot", "golf", "hotel", "india", "juliet",
    "kilo", "lima", "mike", "november", "oscar",
    "papa", "quebec", "romeo", "sierra", "tango",
    "uniform", "victor", "whiskey", "xray", "yankee", "zulu",
}

func BenchmarkStringConcat(b *testing.B) {
    for b.Loop() {
        s := ""
        for _, w := range words {
            s += w
        }
        sink = s
    }
}

func BenchmarkStringBuilder(b *testing.B) {
    var sb strings.Builder
    for b.Loop() {
        sb.Reset()
        for _, w := range words {
            sb.WriteString(w)
        }
        sink = sb.String()
    }
}
```

Let me explain what's happening in these benchmarks. BenchmarkStringConcat creates a new string on every +=. BenchmarkStringBuilder, on the other hand, uses a single growing buffer internally and appends each write to it. It only performs one final copy when calling String(). The builder reuses its existing memory and only reallocates when the buffer isn't large enough. Thanks to this, it results in fewer allocations overall. Fewer allocations mean less work for the garbage collector. So the `StringBuilder` method is generally better performance in both speed and memory usage.

It is even possible to go faster. Remember the [Memory Preallocation](#memory-preallocation) section we covered earlier? Well, `strings.Builder` type also has a method called `Grow` that allows us to preallocate.

```golang
func BenchmarkStringBuilderGrow(b *testing.B) {
    // Precompute total size to avoid internal resizes.
    total := 0
    for _, w := range words {
        total += len(w)
    }

    var sb strings.Builder
    for i := 0; i < b.N; i++ {
        sb.Reset()
        sb.Grow(total)
        for _, w := range words {
            sb.WriteString(w)
        }
        sink = sb.String()
    }
}
```

Now, by running `go test -bench=. -benchmem`, I see the following results:
```bash
BenchmarkStringConcat-8          2353044               510.9 ns/op          2016 B/op         25 allocs/op
BenchmarkStringBuilder-8         6915342               173.9 ns/op           504 B/op          6 allocs/op
BenchmarkStringBuilderGrow-8    13563996                88.26 ns/op          144 B/op          1 allocs/op
```

Which makes sense. Regular string concatenation with `+=` is the slowest. Using `strings.Builder` with preallocation outperforms the plain builder version.

### Example 2: Encoding Messages

Consider a scenario where you're working with an RPC or WebSocket connection. Every time you send a message, you need to serialize (encode) a struct into a sequence of bytes.

Take a look at the two benchmarks in the following example:


```golang
package main

import (
    "bytes"
    "encoding/json"
    "testing"
    "math/rand"
)

type Message struct {
    ID   int    `json:"id"`
    Type string `json:"type"`
    Data string `json:"data"`
}

var sink []byte

func BenchmarkEncodeNewBuffer(b *testing.B) {
    msg := Message { ID: rand.Int(), Type: "subscribe", Data: "fancy_topic" }
    for i := 0; i < b.N; i++ {
        buf := new(bytes.Buffer)
        _ = json.NewEncoder(buf).Encode(msg)
    }
}

func BenchmarkEncodeReusedBuffer(b *testing.B) {
    msg := Message { ID: rand.Int(), Type: "subscribe", Data: "fancy_topic" }
    buf := new(bytes.Buffer)
    enc := json.NewEncoder(buf)

    for i := 0; i < b.N; i++ {
        buf.Reset()
        _ = enc.Encode(msg)
    }
}
```

When we run the benchmarks, we get the following results:

```bash
BenchmarkEncodeNewBuffer-8               7850647               127.7 ns/op           160 B/op             3 allocs/op
BenchmarkEncodeReusedBuffer-8           11686770               102.9 ns/op            48 B/op             1 allocs/op
```

In `BenchmarkEncodeNewBuffer`, as you can see, there are 3 allocations for each operation. This happens because, in each iteration, we create a new buffer with `new(bytes.Buffer)` and a new encoder with `json.NewEncoder(buf)`. Both of these allocate memory. Additionally, the call to `Encode(msg)` also performs some internal allocations. So, we end up with 3 allocations per operation.

`BenchmarkEncodeReusedBuffer`, on the other hand, creates the buffer and encoder for once. After they are created once, they are being reused in the loops. Only the `enc.Encode(msg)` call performs internal allocations. So, we end up with only one allocation per operation.

### Other examples

To be honest, there are many other situations where we can benefit from this idea of reusing memory. We can turn immutable byte data into mutable slices for reuse. We can reuse buffers for I/O operations instead of creating new ones each time (like the encoding example). We can even reuse structs by resetting them instead of allocating new ones.

The key thing to pay attention to here is applying this technique carefully. Avoid overcomplicating your code. Try to keep any side effects from memory reuse as localized as possible. As long as it's used appropriately, reusing memory is a very powerful technique for easing the load on the garbage collector.

# BONUS: The Green Tea Garbage Collector

As I'm writing this, there's an ongoing effort to make Go's garbage collector even more performant. The work is part of a new proposal called "Green Tea GC". You can follow the discussion and progress in [issue](https://github.com/golang/go/issues/73581) on GitHub.

# To Conclude

So, we've covered quite a bit in this essay. The Go GC Pacer, the algorithm choices that are used for implementing the GC, the practical ways to help the GC, such as reducing allocations, pooling objects, preallocating memory, and reusing existing data structures...

I believe, If there's one thing to take away from all this, it's that we should stay mindful of the garbage our code generates. Even though the GC hides it from us, it still happens under the hood and has real effects on how our programs perform.

Anyways, I hope this post helped you build a clearer intuition about how Go's garbage collector works. If you spotted something I missed, or have other insights worth sharing, I'd  love to hear about them.

Thanks for reading all the way through...
