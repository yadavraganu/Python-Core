## Core Built-In Types

| Type         | Mutability | Order | Duplicates | Lookup/Indexing | Insertion/Append  | Memory Footprint | Use Case Highlights |
|--------------|------------|-------|------------|-----------------|-------------------|------------------|---------------------|
| **list**     | Mutable    | Yes   | Yes        | O(1) for index, O(n) for membership test | O(1) amortized append  | Moderate         | Ordered, resizable sequence with duplicates |
| **tuple**    | Immutable  | Yes   | Yes        | O(1) for index, O(n) for membership test | N/A               | Low              | Fixed-size records; hashable keys for dicts |
| **set**      | Mutable    | No    | No         | O(1) membership/test     | O(1)               | Moderate-High    | Unique-item collection; fast de-dup and lookups |
| **frozenset**| Immutable  | No    | No         | O(1) membership/test     | N/A               | Moderate-High    | Hashable set for use as dict keys or in other sets |
| **dict**     | Mutable    | Yes*  | N/A        | O(1) key lookup          | O(1)               | High             | Key→value mapping; fast lookups and inserts |

\*Insertion-ordered since Python 3.7

### list
- **When to Use:**  
  – You need an ordered, growable collection that may include duplicates.  
  – Frequent random access by index.  
  – Sequential iteration with modifications (via loops).
- **When to Avoid:**  
  – Large datasets requiring many membership tests (`x in lst`)—those run in O(n).  
  – Repeated insertions/removals at front or middle (shifts cost O(n)).  
  – Use as dictionary keys (unhashable).
- **Performance Notes:**  
  – `append()`: O(1) amortized (occasional resize costs more).  
  – `pop()` from end: O(1); from front or middle: O(n).  
  – `insert(i, x)`: O(n) due to element shifting.
  – `del lst[i]`: O(n) for front/middle, O(1) for end.
  – Slicing `lst[a:b]`: O(b-a) creates new list.
  – Iteration: O(n), memory overhead grows to support dynamic resizing.
  – **Thread-safe for atomic operations only** (append, pop); not safe for multi-step sequences.

### tuple
- **When to Use:**  
  – Fixed collections of items (e.g., coordinates, record rows, function return values).  
  – Use as keys in dicts or elements in sets (hashable).  
  – When immutability guarantees data integrity.
- **When to Avoid:**  
  – You need to modify, append, or reorder elements.  
  – Very large homogeneous sequences (consider NumPy arrays).
- **Performance Notes:**  
  – Iteration O(n), slightly faster than list due to immutability optimizations.  
  – Lower memory overhead than list because no resizing buffer.  
  – Unpacking: O(n); equality checking: O(n) until first difference.
  – Single element tuple requires trailing comma: `(x,)`.

### set
- **When to Use:**  
  – You need unique elements and O(1) membership tests.  
  – Performing set algebra: union, intersection, difference.  
  – Removing duplicates from a collection.
  – Checking exclusivity/containment efficiently.
- **When to Avoid:**  
  – Order matters for your logic or output.  
  – You require indexed access or slicing.  
  – Storing unhashable items (lists, dicts, sets).
- **Performance Notes:**  
  – `add()`, `remove()`, `discard()`, `in`: O(1) average; worst-case O(n) if many hash collisions.  
  – Set operations: `union()` O(n+m), `intersection()` O(min(n,m)), `difference()` O(n).
  – Memory: higher due to hash table overhead per element (~24 bytes per entry overhead).  
  – Iteration: O(n), order not guaranteed.

### frozenset
- **When to Use:**  
  – All the benefits of `set` plus immutability.  
  – Use as dict keys or inside other sets.  
  – Function parameters where mutation must be prevented.
- **When to Avoid:**  
  – Any scenario where you must update the collection (use `set` instead).  
  – Performance-critical scenarios (microseconds matter)—mutable `set` may be faster due to fewer safety checks.
- **Performance Notes:**  
  – Same time complexities as `set`.  
  – Slightly lower overhead because fewer mutation checks in implementation.  
  – Hashing: computed once at creation, cached (unlike unhashable types).

### dict
- **When to Use:**  
  – Mapping unique keys to values with constant-time lookups.  
  – Configs, lookup tables, JSON-like structures, caching.  
  – Counting occurrences (use alongside or instead of `Counter`).
- **When to Avoid:**  
  – When insertion order pre-3.7 mattered and you needed extra ordering features (use `OrderedDict`).  
  – Keys must be unhashable; use nested structures or custom key transforms instead.
  – When you need ordered iteration in Python <3.7 (use `OrderedDict`).
- **Performance Notes:**  
  – `__getitem__`, `__setitem__`, `in`: O(1) average, O(n) worst-case (hash collisions).  
  – `keys()`, `values()`, `items()`: O(n) to construct views, O(1) to iterate once created.
  – `pop(key)`, `get(key)`: O(1) average.  
  – Higher memory footprint (~240 bytes per empty dict) to maintain hash table and handle collisions.
  – Iteration order: insertion-order (Python 3.7+), but **do not rely on this for compatibility**.

