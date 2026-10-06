# Python Garbage Collection

Python uses a private heap space where all objects and data structures reside. This heap is managed by the Python memory manager; programmers don't access it directly.

Within this private heap, Python's small-object allocator (`pymalloc`) handles the huge number of small, short-lived objects typical code creates, via an arena/pool/block system — covered in full in `CPython-Object-Model.md`. On top of that memory layer, Python primarily uses a hybrid approach to decide **when an object's memory can be reclaimed**, combining:

- **Reference counting** (primary mechanism)
- **Generational (cyclic) garbage collection** (a backstop for reference cycles)

---

## 1. Reference Counting

Every object has a reference count — an integer tracking how many references point to it (`ob_refcnt` in the `PyObject` header, see `CPython-Object-Model.md`).

**Incrementing** happens when:
- A new variable refers to the object.
- The object is passed as a function argument.
- The object is placed in a container (list, tuple, dict, ...).

**Decrementing** happens when:
- A variable referencing it goes out of scope.
- A reference is explicitly deleted (`del`).
- The object is removed from a container.
- A variable is reassigned to something else.

**Deallocation:** when the count hits zero, no part of the program can reach the object anymore, and Python deallocates it **immediately** — not on some later sweep.

**Advantages:** simple to implement, and memory is reclaimed the instant it's no longer needed, avoiding the pause-the-world behavior some other languages' collectors have.

### Inspecting Reference Counts — `sys.getrefcount()`

```python
import sys

x = []
print(sys.getrefcount(x))   # 2, not 1!
```
**The off-by-one gotcha, confirmed by running it:** `getrefcount` itself takes `x` as an argument, creating a temporary extra reference for the duration of the call — so the reported count is always **one higher** than you'd naively expect. Binding a second name to the same object and checking again confirms the pattern:
```python
y = x
print(sys.getrefcount(x))   # 3 — x, y, and the temporary argument reference
del y
print(sys.getrefcount(x))   # back to 2
```

### The Limitation: Cyclic References

Pure reference counting cannot handle **cycles** — two or more objects referencing each other in a loop, even with no external references pointing in:

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

node1 = Node(1)
node2 = Node(2)
node1.next = node2
node2.next = node1   # cyclic reference

# Even after node1 and node2 go out of scope, each still holds a reference
# to the other, so neither's refcount ever reaches zero through refcounting alone.
```
**Confirmed empirically:** with the cycle collector disabled (`gc.disable()`), deleting both names leaves the objects un-finalized — nothing happens until `gc.collect()` is called explicitly, at which point they're correctly found and freed. Reference counting genuinely cannot reclaim this memory on its own; something else has to notice the cycle.

---

## 2. Generational Garbage Collection (Tracing Collector)

To catch what reference counting can't, Python runs a separate, periodic **generational, mark-and-sweep** tracing collector, based on the **generational hypothesis**: most objects die young, and a small fraction live a long time.

Objects are grouped into three generations by age:

- **Generation 0 (youngest):** newly created objects; collected most frequently.
- **Generation 1 (middle-aged):** survived at least one Gen 0 collection; collected less often.
- **Generation 2 (oldest):** survived multiple collections; collected least often. There is **no Generation 3** — once an object reaches Gen 2, it simply stays there and gets re-examined on every full collection from then on.

### How a Collection Runs (Mark-and-Sweep)

1. **Mark phase:** starting from a set of "roots" (globals, the call stack), the collector traverses the object graph and marks everything reachable as alive.
2. **Sweep phase:** everything left unmarked is garbage — unreachable from any live part of the program, *regardless of its reference count* — and is deallocated. This is exactly how a cyclic pair with external references removed gets reclaimed: nothing marks them as reachable, cycle or not.
3. **Promotion:** objects surviving a Gen 0 collection move to Gen 1; surviving Gen 1 moves them to Gen 2.

### What Triggers a Collection

Each generation has a threshold (`gc.get_threshold()`, default `(700, 10, 10)` — confirmed by running it). Gen 0's counter increases with each allocation not yet matched by a deallocation; once it exceeds `threshold0`, a Gen 0 collection runs. Collections cascade upward: after a certain number of Gen 0 collections (tracked against `threshold1`), a Gen 1 collection runs too (which also sweeps Gen 0); similarly, enough Gen 1 collections trigger a full Gen 2 collection (sweeping everything). This cascading is *why* older generations are checked far less often than Gen 0, keeping the common case (short-lived objects) cheap.

### Not Every Object Is Tracked

Only objects that can **hold references to other objects** ("container" types — instances, lists, dicts, sets, most tuples) are tracked by the cycle collector at all. Simple scalars that can't participate in a cycle are skipped entirely:

```python
import gc
gc.is_tracked(5)          # False — an int can't reference anything else
gc.is_tracked("hello")    # False — same for str
gc.is_tracked([1, 2, 3])   # True — a list could hold a reference back to itself
```
This is a nice detail if asked "does the GC have to check every object" — no, untrackable scalar types are excluded by construction, cutting the collector's workload.

---

## 3. `__del__` and Cycles: An Outdated Worry, Fixed Since Python 3.4

Older material (and older Python) will tell you that an object with a `__del__` method caught in a reference cycle is **permanently uncollectable**, leaking into `gc.garbage` forever — true before Python 3.4, but **no longer accurate**. **PEP 442** gave the cycle collector a way to determine a safe finalization order and call `__del__` on cyclic objects properly:

```python
import gc

class WithDel:
    def __init__(self, name): self.name = name
    def __del__(self): print(f"__del__ called for {self.name}")

a = WithDel("a")
b = WithDel("b")
a.other = b
b.other = a
del a, b

