### Numeric Types
These represent numbers and support arithmetic operations.

- **Integral Types**: `int`, `bool` (Boolean is a subclass of `int`)
- **Non-Integral Types**: `float`, `complex`, `Decimal`, `Fraction`

### Sequence Types
Ordered collections of items.

| Type         | Mutability | Description                        |
|--------------|------------|------------------------------------|
| `str`        | Immutable  | Text data                          |
| `list`       | Mutable    | Ordered collection of items        |
| `tuple`      | Immutable  | Ordered, fixed-size collection     |
| `range`      | Immutable  | Sequence of numbers                |

### Mapping Type
Associates keys with values.

- `dict`: Mutable key-value pairs

### Set Types
Unordered collections of unique items.

| Type         | Mutability |
|--------------|------------|
| `set`        | Mutable    |
| `frozenset`  | Immutable  |

### Binary Types
Used for handling binary data.

- `bytes`: Immutable sequence of bytes
- `bytearray`: Mutable sequence of bytes
- `memoryview`: Memory-efficient view of binary data

### Boolean Type
- `bool`: Subclass of `int`, values are `True` or `False`

### Callable Types
Objects that can be called like functions.

- User-defined functions
- Built-in functions/methods
- Classes and class instances with `__call__`
- Generators

### Singleton Types
Special unique values.

- `None`
- `NotImplemented`
- `Ellipsis` (`...`)

### Mutability Overview

| Category     | Mutable Types        | Immutable Types               |
|--------------|----------------------|-------------------------------|
| Sequences    | `list`               | `str`, `tuple`, `range`       |
| Sets         | `set`                | `frozenset`                   |
| Mappings     | `dict`               | —                             |
| Numbers      | —                    | `int`, `float`, `complex`     |
