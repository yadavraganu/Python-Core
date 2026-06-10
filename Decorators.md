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

# This is now our decorated function
ordinary_function()
```

This code does *exactly* the same thing as the manual example. The `@make_pretty` line is equivalent to `ordinary_function = make_pretty(ordinary_function)`.

### Making Decorators Useful (Arguments & Return Values)

The first example was simple, but most functions take arguments and return values. Our decorator needs to handle this.

#### Problem 1: Arguments

What if `ordinary_function` takes an argument? Our `wrapper` doesn't. We use `*args` and `**kwargs` to accept *any* arguments.

#### Problem 2: Return Values

What if `ordinary_function` returns a value? Our `wrapper` needs to capture and return it.

#### Problem 3: Metadata

When you decorate a function, it loses its original "identity" (like its name and docstring).

  * `ordinary_function.__name__` would become `'wrapper'`.
    We fix this using **`functools.wraps`**.

#### The "Proper" Decorator Template

Here is a practical, reusable decorator template that solves all these problems. Let's make a `timer_decorator` to time how long a function takes.

```python
import time
from functools import wraps # Import wraps

def timer_decorator(func):
    """A decorator that prints the time a function takes to run."""
    
    # 3. Use @wraps to preserve function metadata
    @wraps(func)
    
    # 1. Use *args and **kwargs to accept any arguments
    def wrapper(*args, **kwargs):
        start_time = time.perf_counter()
        
        # 2. Capture and return the original function's return value
        value = func(*args, **kwargs)
        
        end_time = time.perf_counter()
        run_time = end_time - start_time
        print(f"Finished {func.__name__!r} in {run_time:.4f} secs")
        return value
    return wrapper

# --- Using the decorator ---

@timer_decorator
def complex_calculation(num1, num2):
    """A function that 'sleeps' to simulate work."""
    time.sleep(1)
    return num1 + num2

# Call the decorated function
result = complex_calculation(10, 5)
print(f"Function result: {result}")

# Thanks to @wraps, the metadata is correct!
print(f"Function name: {complex_calculation.__name__}")
print(f"Function docstring: {complex_calculation.__doc__}")
```

**Output:**

```
Finished 'complex_calculation' in 1.0005 secs
Function result: 15
Function name: complex_calculation
Function docstring: A function that 'sleeps' to simulate work.
```
### Decorators with Arguments

What if you want to pass arguments *to the decorator itself*?
For example, `@repeat(num_times=3)` or `@check_permission(role="admin")`.

This requires **one extra layer** of functions.

1.  An outer function that accepts the decorator's arguments (e.g., `num_times=3`).
2.  This function returns the *actual decorator*.
3.  The decorator returns the *wrapper*.

It's a "function factory" that builds a decorator.

#### Example: `@repeat(num_times=N)`

Let's build a decorator that runs a function `N` times.

```python
from functools import wraps

# 1. Outer function accepts decorator's arguments
def repeat(num_times):
    
    # 2. This is the actual decorator
    def decorator_repeat(func):
        @wraps(func)
        
        # 3. This is the wrapper, as before
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(num_times):
                result = func(*args, **kwargs)
                results.append(result)
            return results # Or return the last result, etc.
        return wrapper
    
    return decorator_repeat # Return the decorator

# --- Using the decorator ---

@repeat(num_times=3)
def greet(name):
    print(f"Hello, {name}!")
    return name

# Call the decorated function
greet("Alice")
```

**Output:**

```
Hello, Alice!
Hello, Alice!
Hello, Alice!
```

**How it works:**

1.  `@repeat(num_times=3)` is called.
2.  The outer `repeat(num_times=3)` function runs and returns `decorator_repeat`.
3.  Python then effectively does `@decorator_repeat`, which decorates `greet`.

### Class-Based Decorators

You can also use a class to build a decorator. This is most useful when you need to **maintain state** between function calls.

A class-based decorator works by implementing two methods:

  * `__init__(self, func)`: Receives the function to be decorated (runs once at decoration time).
  * `__call__(self, *args, **kwargs)`: Makes the class instance *callable*. This method runs *every time* the decorated function is called.

#### Example: `CountCalls`

Let's build a decorator that counts how many times a function has been called.

```python
from functools import update_wrapper

class CountCalls:
    # 1. Runs once when decorating
    def __init__(self, func):
        update_wrapper(self, func) # Helps preserve metadata
        self.func = func
        self.num_calls = 0 # This is our "state"
        
    # 2. Runs every time the decorated function is called
    def __call__(self, *args, **kwargs):
        self.num_calls += 1
        print(f"Call {self.num_calls} of {self.func.__name__!r}")
        return self.func(*args, **kwargs)

# --- Using the decorator ---

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

The `self.num_calls` variable is the *state* that persists between calls. This is much cleaner than using a global variable.

### Common Use Cases

You see decorators all the time in Python:

  * **Logging:** As seen in the `timer_decorator`, used for logging function calls, arguments, and return values.
  * **Authentication & Authorization:** In web frameworks like Flask or Django, you use decorators like `@login_required` or `@permission_required` to protect routes.
  * **Caching / Memoization:** Storing the results of expensive function calls. Python's built-in **`@functools.lru_cache`** is a powerful decorator for this.
  * **Rate Limiting:** Restricting how often a function (like an API endpoint) can be called.
  * **Registering Functions:** Some frameworks (like `pytest` with `@pytest.fixture`) use decorators to register functions in a central registry.

Would you like to dive deeper into a specific example, such as how `@functools.lru_cache` works or how to build an `@login_required` decorator?
