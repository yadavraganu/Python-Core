## Core Built-In Types

| Type         | Mutability | Order | Duplicates | Lookup/Indexing | Insertion/Append  | Memory Footprint | Use Case Highlights |
|--------------|------------|-------|------------|-----------------|-------------------|------------------|---------------------|
| **list**     | Mutable    | Yes   | Yes        | O(1) for index, O(n) for membership test | O(1) amortized append  | Moderate         | Ordered, resizable sequence with duplicates |
| **tuple**    | Immutable  | Yes   | Yes        | O(1) for index, O(n) for membership test | N/A               | Low              | Fixed-size records; hashable keys for dicts |
| **set**      | Mutable    | No    | No         | O(1) membership/test     | O(1)               | Moderate         | Unique-item collection; fast de-dup and lookups |
| **frozenset**| Immutable  | No    | No         | O(1) membership/test     | N/A               | Moderate         | Hashable set for use as dict keys or in other sets |
| **dict**     | Mutable    | Yes*  | N/A        | O(1) key lookup          | O(1)               | High             | Key→value mapping; fast lookups and inserts |

\*Insertion-ordered since Python 3.7

### list
- **When to Use:**  
  – You need an ordered, growable collection that may include duplicates.  
  – Frequent random access by index.  
- **When to Avoid:**  
  – Large datasets requiring many membership tests (`x in lst`)—those run in O(n).  
  – Repeated insertions/removals at front or middle (shifts cost O(n)).  
- **Performance Notes:**  
  – `append()`: O(1) amortized (occasional resize costs more).  
  – `pop()` from end: O(1); from front: O(n).  
  – Memory overhead grows to support dynamic resizing.  

### tuple
- **When to Use:**  
  – Fixed collections of items (e.g., coordinates, record rows).  
  – Use as keys in dicts or elements in sets (hashable).  
- **When to Avoid:**  
  – You need to modify, append, or reorder elements.  
- **Performance Notes:**  
  – Iteration slightly faster than list due to immutability optimizations.  
  – Lower memory overhead than list because no resizing buffer.  

### set
- **When to Use:**  
  – You need unique elements and O(1) membership tests.  
  – Performing set algebra: union, intersection, difference.  
- **When to Avoid:**  
  – Order matters for your logic or output.  
  – You require indexed access or slicing.  
- **Performance Notes:**  
  – `add()`, `remove()`, `in`: O(1) average; worst-case O(n) if many hash collisions.  
  – Memory: stores hash table overhead per element.  

### frozenset
- **When to Use:**  
  – All the benefits of `set` plus immutability.  
  – Use as dict keys or inside other sets.  
- **When to Avoid:**  
  – Any scenario where you must update the collection.  
- **Performance Notes:**  
  – Same time complexities as `set`.  
  – Slightly lower overhead because no methods to mutate internal table structure.  

### dict
- **When to Use:**  
  – Mapping unique keys to values with constant-time lookups.  
  – Configs, lookup tables, JSON-like structures.  
- **When to Avoid:**  
  – When insertion order pre-3.7 mattered and you needed extra ordering features (use `OrderedDict`).  
- **Performance Notes:**  
  – `__getitem__`, `__setitem__`, `in`: O(1) average, O(n) worst-case.  
  – Higher memory footprint to maintain hash table and handle collisions.  

## Specialized Collections (`collections` Module)

| Type            | Mutability | Order | Lookup/Indexing | Insertion/Append | Best For                        | Avoid When                |
|-----------------|------------|-------|-----------------|------------------|---------------------------------|---------------------------|
| **deque**       | Mutable    | Yes   | O(n)            | O(1) at both ends| Fast queues/stacks, sliding windows | Random access/slicing    |
| **defaultdict** | Mutable    | Yes   | O(1)            | O(1)             | Grouping/counting w/o key checks   | Strict missing-key control |
| **OrderedDict** | Mutable    | Yes   | O(1)            | O(1)             | LRU caches, reordering on updates   | Python ≥3.7 plain dicts  |
| **Counter**     | Mutable    | No    | O(1)            | O(1)             | Frequency tallying                  | Large sparse counts      |
| **ChainMap**    | Mutable    | Yes   | O(k) for k maps | O(1)             | Layered configs, nested scopes      | Needing single underlying dict |
| **namedtuple**  | Immutable  | Yes   | O(1) via name   | N/A              | Lightweight record objects          | Behavior-rich classes    |
| **UserDict**, **UserList**, **UserString** | Mutable/Immutable | Yes | Respective type | Respective type | Customizing behavior of built-ins | When subclassing built-ins directly |

### deque
- **When to Use:**  
  – Real-time FIFO/LIFO queues, sliding window, BFS.  
- **When to Avoid:**  
  – Random-access lookups or slicing—those are O(n).  
- **Performance:**  
  – `append()`, `appendleft()`, `pop()`, `popleft()`: O(1)

### defaultdict
- **When to Use:**  
  – Auto-initialize missing keys (e.g., grouping lists, counting).  
- **When to Avoid:**  
  – You need to strictly detect missing keys (avoiding silent defaults).  
- **Performance:**  
  – Same as dict: O(1) for get/set, with default factory invoked on miss

### OrderedDict
- **When to Use:**  
  – Pre-3.7: preserve insertion order plus support methods like `move_to_end()`.  
- **When to Avoid:**  
  – Python 3.7+ when basic insertion order suffices (plain `dict` is faster).  
- **Performance:**  
  – Slight overhead vs `dict` due to doubly linked list maintenance

### Counter
- **When to Use:**  
  – Counting frequencies of hashable items in one pass.  
- **When to Avoid:**  
  – When you need order or grouping beyond counts.  
- **Performance:**  
  – `update()` and lookups are O(1) per element; uses dict underneath

### ChainMap
- **When to Use:**  
  – Merging multiple dict contexts (e.g., multiple config layers).  
- **When to Avoid:**  
  – You must update all underlying maps at once or need a standalone dict.  
- **Performance:**  
  – Lookup: checks each mapping in turn, O(k) for k maps; insertions go to first map

### namedtuple
- **When to Use:**  
  – Lightweight record types with named fields and tuple semantics.  
- **When to Avoid:**  
  – When you need methods or mutable attributes—use `@dataclass` or classic class.  
- **Performance:**  
  – Attribute access: O(1); memory usage comparable to tuple plus small overhead for field names

## Choosing the Right Collection

1. **Need mutability?**  
   – Yes: `list`, `dict`, `set`, `deque`, `defaultdict`, `Counter`, `OrderedDict`  
   – No: `tuple`, `frozenset`, `namedtuple`

2. **Need ordering?**  
   – Yes: `list`, `tuple`, `dict` (3.7+), `deque`, `OrderedDict`, `namedtuple`, `ChainMap`  
   – No: `set`, `frozenset`, `Counter`

3. **Need fast membership?**  
   – O(1): `set`, `frozenset`, `dict`, `defaultdict`, `Counter`  
   – O(n): `list`, `tuple`, `deque` (for value search)

4. **Need fixed-size record/hashable?**  
   – Use `tuple` or `frozenset`; for readability, prefer `namedtuple`.

5. **Need frequency counts or auto-key defaults?**  
   – Use `Counter` or `defaultdict(int)` respectively.
