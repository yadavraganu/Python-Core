## 1. The Exception Hierarchy

All exceptions in Python are objects, and every exception class inherits — directly or indirectly — from `BaseException`.

```
BaseException
 ├── SystemExit
 ├── KeyboardInterrupt
 ├── GeneratorExit
 └── Exception                 # <- what you should almost always catch/subclass
      ├── ArithmeticError
      │    ├── ZeroDivisionError
      │    └── OverflowError
      ├── LookupError
      │    ├── IndexError
      │    └── KeyError
      ├── ValueError
      ├── TypeError
      ├── AttributeError
      ├── NameError
      │    └── UnboundLocalError
      ├── OSError (aka IOError)
      │    ├── FileNotFoundError
      │    ├── PermissionError
      │    └── TimeoutError
      ├── RuntimeError
      │    ├── RecursionError
      │    └── NotImplementedError
      └── StopIteration
```

**Note:** `SystemExit`, `KeyboardInterrupt`, and `GeneratorExit` inherit from `BaseException`, **not** `Exception`, specifically so that a broad `except Exception:` won't accidentally swallow a user's Ctrl+C or a clean `sys.exit()` call. This is the main reason a **bare `except:`** (which catches `BaseException`) is considered bad practice — it also catches these control-flow signals.

## 2. Basic try/except/else/finally

```python
try:
    result = 10 / x
except ZeroDivisionError:
    print("Can't divide by zero")
else:
    # Runs only if the try block raised NOTHING
    print(f"Result: {result}")
finally:
    # Always runs — success, exception, or even a return/break in try
    print("Cleanup runs regardless")
```

- **`except`** — handles a specific error.
- **`else`** — runs only when no exception occurred; keeps the "happy path" separate from error handling and from the code that might raise.
- **`finally`** — always runs, used for guaranteed cleanup (closing files, releasing locks), even if the `try` block returns or re-raises.

### Catching Multiple / Specific Exceptions

```python
try:
    risky_call()
except (ValueError, TypeError) as e:      # tuple = catch either
    print(f"Bad input: {e}")
except KeyError as e:
    print(f"Missing key: {e}")
except Exception as e:                     # generic fallback — keep it last, and narrow
    print(f"Unexpected: {e}")
```

Order matters: Python checks `except` clauses top to bottom and uses the **first match** — so put more specific exceptions before more general ones (a broad one listed first would shadow the specific ones below it).

## 3. Raising Exceptions

```python
def set_age(age):
    if age < 0:
        raise ValueError(f"age cannot be negative, got {age}")
```

### Re-raising

```python
try:
    do_something()
except ValueError:
    log.warning("bad value, re-raising")
    raise            # re-raises the SAME exception with its original traceback intact
```

### Exception Chaining — `raise ... from ...`

When you raise a new exception while handling another, Python automatically records the original as `__context__`. Using `raise X from Y` makes that link explicit (`__cause__`) and produces a clearer traceback ("The above exception was the direct cause of...").

```python
try:
    parse(data)
except ValueError as e:
    raise RuntimeError("failed to process record") from e
```

To deliberately hide the chain (e.g., in a public API where the internal cause is noise):
```python
raise RuntimeError("failed") from None
```

## 4. Custom Exceptions

Define your own exceptions by subclassing `Exception` (never `BaseException` directly, unless you specifically want to bypass broad `except Exception` handlers — rare and usually wrong).

```python
class InsufficientFundsError(Exception):
    """Raised when a withdrawal exceeds the available balance."""
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(
            f"Cannot withdraw {amount}; balance is only {balance}"
        )

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(balance, amount)
    return balance - amount
```

Custom exception **hierarchies** are common in larger codebases so callers can catch broadly or narrowly:

```python
class AppError(Exception):          # base for all app-specific errors
    pass

class ValidationError(AppError):
    pass

class NotFoundError(AppError):
    pass

# caller can do: except AppError:  to catch anything from this app
```

## 5. `finally`, Context Managers, and Resource Cleanup

`finally` guarantees cleanup, but the idiomatic tool for resource management is the **context manager** (`with` statement), which calls cleanup automatically via `__exit__` — even on an exception.

```python
# Manual approach
f = open("file.txt")
try:
    data = f.read()
finally:
    f.close()

# Idiomatic approach — same guarantee, less code
with open("file.txt") as f:
    data = f.read()
```

