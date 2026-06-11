Python's data types can be classified into several categories based on their properties and use cases.
## Numeric Types
These represent numbers and support arithmetic operations.
### Integral Types
- `int`: Arbitrary-precision integers (no overflow)
- `bool`: Boolean values (`True` or `False`), subclass of `int`
### Non-Integral Types
- `float`: Floating-point numbers (IEEE 754 double precision)
- `complex`: Complex numbers with real and imaginary parts
- `Decimal`: High-precision decimal arithmetic (from `decimal` module)
- `Fraction`: Rational numbers with exact arithmetic (from `fractions` module)
## Sequence Types
Ordered collections of items. Items can be accessed by index.
| Type | Mutability | Description |
|------|-----------|-------------|
| `str` | Immutable | Text/Unicode strings |
| `list` | Mutable | Ordered, dynamic collection of items |
| `tuple` | Immutable | Ordered, fixed-size collection |
| `range` | Immutable | Sequence of numbers (memory-efficient) |
## Mapping Type
Associates keys with values. Provides fast lookup by key.
| Type | Mutability | Description |
|------|-----------|-------------|
| `dict` | Mutable | Key-value pairs (hash table) |
## Set Types
Unordered collections of unique items. Support mathematical set operations.
| Type | Mutability | Description |
|------|-----------|-------------|
| `set` | Mutable | Unordered collection of unique items |
| `frozenset` | Immutable | Immutable version of `set` |
## Binary Types
Used for handling raw binary data.
- `bytes`: Immutable sequence of bytes (0-255)
- `bytearray`: Mutable sequence of bytes
- `memoryview`: Memory-efficient view of binary data (read/write access)
## Boolean Type
- `bool`: Subclass of `int` with values `True` (1) or `False` (0)
## Callable Types
Objects that can be called like functions.
- User-defined functions
- Built-in functions/methods
- Classes and class instances with `__call__` method
- Generators and coroutines
- Lambda functions
## Singleton Types
Special unique values representing specific conditions.
- `None`: Represents the absence of a value
- `NotImplemented`: Indicates an operation is not implemented for the given types
- `Ellipsis` (`...`): Used in slicing and function signatures
## Mutability Overview
| Category | Mutable Types | Immutable Types |
|----------|---------------|-----------------|
| Sequences | `list`, `bytearray` | `str`, `tuple`, `range`, `bytes` |
| Sets | `set` | `frozenset` |
| Mappings | `dict` | — |
| Numbers | — | `int`, `float`, `complex`, `bool` |
| Special | — | `None`, `NotImplemented`, `Ellipsis` |
## Type Checking & Conversion
### Checking Types
```python
type(obj)           # Returns the exact type
isinstance(obj, type_or_tuple)  # More flexible type checking
```
### Common Type Conversions
- `int()`, `float()`, `str()`, `list()`, `tuple()`, `dict()`, `set()`, `bool()`
- `bytes()`, `bytearray()`, `complex()`
## Key Characteristics by Category
### Ordered vs Unordered
- **Ordered**: `str`, `list`, `tuple`, `range`, `bytes`, `bytearray`
- **Unordered**: `set`, `frozenset`, `dict` (though dicts maintain insertion order in Python 3.7+)
### Subscriptable (Support Indexing)
- Sequences: `str`, `list`, `tuple`, `range`, `bytes`, `bytearray`
- Mappings: `dict` (indexed by key)
### Iterable
- All sequences, sets, dicts, and generators are iterable
## Notes
- **Duck Typing**: Python focuses on what an object can do rather than its type
- **Hashable Types**: Only immutable types are hashable and can be used as dict keys or in sets
- **Type Hints**: Python 3.5+ supports optional type hints for better code clarity (e.g., `def func(x: int) -> str:`)
