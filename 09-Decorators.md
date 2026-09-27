### What is a Decorator?

At its simplest, **a decorator is a function that takes another function as an argument, adds some kind of functionality, and then returns another function.**

This is possible because Python treats functions as "first-class citizens," meaning you can:

  * Assign functions to variables.
  * Pass functions as arguments to other functions.
  * Return functions from other functions.

#### The "Manual" Way

Let's see it without any special syntax.

```python
# 1. This is our "decorator" function
def make_pretty(func):
    # 3. Define the 'wrapper' that adds new behavior
    def wrapper():
        print("I am being decorated!")
        # 4. Call the original function
        func()
    # 5. Return the new function
    return wrapper

# 2. This is the function we want to decorate
def ordinary_function():
    print("I am an ordinary function.")

# 6. Manually "decorate" it
decorated_func = make_pretty(ordinary_function)

# 7. Call the new, decorated function
decorated_func()
```

**Output:**
```
I am being decorated!
I am an ordinary function.
```

#### The `@` Syntax (Syntactic Sugar)

The `@` symbol is just a shortcut (syntactic sugar) for the manual process above.

```python
@make_pretty
def ordinary_function():
    print("I am an ordinary function.")

ordinary_function()
```

This code does *exactly* the same thing as the manual example. The `@make_pretty` line is equivalent to `ordinary_function = make_pretty(ordinary_function)`.

### Making Decorators Useful (Arguments & Return Values)

Most functions take arguments and return values — the decorator needs to handle both.

**Problem 1 — Arguments:** if `ordinary_function` takes an argument, `wrapper` needs to accept it too. Use `*args`/`**kwargs` to accept *any* signature.

**Problem 2 — Return Values:** if `ordinary_function` returns a value, `wrapper` must capture and return it — otherwise the decorated function silently returns `None`.

**Problem 3 — Metadata:** a decorated function loses its original identity — `ordinary_function.__name__` becomes `'wrapper'`. Fix with **`functools.wraps`**.

#### The "Proper" Decorator Template

```python
import time
from functools import wraps

def timer_decorator(func):
    """A decorator that prints the time a function takes to run."""

    @wraps(func)                      # 3. preserve function metadata
    def wrapper(*args, **kwargs):     # 1. accept any arguments
        start_time = time.perf_counter()
        value = func(*args, **kwargs)  # 2. capture and return the value
        end_time = time.perf_counter()
        run_time = end_time - start_time
        print(f"Finished {func.__name__!r} in {run_time:.4f} secs")
        return value
    return wrapper

@timer_decorator
def complex_calculation(num1, num2):
    """A function that 'sleeps' to simulate work."""
    time.sleep(1)
    return num1 + num2

result = complex_calculation(10, 5)
print(f"Function result: {result}")
print(f"Function name: {complex_calculation.__name__}")       # correct, thanks to @wraps
print(f"Function docstring: {complex_calculation.__doc__}")
```

**Output:**
```
Finished 'complex_calculation' in 1.0005 secs
Function result: 15
Function name: complex_calculation
Function docstring: A function that 'sleeps' to simulate work.
```

**What `@wraps` actually does under the hood:** it copies `__name__`, `__doc__`, `__module__`, and `__dict__` from the original function onto the wrapper, and also sets `wrapper.__wrapped__ = func` — a direct reference to the original, unwrapped function. This is what lets tools like `inspect.signature()` and `help()` see through the wrapper to the real signature, and it's how you'd manually "unwrap" a decorated function if you ever needed the original back.

### Decorators with Arguments

To pass arguments *to the decorator itself* (e.g., `@repeat(num_times=3)`), you need **one extra layer**:

1. An outer function that accepts the decorator's arguments.
2. It returns the *actual decorator*.
3. That decorator returns the *wrapper*.

It's a "function factory" that builds a decorator.

```python
from functools import wraps

def repeat(num_times):                  # 1. outer function accepts decorator args
    def decorator_repeat(func):          # 2. the actual decorator
        @wraps(func)
        def wrapper(*args, **kwargs):     # 3. the wrapper, as before
            results = []
            for _ in range(num_times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator_repeat

@repeat(num_times=3)
def greet(name):
    print(f"Hello, {name}!")
    return name

greet("Alice")
```

**Output:**
```
Hello, Alice!
Hello, Alice!
Hello, Alice!
```

**How it works:** `@repeat(num_times=3)` runs first and returns `decorator_repeat`; Python then effectively applies `@decorator_repeat` to `greet`.

### Stacking Multiple Decorators

Decorators can be chained — a very common "predict the output" interview question, since **order matters**.

```python
def bold(func):
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold
@italic
def greet():
    return "Hello"

print(greet())   # <b><i>Hello</i></b>
```

**Reading order:** decorators are **applied bottom-up but execute outside-in**.
- `@bold` / `@italic` stacked on `greet` is equivalent to `greet = bold(italic(greet))`.
- So `italic` wraps `greet` first (closest to the function), and `bold` wraps the *result* of that.
- At **call time**, execution goes the other way: `bold`'s wrapper runs first (prints/does its "before" logic first), then calls into `italic`'s wrapper, which calls the real `greet`.