---

## Specialized Collections (`collections` Module)

| Type            | Mutability | Order | Lookup/Indexing | Insertion/Append | Best For                        | Avoid When                |
|-----------------|------------|-------|-----------------|------------------|---------------------------------|---------------------------|
| **deque**       | Mutable    | Yes   | O(n)            | O(1) at both ends| Fast queues/stacks, sliding windows | Random access/slicing    |
| **defaultdict** | Mutable    | Yes   | O(1)            | O(1)             | Grouping/counting w/o key checks   | Strict missing-key control |
| **OrderedDict** | Mutable    | Yes   | O(1)            | O(1)             | LRU caches, reordering on updates   | Python ≥3.7 plain dicts  |
| **Counter**     | Mutable    | No    | O(1)            | O(1)             | Frequency tallying                  | Large sparse counts      |
| **ChainMap**    | Mutable    | Yes   | O(k) for k maps | O(1)             | Layered configs, nested scopes      | Needing single underlying dict |
| **namedtuple**  | Immutable  | Yes   | O(1) via name   | N/A              | Lightweight record objects          | Behavior-rich classes    |
| **UserDict**, **UserList**, **UserString** | Mutable | Yes | Respective type | Respective type | Customizing behavior of built-ins | Direct subclassing built-ins |

### deque
- **When to Use:**  
  – Real-time FIFO/LIFO queues, sliding window algorithms, BFS/DFS.  
  – Efficient rotation of collections (`rotate(n)`).
  – Buffer with fast append/pop on both ends.
- **When to Avoid:**  
  – Random-access lookups or slicing—those are O(n).  
  – Frequent index-based updates; use `list` instead.
  – Memory-constrained scenarios (overhead per node: ~56 bytes).
- **Performance:**  
  – `append()`, `appendleft()`, `pop()`, `popleft()`: O(1)  
  – `rotate(n)`: O(n)  
  – Iteration: O(n), access by index: O(n)  
  – **Thread-safe for atomic operations** when `maxlen` is set; otherwise similar caveats as list.

### defaultdict
- **When to Use:**  
  – Auto-initialize missing keys without KeyError (e.g., grouping lists, counting).  
  – Simplifies code: no need for `if key in dict` checks.
  – Building accumulator structures (e.g., `defaultdict(list)` or `defaultdict(int)`).
