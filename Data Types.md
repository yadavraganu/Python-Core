# Python Data Types

Python's data types can be classified into several categories based on their properties and use cases. Everything in Python is an **object** — every type below is a class, and `type(x)` returns that class.

## 1. Numeric Types

Represent numbers and support arithmetic operations.

### Integral Types
| Type | Description |
|------|-------------|
| `int` | Arbitrary-precision integers (no overflow) |
| `bool` | Boolean values (`True`/`False`), subclass of `int` |

### Non-Integral Types
| Type | Description |
|------|-------------|
| `float` | Floating-point numbers (IEEE 754 double precision) |
| `complex` | Complex numbers with real and imaginary parts (`3+4j`) |
| `Decimal` | High-precision decimal arithmetic (`decimal` module) |
| `Fraction` | Rational numbers with exact arithmetic (`fractions` module) |

**Integer Caching:** CPython pre-allocates and reuses small integers `-5` to `256`. So `a = 100; b = 100; a is b` → `True`, but `a = 1000; b = 1000; a is b` → often `False`. This trips people up when explaining `is` vs `==`.

```python
a = 256
b = 256
print(a is b)   # True (cached)

a = 257
b = 257
print(a is b)   # False (not cached, implementation-dependent)
```

## 2. Sequence Types

Ordered collections, accessed by index.

| Type | Mutability | Description |
|------|-----------|-------------|
| `str` | Immutable | Text/Unicode strings |
| `list` | Mutable | Ordered, dynamic collection of items |
| `tuple` | Immutable | Ordered, fixed-size collection |
| `range` | Immutable | Sequence of numbers (memory-efficient, lazy) |

**String interning:** short string literals that look like identifiers are often interned (reused), which is why `"abc" is "abc"` can be `True` while `("a"*100) is ("a"*100)` is usually `False`.

## 3. Mapping Type

Associates keys with values; fast average-case O(1) lookup by key.

| Type | Mutability | Description |
|------|-----------|-------------|
| `dict` | Mutable | Key-value pairs (hash table), insertion-ordered since 3.7 |

## 4. Set Types

Unordered collections of unique, hashable items. Support mathematical set operations (union, intersection, difference).

| Type | Mutability | Description |
|------|-----------|-------------|
| `set` | Mutable | Unordered collection of unique items |
| `frozenset` | Immutable | Immutable, hashable version of `set` |

## 5. Binary Types

Handle raw binary data.

| Type | Mutability | Description |
|------|-----------|-------------|
| `bytes` | Immutable | Sequence of integers 0–255 |
| `bytearray` | Mutable | Mutable sequence of bytes |
| `memoryview` | — | Zero-copy view over a buffer-protocol object (e.g. `bytes`, `bytearray`, `array`) |

## 6. Boolean Type

- `bool` — subclass of `int`; `True == 1`, `False == 0`. Only two instances exist (`True`, `False`), so `is` comparisons on booleans are always safe.

## 7. Callable Types

Objects callable with `()`.

- User-defined functions
- Built-in functions/methods
- Classes (calling a class invokes `__new__`/`__init__`)
- Instances with a `__call__` method
- Generators and coroutines
- Lambda functions

## 8. Singleton / Sentinel Types

- `None` — absence of a value; the sole instance of `NoneType`
- `NotImplemented` — returned by a comparison/arithmetic dunder when the operation isn't supported for the given types (signals Python to try the reflected method)
- `Ellipsis` (`...`) — used in slicing, stub files, and type hints (`Callable[..., int]`)

## 9. Enumerations *(often missed but common in interviews)*

`enum` module types provide named, immutable, comparable constants.

| Type | Description |
|------|-------------|
| `Enum` | Base class for enumerations |
| `IntEnum` | Enum members that also behave as `int` |
| `Flag` / `IntFlag` | Support bitwise combination of members |
| `auto()` | Auto-assigns values |

```python
from enum import Enum

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

Color.RED is Color.RED   # True — enum members are singletons
```

## 10. Structured/Container Helpers (from `collections`)

Not "types" in the numeric-sequence-mapping sense, but frequently tested alongside data types:

| Type | Description |
|------|-------------|
| `namedtuple` | Tuple subclass with named fields |
| `defaultdict` | `dict` subclass with a default factory for missing keys |
| `OrderedDict` | `dict` subclass with explicit reordering methods (less needed since 3.7) |
| `Counter` | `dict` subclass for counting hashable items |
| `deque` | Double-ended queue, O(1) appends/pops from both ends |

## 11. Mutability Overview

| Category | Mutable | Immutable |
|----------|---------|-----------|
| Sequences | `list`, `bytearray` | `str`, `tuple`, `range`, `bytes` |
| Sets | `set` | `frozenset` |
| Mappings | `dict` | — |
| Numbers | — | `int`, `float`, `complex`, `bool`, `Decimal`, `Fraction` |
| Special | — | `None`, `NotImplemented`, `Ellipsis`, `Enum` members |

## 12. Type Checking & Conversion

```python
type(obj)                        # exact type
isinstance(obj, type_or_tuple)   # flexible — respects inheritance
issubclass(SubCls, BaseCls)      # class-level relationship check
```

**`type()` vs `isinstance()`:** `isinstance()` is preferred in most code because it accounts for subclassing (e.g. `isinstance(True, int)` → `True`), whereas `type(obj) == int` would reject a `bool` or any subclass.

### Common Conversions
`int()`, `float()`, `str()`, `list()`, `tuple()`, `dict()`, `set()`, `frozenset()`, `bool()`, `bytes()`, `bytearray()`, `complex()`

## 13. Key Characteristics by Category

### Ordered vs Unordered
- **Ordered:** `str`, `list`, `tuple`, `range`, `bytes`, `bytearray`, `dict` (3.7+, insertion order — but not "sorted")
- **Unordered:** `set`, `frozenset`

### Subscriptable (support indexing)
- Sequences: `str`, `list`, `tuple`, `range`, `bytes`, `bytearray`
- Mappings: `dict` (by key)
- Sets are **not** subscriptable (no defined order)

### Iterable
All sequences, sets, dicts (iterates over keys), and generators.

### Hashable
Only immutable, hash-stable types are hashable and usable as dict keys / set members: `int`, `float`, `str`, `tuple` (if all elements are hashable), `frozenset`, `bool`, `Enum` members, `None`. Lists, dicts, and sets are **unhashable**.

```python
hash((1, 2, 3))     # OK — tuple of hashables
hash([1, 2, 3])      # TypeError: unhashable type: 'list'
{[1, 2]: "x"}         # TypeError — can't use a list as a dict key
```

## 14. Notes & Gotchas
- **Duck Typing:** Python cares about what an object *can do* (its methods/protocol), not its declared type.
- **Everything is an object:** functions, classes, and modules all have a `type()`.
- **`is` vs `==`:** `is` checks identity (same object in memory); `==` checks value equality (calls `__eq__`). Never use `is` to compare numbers or strings for value equality except with singletons like `None`.
- **Mutable default arguments:** `def f(x=[]):` — the default list is created once and shared across calls, a classic bug source.
- **Type hints (3.5+):** `def func(x: int) -> str:` — not enforced at runtime, purely for tooling/clarity (`mypy`, IDEs).
- **`NotImplemented` vs `NotImplementedError`:** the former is a value returned from dunder methods; the latter is an exception raised in abstract methods. Frequently confused.
