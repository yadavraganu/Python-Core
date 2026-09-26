Python uses a private heap space where all objects and data structures reside. This heap is managed by the Python memory manager, and programmers do not have direct access to it.

Within this private heap, Python employs:
- __Object-specific allocators:__ Different data types (integers, strings, lists, dictionaries, etc.) have optimized allocators for efficient memory handling.
- __Memory pools and block allocators:__ For small objects, Python uses memory pools (pre-allocated chunks of memory) to quickly allocate and deallocate. For larger objects, block allocators are used.

Python primarily uses a hybrid approach for garbage collection, combining:
- Reference Counting (Primary Mechanism)
- Generational Garbage Collection (for Cyclic References)

## 1. Reference Counting
Reference counting is Python's fundamental and most immediate garbage collection mechanism.
How it works: Every object in Python has a reference count, which is an integer that tracks the number of references (variables, container elements, etc.) pointing to that object.

__Incrementing the count:__ The reference count increases whenever:
- A new variable refers to the object.
- The object is passed as an argument to a function.
- The object is placed in a container (e.g., a list, tuple, dictionary).
  
__Decrementing the count:__ The reference count decreases whenever:
- A variable referencing the object goes out of scope.
- A reference is explicitly deleted (e.g., using del).
- The object is removed from a container.
- A variable referencing the object is reassigned to another object.
  
__Deallocation:__ When an object's reference count drops to zero, it means no part of the program can access that object anymore. Python immediately deallocates the memory occupied by that object, making it available for new objects.

__Advantages of Reference Counting:__
- Simplicity: It's a straightforward mechanism to implement.
- Immediacy: Memory is reclaimed as soon as an object is no longer referenced, reducing memory footprint and potential memory spikes.
- Limitations of Reference Counting: Cyclic References
  
The main drawback of pure reference counting is its inability to handle cyclic references. This occurs when two or more objects reference each other, forming a closed loop, even if no external references point to the cycle.
```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

node1 = Node(1)
node2 = Node(2)

node1.next = node2
node2.next = node1 # Cyclic reference

# At this point, even if node1 and node2 go out of scope,
# their reference counts will never drop to zero because they refer to each other.
# They become "uncollectable" by reference counting alone.
```

## 2. Generational Garbage Collection (Tracing Garbage Collector)
To address the issue of cyclic references, Python employs a more sophisticated generational garbage collector, which is also a form of tracing garbage collection (specifically, a mark-and-sweep variant). This collector runs periodically.
The core idea behind generational garbage collection is based on the generational hypothesis:
Most objects are short-lived and die soon after creation (e.g., temporary variables in a function).
A small percentage of objects are long-lived and survive for a long time.
Python's generational garbage collector categorizes objects into three "generations" based on their age:

- __Generation 0 (Youngest):__ Contains newly created objects. This generation is collected most frequently because most new objects are expected to become unreachable quickly.

- __Generation 1 (Middle-aged):__ Holds objects that have survived one or more Generation 0 collections. It's collected less often than Generation 0.

- __Generation 2 (Oldest):__ Contains long-lived objects that have survived multiple collections in Generation 0 and 1. This generation is collected least frequently.

### How Generational Collection Works (Mark-and-Sweep):
When a generational collection is triggered (based on internal thresholds, which track the number of allocations and deallocations):

- __Mark Phase:__ The collector starts from a set of "root" objects (e.g., global variables, objects on the call stack) and traverses the object graph, marking all reachable objects. This means any object that can be accessed by the running program is marked as "alive."

- __Sweep Phase:__ After marking, the collector "sweeps" through the memory, deallocating all unmarked objects. These unmarked objects are considered "garbage" because they are not reachable from any active part of the program, even if they have a non-zero reference count due to cyclic references.

- __Promotion:__ Objects that survive a collection in Generation 0 are "promoted" to Generation 1. Similarly, objects surviving Generation 1 are promoted to Generation 2. This ensures that long-lived objects are checked less frequently, reducing the overhead of garbage collection.

- __Thresholds:__
The garbage collector runs when the number of allocations minus deallocations in a generation exceeds a specific threshold. These thresholds can be inspected and modified using the gc module. For example, gc.get_threshold() returns a tuple (threshold0, threshold1, threshold2).

## The gc Module
The gc module in Python provides an interface to interact with the garbage collector.
### Key functions and attributes:
- `gc.enable()`: Enables automatic garbage collection. (It's enabled by default).
- `gc.disable()`: Disables automatic garbage collection. Use with caution, as it can lead to memory leaks if not managed manually.
- `gc.collect(generation=None)`: Forces a garbage collection. If generation is not provided, it runs a full collection across all generations.You can specify generation (0, 1, or 2) to collect a specific generation and younger ones.Returns the number of uncollectable objects found.
- `gc.get_count()`: Returns a tuple (count0, count1, count2) representing the current collection counts for each generation.
- `gc.get_threshold()`: Returns the current collection thresholds as a tuple (threshold0, threshold1, threshold2).
- `gc.set_threshold(threshold0[, threshold1[, threshold2]])`: Sets the collection thresholds.
- `gc.get_objects()`: Returns a list of all objects that the collector is currently tracking. Useful for debugging memory usage.
- `gc.get_referrers(obj)`: Returns a list of objects that directly refer to obj.
- `gc.get_referents(obj)`: Returns a list of objects that obj directly refers to.
gc.garbage: A list of objects that the collector found to be uncollectable (e.g., due to cyclic references with `__del__` methods that prevent finalization).