- **When to Avoid:**  
  – You need to strictly detect missing keys (avoiding silent defaults).  
  – Silent initialization masks logic errors.
  – Serialization (JSON doesn't handle defaultdict natively).
- **Performance:**  
  – Same as dict: O(1) for get/set, with default factory invoked on miss.  
  – Factory function called only for missing keys.  
  – No performance penalty over `dict` for existing keys.

### OrderedDict
- **When to Use:**  
  – Pre-3.7: preserve insertion order plus support methods like `move_to_end()`.  
  – LRU cache implementations (reorder on access via `move_to_end()`).
  – Explicit ordering guarantees for backward compatibility.
- **When to Avoid:**  
  – Python 3.7+ when basic insertion order suffices (plain `dict` is faster, ~20% overhead reduction).  
  – Extreme performance requirements; plain `dict` is preferred.
- **Performance:**  
  – Slight overhead vs `dict` (~25-30% slower) due to doubly linked list maintenance.  
  – `move_to_end(key)`: O(1) operation.  
  – Iteration: O(n), guaranteed insertion order.

### Counter
- **When to Use:**  
  – Counting frequencies of hashable items in one pass.  
  – Finding most common elements via `.most_common(n)`.
  – Set-like operations on tallies (add/subtract counts).
- **When to Avoid:**  
  – When you need custom ordering beyond counts.  
  – Sparse data where most keys have count=0 (memory overhead).
  – Negative counts are rare; use manually if needed.
- **Performance:**  
  – `update()` and lookups: O(1) per element; uses `dict` underneath.  
  – `.most_common(n)`: O(k log k) where k is number of unique elements.  
  – Subtraction/intersection: O(n) where n is total keys.  
  – Memory: same as dict plus count storage (~28 bytes per entry).

### ChainMap
- **When to Use:**  
  – Merging multiple dict contexts (e.g., multiple config layers, environment + defaults).  
  – Scoping/namespace simulation without copying dicts.
  – Fallback chains: primary config → secondary config → defaults.
- **When to Avoid:**  
  – You must update all underlying maps at once (only updates first map).  
  – Needing a single unified dict (use `.copy()` or dict merge instead).
  – Performance-critical lookups across many layers.
- **Performance:**  
  – Lookup: checks each mapping in turn, O(k) for k maps; returns first match.  
  – Insertion/deletion: only affects first map, O(1) per operation.  
  – Iteration: visits all keys from all maps; may see duplicates (first wins).  
  – Memory: O(k) overhead for k mappings; no data duplication.

### namedtuple
- **When to Use:**  
  – Lightweight record types with named fields and tuple semantics.  
  – Function return values with named components (`return Point(x=1, y=2)`).  
  – Keys in dicts or set members (hashable, lightweight).
- **When to Avoid:**  
  – When you need methods or mutable attributes—use `@dataclass` or classic class.  
  – High inheritance complexity; use classes instead.
  – Large datasets (use NumPy structured arrays for better performance).
- **Performance:**  
  – Attribute access: O(1) (faster than plain tuple unpacking for large tuples).  
  – Memory usage: comparable to tuple plus ~8 bytes per field name overhead.  
  – Creation: `_make()` O(n); unpacking O(n).  
  – Iteration: O(n), same as tuple.

---

## Common Pitfalls & Best Practices

### Mutability & Hashing
- **Never use mutable objects (list, dict, set) as dict keys or set members**—they're unhashable.
- **Mutable default arguments** in functions: `defaultdict(list)` is safe; `dict = {}` in function signature is not.

### Iteration Safety
- **Don't modify a list while iterating over it** (causes skipped/repeated elements). Use list comprehension or `list.copy()` instead.
- **Set/dict mutation during iteration** raises `RuntimeError`; use `.copy()` or create a list of items first.

### Performance Traps
- **Checking `x in list` is O(n)**; use `set` for large datasets with many lookups.
- **String concatenation in loops**: use `''.join([...])` or `io.StringIO()`, not `+=`.
- **Deepcopy is slow**: verify you need it; often a reference or shallow copy suffices.

### Memory Awareness
- **Empty containers still consume memory**: `[] → ~56 bytes`, `{} → ~240 bytes`, `set() → ~200 bytes`.
- **Slicing creates new objects**: `lst[1:1000]` copies all 999 elements; consider generators if possible.

---

## Choosing the Right Collection: Decision Tree

1. **Need mutability?**  
   – **Yes:** `list`, `dict`, `set`, `deque`, `defaultdict`, `Counter`, `OrderedDict`  
   – **No:** `tuple`, `frozenset`, `namedtuple`

2. **Need ordering?**  
   – **Yes:** `list`, `tuple`, `dict` (3.7+), `deque`, `OrderedDict`, `namedtuple`, `ChainMap`  
   – **No:** `set`, `frozenset`, `Counter`

3. **Need O(1) random access by index?**  
   – **Yes:** `list`, `tuple`, `namedtuple`  
   – **No:** `set`, `frozenset`, `dict`, `deque`

4. **Need fast membership tests (`x in collection`)?**  
   – **O(1):** `set`, `frozenset`, `dict`, `defaultdict`, `Counter`  
   – **O(n):** `list`, `tuple`, `deque`

5. **Need to use as dict key or set member (hashable)?**  
   – **Use:** `tuple`, `frozenset`, `namedtuple`, or scalar types (`int`, `str`, etc.)  
   – **Avoid:** `list`, `dict`, `set`

6. **Frequent insertions/deletions at both ends?**  
   – **Yes:** `deque` (O(1) for both ends)  
   – **No:** `list` (O(n) for front/middle)

7. **Need frequency counts or auto-key defaults?**  
   – **Frequency counts:** `Counter`  
   – **Auto-initialize missing keys:** `defaultdict(list)` or `defaultdict(int)`

8. **Need lightweight records with named attributes?**  
   – **Yes:** `namedtuple` (immutable) or `@dataclass` (mutable, more features)  
   – **No:** use dict or class instance

---

## Quick Reference: Memory & Performance

| Collection  | Memory/Item | Empty Overhead | Insertion at End | Lookup | Membership | Notes |
|-------------|-------------|---|---|---|---|---|
| `list`      | 28-32 bytes | ~56 bytes | O(1) amortized | O(1) by index | O(n) | Dynamic resizing overhead |
| `tuple`     | 28-32 bytes | ~56 bytes | N/A | O(1) by index | O(n) | Immutable, hashable |
| `dict`      | ~80 bytes (key+value) | ~240 bytes | O(1) | O(1) by key | O(1) | Hash collision cost |
| `set`       | ~24 bytes   | ~200 bytes | O(1) | N/A | O(1) | Unordered, unique only |
| `frozenset` | ~24 bytes   | ~200 bytes | N/A | N/A | O(1) | Immutable set |
| `deque`     | ~56 bytes   | ~80 bytes | O(1) both ends | O(n) by index | O(n) | Linked list structure |
| `Counter`   | ~28 bytes (count only) | ~240 bytes | O(1) | O(1) | O(1) | Uses dict underneath |
| `namedtuple`| 28-32 bytes | ~80 bytes | N/A | O(1) by name | O(n) | Lightweight, hashable |