print("gc.garbage before collect:", gc.garbage)   # []
n = gc.collect()
# __del__ called for a
# __del__ called for b
print("collected:", n)                              # 2
print("gc.garbage after collect:", gc.garbage)       # still [] — properly finalized and freed
```
Modern `gc.garbage` is **normally empty**. It can still end up non-empty in rare cases — most notably if an object's own `__del__` *resurrects* it (creates a new strong reference to `self` during finalization), or with certain legacy C-extension types that don't fully support the cycle collector's finalization protocol — but "cycle + `__del__` = permanent leak" is no longer the correct mental model.

---

## 4. `weakref` — Intentionally Breaking Cycles

Rather than relying on the cycle collector to clean up after the fact, you can avoid creating a true cycle in the first place with the `weakref` module. A **weak reference** points to an object **without** incrementing its reference count, so it doesn't keep the object alive on its own.

```python
import weakref

class Big:
    pass

obj = Big()
r = weakref.ref(obj)
print(r() is obj)   # True — call it to get the actual object back
del obj
print(r())            # None — the weak reference doesn't block collection
```
With an optional callback, you can even be notified exactly when the referent is collected:
```python
def on_collected(weak_ref):
    print("the object was just garbage collected")

obj2 = Big()
r2 = weakref.ref(obj2, on_collected)
del obj2   # prints the callback message immediately — pure refcounting handles it,
            # since there's no real cycle to wait for the tracing collector on
```

**The classic use case — parent/child back-references:** a `Parent` holding a `Child`, and the `Child` holding a reference back to its `Parent`, is exactly the cyclic shape from Section 1. Making the child's back-reference a `weakref.ref` instead of a normal attribute breaks the cycle entirely — confirmed empirically: with the back-reference as a weak reference, `gc.collect()` after deleting both finds **0** unreachable objects, because plain reference counting already reclaimed them with no cycle to worry about.

Other common tools in the module:
- **`weakref.WeakValueDictionary`** / **`WeakKeyDictionary`** — dict variants whose entries disappear automatically once the referenced object is collected elsewhere. Ideal for caches that shouldn't themselves keep an object alive:
  ```python
  cache = weakref.WeakValueDictionary()
  val = Big()
  cache["key"] = val
  print("key" in cache)   # True
  del val
  print("key" in cache)    # False — the entry vanished on its own once val was collected
  ```
- **`weakref.proxy(obj)`** — like `ref()`, but behaves transparently like the object itself (attribute access forwards through), raising `ReferenceError` if used after the referent is gone, instead of requiring you to call it like `ref()` does.

---

## 5. The `gc` Module — Reference

| Function | What it does |
|---|---|
| `gc.enable()` | Enables automatic collection (on by default) |
| `gc.disable()` | Disables automatic collection — use with caution; cycles will only go away via a manual `gc.collect()` |
| `gc.collect(generation=None)` | Forces a collection (full, by default). Returns **the number of unreachable objects found** (and collected) — not "uncollectable objects," a common mis-description |
| `gc.get_count()` | Current `(count0, count1, count2)` allocation counters per generation |
| `gc.get_threshold()` / `gc.set_threshold(...)` | Reads/sets the `(threshold0, threshold1, threshold2)` trigger points |
| `gc.get_objects()` | All objects currently tracked by the collector — useful for memory debugging |
| `gc.get_referrers(obj)` | What refers *to* `obj` |
| `gc.get_referents(obj)` | What `obj` refers *to* |
| `gc.is_tracked(obj)` | Whether the collector tracks this object at all (see Section 2) |
| `gc.garbage` | Objects the collector found uncollectable — normally empty in modern Python (see Section 3) |
| `gc.freeze()` / `gc.unfreeze()` / `gc.get_freeze_count()` | Move all currently-tracked objects into a special permanent generation the collector skips, confirmed to work as described — used by frameworks that `fork()` worker processes (e.g., pre-loading app state in Gunicorn) to avoid the copy-on-write memory duplication a collection sweep would otherwise trigger across forked children |
| `gc.set_debug(flags)` | Enables collector debug output (e.g., `gc.DEBUG_STATS`), for diagnosing GC behavior directly |

---

## Notes & Gotchas

- **Reference counting is immediate; the generational collector is a periodic backstop specifically for cycles** — a strong answer distinguishes these two mechanisms rather than treating "Python's GC" as one monolithic thing.
- **`sys.getrefcount(x)` always reports one more than you'd expect**, because the call itself holds a temporary reference — confirmed above, and a common off-by-one trap when debugging reference leaks.
- **"A `__del__` method inside a cycle means it can never be collected" is outdated** — true before Python 3.4, fixed by PEP 442. Stating the *old* belief confidently in an interview is itself a minor red flag; know the modern behavior.
- **`gc.collect()`'s return value is unreachable objects found, not "uncollectable" ones** — the two are easy to conflate, and the actual docstring says "unreachable."
- **Not everything is GC-tracked** — scalar types that can't reference other objects (`int`, `str`, ...) are skipped by `gc.is_tracked()`, so the collector's real workload is smaller than "every live object."
- **`weakref` prevents a cycle from forming in the first place**, rather than relying on the tracing collector to clean one up after the fact — the standard fix for parent/child or observer/subject back-references, and a common follow-up question after discussing cycles.
- **There is no Generation 3** — Gen 2 is the ceiling; survivors just stay there and get rechecked on future full collections.
- **Disabling `gc` entirely can boost throughput for short-lived scripts that create no cycles**, but is risky in long-running processes, since any cycle created will leak for the process's lifetime without a manual `gc.collect()`.
