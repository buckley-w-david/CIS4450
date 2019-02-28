# Garbage Collection

## History / Origins

* *Read history from Wiki*

## Current Best Practices

The advantage gained with garbage collection is mostly one of convienence. It is simply harder to intorduce a memory error when the language runtime is cleaning up after you.

The disadvantages of garbage collection are mostly performance concerns. The garbage collector in itself requires a non-zero amount of resources to run, and on top of that when it actually decides to cull unused memory that uses additional resources, and sometimes requires pausing execution of your code.

There are actually many different techniques that are all refered to as "garbage collection", that all have different characteristics.

### Tracing

Tracing is the most common type of garbage collection, used in languages such as Java and C#.

### Mark and Sweep

Mark and sweep garbage collection works by scanning the memory and marking each object that is currently in use by the program. Then it scans again, removing (sweeping) the unrefered to objects.

This was the method that McCarthy created for Lisp in 1960. A modernized variant of this algorithm is also using in golang.

### Reference Counting

Reference counting assigns a counter to each block of memory that will increment whenever a reference to that memory is added, and decrememnt whenever a reference is removed.

When the counter reaches 0, the block is deallocated.

An advantage of this method is that program execution does not need to halt wile garbage collection is happening like in some other methods.

Dissadvantages are that circular references (A references B, and B references A, but nothing else references A or B) will not be deallocated, and so represent a memory leak.

Likely the most popular example of this method is by Python.

### Escape analysis

While not a garbage collection mechanism itself, escape analysis is a compiler technique that can help garbage collected languages by determining memory that was going to be allocated on the heap that can actually be allocated on the stack, meaning less work for the garbage collector.

This is done using a compile-time analysis to determine whether an object allocated within a function is not accessible outside of it (i.e. escape) to other functions or threads.

---

## Practical Examples

### Perl circular reference leaks

```perl
sub leak {
    my ($foo, $bar);
    $foo = \$bar;
    $bar = \$foo;
}
```

In the example code above, $foo and $bar are never collected and a copy will persist after every invocation of leak() because both variables have a reference count of 1.

### Go runtime analysis

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	var m runtime.MemStats

	runtime.ReadMemStats(&m)
	fmt.Println("Mallocs: ", m.Mallocs)
	fmt.Println("Frees: ", m.Frees)

	runtime.ReadMemStats(&m)
	fmt.Println("Mallocs: ", m.Mallocs)
	fmt.Println("Frees: ", m.Frees)

	runtime.ReadMemStats(&m)
	fmt.Println("Mallocs: ", m.Mallocs)
	fmt.Println("Frees: ", m.Frees)
}
```

```bash
 $ go run example.go
Mallocs:  148
Frees:  1
Mallocs:  155
Frees:  2
Mallocs:  157
Frees:  3

```

The above go program reads it's own memory stats from the runtime and prints out the number of allocations and the number of deallocations performed.

You can see the garbage collector in action by watching the number of frees. Each time we read in the memory stats some of the information in the struct `m` (A struct of type `runtime.MemStats`) that had to be heap allocated and referecned by the struct.

When the reference was overwritten by reading in the data again, the garbage collector freed it.

---

## References

 * https://medium.com/@cjab/garbage-collection-cb91e45c8225
 * https://www.lucidchart.com/techblog/2017/10/30/the-dangers-of-garbage-collected-languages/
 * https://courses.cs.washington.edu/courses/csep521/07wi/prj/rick.pdf
 * https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)#Strategies
 * https://www.mtsoukalos.eu/Go-Garbage-Collector
 * https://stackoverflow.com/questions/2223721/common-perl-memory-reference-leak-patterns
