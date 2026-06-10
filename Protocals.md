# Python Protocols - Structural Subtyping Guide

In Python, a **Protocol** is an explicit way to define an interface or behavioral contract using **structural subtyping** (often called static duck typing). Introduced in Python 3.8 via the `typing.Protocol` class, protocols let you specify the exact methods and attributes an object must have without requiring classes to explicitly inherit from it.

If a class implements the exact names and type signatures defined in a protocol, Python's static type checkers (like mypy or Pyright) automatically accept it as a valid subtype.

## Basic Implementation Pattern

To implement a protocol:
1. Define a class that inherits from `typing.Protocol`
2. List the expected methods or attributes using an ellipsis (`...`) as a placeholder
3. Use that protocol as a type hint

### 1. Define the Protocol

```python
from typing import Protocol

class Document(Protocol):
    title: str  # Expected attribute

    def extract_text(self) -> str:
        """Expected method"""
        ...
```

### 2. Create Conforming Classes (No Inheritance Needed)

You do not need to inherit from `Document`. You only need to match its structure.

```python
class PDFBook:
    def __init__(self, title: str, content: str):
        self.title = title
        self.content = content

    def extract_text(self) -> str:
        return f"PDF Content: {self.content}"


class WordFile:
    def __init__(self, title: str, text: str):
        self.title = title
        self.text = text

    def extract_text(self) -> str:
        return f"Word Content: {self.text}"
```

### 3. Use the Protocol as a Type Hint

```python
def print_summary(doc: Document) -> None:
    # Type checkers verify that 'title' and 'extract_text' exist
    print(f"Reading: {doc.title}")
    print(doc.extract_text())

# Both work flawlessly with static type checkers
print_summary(PDFBook("Python Guide", "Hello World"))
print_summary(WordFile("Notes", "Some text"))
```

## Advanced Implementation Rules

### Defining Properties (Read-Only vs. Writable Attributes)

By default, protocol variables are assumed to be writable. If an attribute should only be readable, define it using Python's `@property` decorator.

```python
from typing import Protocol

class UserProfile(Protocol):
    @property
    def username(self) -> str: ...  # Read-only attribute
    
    email: str  # Writable attribute (must support read AND write)
```

### Generic Protocols

Protocols can be made generic using `typing.Generic` or Python 3.12+'s native generic syntax. This is highly useful for abstracting data structures or repositories.

```python
from typing import Protocol, TypeVar

T = TypeVar('T')

class Repository(Protocol[T]):
    def get_by_id(self, item_id: int) -> T: ...
    def save(self, item: T) -> None: ...
```

### Subclassing and Extending Protocols

You can extend an existing protocol to create a more specific interface. A class must satisfy both protocols to conform to the extended version.

```python
from typing import Protocol

class Reader(Protocol):
    def read(self) -> bytes: ...

class ReadWriter(Reader, Protocol):  # Inherits read(), adds write()
    def write(self, data: bytes) -> None: ...
```

## Key Features & Runtime Mechanisms

- **Implicit Compliance**: Classes conform to a protocol simply by having matching methods and attributes.
- **Static Verification**: They are primarily meant for static analysis tools like mypy to catch bugs before code runs.
- **Runtime Type Checking (@runtime_checkable)**: By default, protocols vanish at runtime and cannot be used with `isinstance()`. If you want to use them for runtime checks, decorate the protocol with `@runtime_checkable`.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Greeter(Protocol):
    def greet(self) -> str: ...

class Human:
    def greet(self) -> str:
        return "Hello!"

obj = Human()
print(isinstance(obj, Greeter))  # Returns True
```

## Python's Built-In Protocols

Python uses structural subtyping internally. The `collections.abc` and `typing` modules provide many built-in protocols that you use daily without realizing it:

- **Iterable**: Requires `__iter__`
- **Iterator**: Requires `__next__` and `__iter__`
- **Sized**: Requires `__len__`
- **Container**: Requires `__contains__`
- **ContextManager**: Requires `__enter__` and `__exit__`

```python
from typing import Sized

def print_length(item: Sized) -> None:
    print(len(item))  # Works for lists, strings, dicts, etc.
```

## Protocol vs. Abstract Base Class (ABC)

| Feature | Protocol (typing.Protocol) | Abstract Base Class (abc.ABC) |
|---------|----------------------------|-------------------------------|
| Typing Style | Structural (Looks like a duck) | Nominal (Named inheritance) |
| Coupling | Decoupled: Classes don't know the protocol exists. | Tightly Coupled: Classes must explicitly inherit. |
| Implementation | Cannot contain concrete method code. | Can contain concrete/shared method logic. |
| Best Case | Third-party libraries, flexible APIs, modern type hints. | Internal frameworks, core shared logic hierarchies. |

## When to Use Protocols

✅ **Use Protocols when:**
- Working with third-party libraries or external code
- Building flexible APIs that accept multiple types
- Defining interfaces for modern, type-safe Python code
- You want decoupled, composable type contracts

❌ **Avoid Protocols when:**
- You need shared implementation logic (use ABC instead)
- Working with legacy code that doesn't support type hints
- You require runtime enforcement without `@runtime_checkable`
