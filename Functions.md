## 1. The Anatomy of a Function

Every function in Python is an instance of the `function` class. When you define one, Python creates an object in memory and assigns it to the name you provided.

```python
def greet(name: str) -> str:
    """Returns a greeting string."""
    return f"Hello, {name}!"

# Functions are objects!
print(greet.__doc__)    # Access the docstring
print(greet.__name__)   # Access the function name

```

### Argument Types

Python offers incredible flexibility in how you pass data:

* **Positional:** `func(1, 2)`
* **Keyword:** `func(a=1, b=2)`
* **Default Values:** `def func(a=10):` (Warning: Never use mutable defaults like `[]` or `{}`).
* **Variadic (\*args):** Collects extra positional arguments into a tuple.
* **Keyword Variadic (\*\*kwargs):** Collects extra keyword arguments into a dictionary.

## 2. Advanced Parameter Control

Modern Python (3.8+) allows you to strictly enforce how arguments are passed using special delimiters:

```python
def complex_func(pos_only, /, standard, *, kw_only):
    pass

# /  -> Everything before this MUST be positional.
# *  -> Everything after this MUST be keyword-only.

```

## 3. Scoping and the LEGB Rule

When you reference a variable inside a function, Python searches in a specific order:

1. **L**ocal: Inside the current function.
2. **E**nclosing: Inside any nested "parent" functions (closures).
3. **G**lobal: At the top level of the module.
4. **B**uilt-in: Reserved words like `len`, `range`, or `print`.

## 4. Introspection and the `inspect` Module

If you are building frameworks or debugging complex systems, the `inspect` module lets you look "inside" a function object to see its source code, arguments, and live stack frames.

```python
import inspect

def my_func(a, b=5): pass

sig = inspect.signature(my_func)
print(sig.parameters) # OrderedDict([('a', <Parameter "a">), ('b', <Parameter "b=5">)])
```