Swapping the order (`@italic` above `@bold`) changes the output to `<i><b>Hello</b></i>` — this exact swap is a classic interview trick question.

### Decorating Methods (Instance Methods)

A decorator applied to a method needs its wrapper to also accept `self` (via `*args` typically handles this transparently, since `self` just becomes `args[0]`):

```python
def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):   # self flows through args automatically
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

class Greeter:
    @log_call
    def greet(self, name):
        return f"Hi, {name}"
```

Using `*args, **kwargs` (rather than writing `def wrapper(self, ...)` explicitly) is exactly why the same decorator works transparently on both plain functions and methods — the wrapper doesn't need to know or care whether the first argument is `self`.

### Class Decorators (Decorating a Class Itself)

Distinct from a *class-based decorator* (below, which uses a class to build a decorator for functions) — this is a decorator applied directly to a **class definition**, to modify or register the class itself.

```python
def add_greeting(cls):
    cls.greeting = "Hello from a class decorator!"
    return cls

@add_greeting
class Widget:
    pass

print(Widget.greeting)   # "Hello from a class decorator!"
```

This is exactly how `@dataclass` works — it takes a plain class, inspects its type-annotated attributes, and injects generated methods (`__init__`, `__repr__`, `__eq__`) directly onto the class before returning it.

### Class-Based Decorators (Using a Class to Decorate a Function)

Most useful when you need to **maintain state** between calls. Implemented with two methods:

  * `__init__(self, func)` — receives the function to decorate (runs once, at decoration time).
  * `__call__(self, *args, **kwargs)` — makes the instance callable; runs on *every call* of the decorated function.

```python
from functools import update_wrapper

class CountCalls:
    def __init__(self, func):
        update_wrapper(self, func)   # preserve metadata (the class-based equivalent of @wraps)
        self.func = func
        self.num_calls = 0            # persistent state

    def __call__(self, *args, **kwargs):
        self.num_calls += 1
        print(f"Call {self.num_calls} of {self.func.__name__!r}")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    print("Hello!")

say_hello()
say_hello()
say_hello()
```

**Output:**
```
Call 1 of 'say_hello'
Hello!
Call 2 of 'say_hello'
Hello!
Call 3 of 'say_hello'
Hello!
```

`self.num_calls` is the state that persists between calls — cleaner than a `global` variable, and each differently-decorated function gets its own independent counter automatically (each `@CountCalls` application creates a separate instance).

### A Closer Look: `functools.lru_cache`

Python's built-in memoization decorator, referenced often but worth seeing directly:

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

fib(30)          # fast — cached subcalls avoid exponential recomputation
print(fib.cache_info())   # CacheInfo(hits=28, misses=31, maxsize=128, currsize=31)
```

- **`maxsize`** — how many distinct argument combinations to remember; `maxsize=None` means unbounded cache.
- **Arguments must be hashable** — calling `fib([1,2])` with a list raises `TypeError`, since the cache is keyed by the arguments.
- **`.cache_info()`** and **`.cache_clear()`** are added automatically — useful for debugging cache effectiveness or resetting state (e.g., between test cases).
- Turns naive recursive Fibonacci from exponential to linear time — a frequent live-coding follow-up after writing the recursive version.

### Common Use Cases

  * **Logging** — as in `timer_decorator`, for tracking calls, arguments, and return values.
  * **Authentication & Authorization** — `@login_required` or `@permission_required` in frameworks like Flask/Django.
  * **Caching / Memoization** — `@functools.lru_cache`, shown above.
  * **Rate Limiting** — restricting how often a function (like an API endpoint) can be called.
  * **Registering Functions** — frameworks like `pytest` (`@pytest.fixture`) use decorators to register functions in a central registry.
  * **Built-in decorators you already use constantly** — `@property`, `@staticmethod`, and `@classmethod` are themselves decorators (in fact, descriptor-producing ones); worth connecting this material back to those, since the mechanism is identical — a callable that wraps another callable and returns something new.

## Notes & Gotchas

- **"Applied bottom-up, executed outside-in"** is the precise way to describe stacked decorator order — memorize this phrasing, since "predict the output" questions with 2+ stacked decorators are extremely common.
- **Always use `@wraps(func)`** — forgetting it is a common code-review flag; without it, `__name__`, `__doc__`, and introspection tools all report the wrapper's identity instead of the original function's.
- **A decorator without `*args, **kwargs` in its wrapper only works on functions matching that exact fixed signature** — this is why nearly every general-purpose decorator template uses `*args, **kwargs` rather than hardcoding parameters.
- **`lru_cache` requires hashable arguments** — a very common "why did my cached function break" bug when someone passes a list or dict.
- **A class-based decorator's `__init__` runs once, at decoration time; `__call__` runs on every invocation** — mixing these up is a common source of "why is my counter/state wrong" bugs (e.g., accidentally resetting state inside `__call__`).
- **Class decorators (on a `class` statement) vs. class-based decorators (a class implementing `__call__` to decorate a function) are easy to conflate by name** — but they're solving different problems: one modifies/wraps a class, the other uses a class as the *mechanism* for decorating a function.
