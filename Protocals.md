# Python Protocols - Structural Subtyping Guide

In Python, a **Protocol** is an explicit way to define an interface or behavioral contract using **structural subtyping** (often called static duck typing). Introduced in Python 3.8 via the `typing.Protocol` class, protocols let you specify the exact methods and attributes an object must have without requiring classes to explicitly inherit from it.

If a class implements the exact names and type signatures defined in a protocol, Python's static type checkers (like mypy or Pyright) automatically accept it as a valid subtype.

## Table of Contents
1. [Basic Implementation Pattern](#basic-implementation-pattern)
2. [Advanced Implementation Rules](#advanced-implementation-rules)
3. [Key Features & Runtime Mechanisms](#key-features--runtime-mechanisms)
4. [Python's Built-In Protocols](#pythons-built-in-protocols)
5. [Protocol vs. Abstract Base Class (ABC)](#protocol-vs-abstract-base-class-abc)
6. [Callable Protocols](#callable-protocols)
7. [Protocol Composition](#protocol-composition)
8. [Structural vs. Nominal Subtyping](#structural-vs-nominal-subtyping)
9. [Common Pitfalls & Best Practices](#common-pitfalls--best-practices)
10. [Real-World Examples](#real-world-examples)
11. [When to Use Protocols](#when-to-use-protocols)

---

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

---

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

---

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

**Note:** `@runtime_checkable` only checks for the presence of methods/attributes, not their signatures at runtime.

---

## Python's Built-In Protocols

Python uses structural subtyping internally. The `collections.abc` and `typing` modules provide many built-in protocols that you use daily without realizing it:

- **Iterable**: Requires `__iter__`
- **Iterator**: Requires `__next__` and `__iter__`
- **Sized**: Requires `__len__`
- **Container**: Requires `__contains__`
- **ContextManager**: Requires `__enter__` and `__exit__`
- **Callable**: Represents callable objects
- **Hashable**: Objects that can be used as dictionary keys
- **Reversible**: Objects that support `__reversed__`

```python
from typing import Sized

def print_length(item: Sized) -> None:
    print(len(item))  # Works for lists, strings, dicts, etc.

print_length([1, 2, 3])       # ✓ Works
print_length("hello")         # ✓ Works
print_length({"a": 1})        # ✓ Works
```

---

## Protocol vs. Abstract Base Class (ABC)

| Feature | Protocol (typing.Protocol) | Abstract Base Class (abc.ABC) |
|---------|----------------------------|-------------------------------|
| Typing Style | Structural (Looks like a duck) | Nominal (Named inheritance) |
| Coupling | Decoupled: Classes don't know the protocol exists. | Tightly Coupled: Classes must explicitly inherit. |
| Implementation | Cannot contain concrete method code. | Can contain concrete/shared method logic. |
| Best Case | Third-party libraries, flexible APIs, modern type hints. | Internal frameworks, core shared logic hierarchies. |

---

## Callable Protocols

Protocols are especially useful for defining function signatures. Use `Callable` for simple functions or create custom callable protocols for complex behavior.

```python
from typing import Protocol, Callable

# Simple approach using Callable
def execute_operation(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

# Protocol approach for more complex callable behavior
class Transformer(Protocol):
    """Protocol for objects that transform data"""
    def transform(self, data: str) -> str: ...
    def validate(self) -> bool: ...

class UppercaseTransformer:
    def transform(self, data: str) -> str:
        return data.upper()
    
    def validate(self) -> bool:
        return True

def apply_transformer(transformer: Transformer, text: str) -> str:
    if transformer.validate():
        return transformer.transform(text)
    return text
```

---

## Protocol Composition

Combine multiple protocols to create more specific requirements.

```python
from typing import Protocol

class Readable(Protocol):
    def read(self) -> str: ...

class Writable(Protocol):
    def write(self, data: str) -> None: ...

class Closeable(Protocol):
    def close(self) -> None: ...

# Composed protocol
class FileHandler(Readable, Writable, Closeable, Protocol):
    """Combines all three protocols"""
    pass

class TextFile:
    def read(self) -> str:
        return "content"
    
    def write(self, data: str) -> None:
        pass
    
    def close(self) -> None:
        pass

# Type checker accepts TextFile as FileHandler
def process_file(handler: FileHandler) -> None:
    content = handler.read()
    handler.write(content.upper())
    handler.close()
```

---

## Structural vs. Nominal Subtyping

### Nominal Subtyping (Traditional OOP)
Requires explicit inheritance relationship.

```python
class Animal:
    def speak(self) -> str:
        raise NotImplementedError

class Dog(Animal):  # MUST explicitly inherit from Animal
    def speak(self) -> str:
        return "Woof!"

def make_sound(animal: Animal) -> None:
    print(animal.speak())

dog = Dog()
make_sound(dog)  # ✓ Works because Dog inherits from Animal
```

### Structural Subtyping (Protocols)
Only requires matching structure, no explicit relationship.

```python
from typing import Protocol

class Speaker(Protocol):
    def speak(self) -> str: ...

class Dog:  # NO inheritance from Speaker
    def speak(self) -> str:
        return "Woof!"

def make_sound(speaker: Speaker) -> None:
    print(speaker.speak())

dog = Dog()
make_sound(dog)  # ✓ Works! Type checker accepts it because structure matches
```

---

## Common Pitfalls & Best Practices

### ✅ Best Practices

1. **Use Protocols for public APIs** - Allow flexibility in implementations
   ```python
   def save_data(writer: Writer) -> None:
       writer.write(b"data")  # Works with any object that has write()
   ```

2. **Prefer small, focused protocols** - Single responsibility principle
   ```python
   # Good
   class Readable(Protocol):
       def read(self) -> bytes: ...
   
   class Writable(Protocol):
       def write(self, data: bytes) -> None: ...
   ```

3. **Use @runtime_checkable sparingly** - Primarily for static type checking
   ```python
   from typing import Protocol, runtime_checkable
   
   @runtime_checkable
   class Drawable(Protocol):
       def draw(self) -> None: ...
   ```

4. **Document protocol requirements clearly**
   ```python
   class Serializable(Protocol):
       """
       Objects that can be converted to bytes.
       
       Methods:
           to_bytes() -> bytes: Convert to binary representation
       """
       def to_bytes(self) -> bytes: ...
   ```

### ❌ Common Pitfalls

1. **Forgetting @runtime_checkable for isinstance() checks**
   ```python
   # ✗ Won't work at runtime without @runtime_checkable
   if isinstance(obj, MyProtocol):
       pass
   ```

2. **Creating overly complex protocols**
   ```python
   # ✗ Too many unrelated methods
   class BadProtocol(Protocol):
       def read(self) -> str: ...
       def write(self, data: str) -> None: ...
       def delete(self) -> None: ...
       def compress(self) -> bytes: ...
   ```

3. **Using protocols when ABC is more appropriate**
   ```python
   # ✗ If you need shared implementation
   class BaseClass(Protocol):
       def process(self) -> None:
           # This won't work in Protocol!
           self.validate()
   ```

4. **Mixing structural and nominal typing inconsistently**
   ```python
   # ✗ Confusing pattern
   class MyClass(SomeProtocol):  # Don't explicitly inherit from Protocol classes
       pass
   ```

---

## Real-World Examples

### Example 1: Plugin System

```python
from typing import Protocol

class Plugin(Protocol):
    """Interface for extensible plugins"""
    name: str
    version: str
    
    def initialize(self) -> None: ...
    def execute(self, data: dict) -> dict: ...
    def shutdown(self) -> None: ...

class ImageProcessorPlugin:
    name = "ImageProcessor"
    version = "1.0"
    
    def initialize(self) -> None:
        print("Loading PIL...")
    
    def execute(self, data: dict) -> dict:
        # Process image
        return {"result": "processed"}
    
    def shutdown(self) -> None:
        print("Cleaning up...")

def load_plugin(plugin: Plugin) -> None:
    plugin.initialize()
    result = plugin.execute({})
    plugin.shutdown()
```

### Example 2: Data Serialization

```python
from typing import Protocol, Any

class Serializer(Protocol):
    """Any object that can serialize data"""
    def serialize(self, data: Any) -> str: ...
    def deserialize(self, raw: str) -> Any: ...

import json

class JSONSerializer:
    def serialize(self, data: Any) -> str:
        return json.dumps(data)
    
    def deserialize(self, raw: str) -> Any:
        return json.loads(raw)

import pickle

class PickleSerializer:
    def serialize(self, data: Any) -> str:
        return pickle.dumps(data).decode()
    
    def deserialize(self, raw: str) -> Any:
        return pickle.loads(raw.encode())

def save_to_file(filename: str, data: dict, serializer: Serializer) -> None:
    """Works with any serializer that matches the Serializer protocol"""
    with open(filename, 'w') as f:
        f.write(serializer.serialize(data))

# Both JSONSerializer and PickleSerializer work seamlessly
save_to_file("data.json", {"key": "value"}, JSONSerializer())
save_to_file("data.pkl", {"key": "value"}, PickleSerializer())
```

### Example 3: Logger Abstraction

```python
from typing import Protocol
from abc import ABC, abstractmethod

class Logger(Protocol):
    """Protocol for logging implementations"""
    def debug(self, msg: str) -> None: ...
    def info(self, msg: str) -> None: ...
    def warning(self, msg: str) -> None: ...
    def error(self, msg: str) -> None: ...

class ConsoleLogger:
    def debug(self, msg: str) -> None:
        print(f"[DEBUG] {msg}")
    
    def info(self, msg: str) -> None:
        print(f"[INFO] {msg}")
    
    def warning(self, msg: str) -> None:
        print(f"[WARNING] {msg}")
    
    def error(self, msg: str) -> None:
        print(f"[ERROR] {msg}")

class FileLogger:
    def __init__(self, filename: str):
        self.file = filename
    
    def debug(self, msg: str) -> None:
        with open(self.file, 'a') as f:
            f.write(f"[DEBUG] {msg}\n")
    
    def info(self, msg: str) -> None:
        with open(self.file, 'a') as f:
            f.write(f"[INFO] {msg}\n")
    
    def warning(self, msg: str) -> None:
        with open(self.file, 'a') as f:
            f.write(f"[WARNING] {msg}\n")
    
    def error(self, msg: str) -> None:
        with open(self.file, 'a') as f:
            f.write(f"[ERROR] {msg}\n")

def run_application(logger: Logger) -> None:
    logger.info("Starting application...")
    logger.debug("Debug mode enabled")
    logger.warning("Low memory")
    logger.error("Connection failed")

# Use either logger without changing application code
run_application(ConsoleLogger())
run_application(FileLogger("app.log"))
```

---

## When to Use Protocols

### ✅ **Use Protocols when:**
- Working with third-party libraries or external code you can't modify
- Building flexible APIs that accept multiple types
- Defining interfaces for modern, type-safe Python code
- You want decoupled, composable type contracts
- Creating plugin systems or extensible architectures
- Supporting duck typing with static type checking

### ❌ **Avoid Protocols when:**
- You need shared implementation logic (use ABC instead)
- Working with legacy code that doesn't support type hints
- You require strict runtime enforcement of contracts
- Creating internal class hierarchies with `isinstance()` checks
- Simplicity matters more than flexibility

---

## Summary

Python Protocols provide a powerful way to define interfaces using structural typing, allowing for flexible and decoupled code while maintaining strong static type checking. They're especially valuable in modern Python for building extensible systems, working with third-party libraries, and creating APIs that accept any object with the right structure.