**How `with` interacts with exceptions:** `__exit__(exc_type, exc_value, traceback)` is called even if the block raises. If `__exit__` returns a truthy value, the exception is **suppressed**; otherwise it propagates after cleanup runs.

```python
class Suppressor:
    def __enter__(self): return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        return True   # swallows any exception raised inside the block

with Suppressor():
    raise ValueError("this will be silently suppressed")
```

## 6. EAFP vs. LBYL

Two philosophies for handling conditions that might fail:

- **LBYL** ("Look Before You Leap") — check first, then act:
  ```python
  if key in d:
      value = d[key]
  ```
- **EAFP** ("Easier to Ask Forgiveness than Permission") — try it, handle failure:
  ```python
  try:
      value = d[key]
  except KeyError:
      value = default
  ```

**Python idiom strongly favors EAFP** — it avoids a race condition between the check and the action (relevant for files/threads), and it's often faster when failures are rare, since exceptions have near-zero cost on the success path. This is a frequent "why does Python prefer try/except over if-checks" interview question.

## 7. `assert` Statements

```python
def divide(a, b):
    assert b != 0, "b must not be zero"
    return a / b
```

- Raises `AssertionError` if the condition is falsy.
- **Not for validating user input or production-critical invariants** — `assert` statements are stripped out entirely when Python is run with the `-O` (optimize) flag, so they must never be relied on for security or required business logic.

## 8. Inspecting and Logging Exceptions

```python
import traceback, logging

try:
    risky_call()
except Exception:
    traceback.print_exc()                 # prints full traceback to stderr
    tb_str = traceback.format_exc()        # same, as a string
    logging.exception("risky_call failed")  # logs message + traceback automatically
```

`logging.exception(...)` must be called from inside an `except` block — it automatically attaches the current exception's traceback, and is generally preferred over `print()` for anything beyond a quick script.

## 9. `StopIteration` and Generators

`StopIteration` is a normal, expected exception — it's how iterators signal "no more items," raised internally by `next()` and caught automatically by `for` loops.

```python
it = iter([1, 2])
next(it)         # 1
next(it)         # 2
next(it)         # raises StopIteration
next(it, "done")  # "done" — pass a default to avoid the exception
```

Since PEP 479 (Python 3.7+), an unhandled `StopIteration` escaping a generator body is converted into a `RuntimeError`, to prevent it from silently terminating an enclosing `for` loop unexpectedly.

## 10. Exception Groups (Python 3.11+)

For situations where multiple unrelated exceptions need to be raised together (e.g., concurrent tasks each failing independently):

```python
try:
    raise ExceptionGroup("multiple failures", [ValueError("bad value"), TypeError("bad type")])
except* ValueError as eg:
    print("caught ValueErrors:", eg.exceptions)
except* TypeError as eg:
    print("caught TypeErrors:", eg.exceptions)
```

`except*` (new syntax) matches exceptions by type **within** the group without stopping at the first match, unlike a normal `except`.

## Notes & Gotchas

- **Never use a bare `except:`** — it also catches `SystemExit` and `KeyboardInterrupt`. Use `except Exception:` at minimum if you must catch broadly.
- **`except Exception as e:`** — the exception object `e` is deleted automatically at the end of the `except` block (a CPython/PEP 3110 detail); trying to reference it afterward raises `NameError`.
- **`finally` overriding `return`:** a `return` inside `finally` silently swallows any exception or return value from the `try`/`except` — a classic "gotcha" question.
  ```python
  def f():
      try:
          return 1
      finally:
          return 2   # this wins — f() returns 2, and any exception is lost too
  ```
- **Custom exceptions should call `super().__init__(...)`** so the message is captured properly and `str(exception)` works as expected.
- **`raise` with no argument** only works inside an `except` block (re-raises the currently-handled exception); outside one it raises `RuntimeError: No active exception to re-raise`.
- **Exceptions are relatively cheap on the success path** in Python, which is part of why EAFP is idiomatic — unlike some languages where exceptions carry heavy overhead regardless of whether they're thrown.
- **`else` after `try` is easy to forget** but is the right place for code that should only run when nothing went wrong, keeping it out of the `try` block (so it isn't accidentally caught by your own `except`).
