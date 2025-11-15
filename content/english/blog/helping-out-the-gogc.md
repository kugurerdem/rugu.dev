
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
