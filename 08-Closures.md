# Python Closures

A closure is a technique by which a function retains the memory of the environment in which it was created, even after the outer (enclosing) function has finished executing. More precisely, a closure is a nested function that **remembers** and can access the non-local variables from its enclosing scope, even after that enclosing scope has exited.

This concept is fundamental to understanding **decorators** — a decorator is, at its core, a closure that wraps another function.

## Core Concepts

- **Nested Function:** A function defined inside another function.
- **Non-local Variable:** A variable that is neither local to the nested function nor global, but lives in the scope of an enclosing (outer) function.
- **Returning a Nested Function:** The outer function must return the nested function for a closure to be useful (though closures can also just be stored/passed around).
- **Retention of Environment:** The returned nested function "closes over" the variables from its creation environment — this is where the name *closure* comes from.

```python
def outer_function(x):
    # 'x' is a non-local variable for inner_function
    def inner_function(y):
        return x + y
    return inner_function

# Step 1: Call outer_function, which returns inner_function.
# 'add_five' now holds a reference to inner_function.
# The environment where inner_function was defined (where x=5) is "closed over".
add_five = outer_function(5)

# Step 2: Call the returned inner_function.
# Even though outer_function has finished executing, add_five still remembers x=5.
result1 = add_five(3)  # 5 + 3 = 8
print(f"Result 1: {result1}")

add_ten = outer_function(10)
result2 = add_ten(7)   # 10 + 7 = 17
print(f"Result 2: {result2}")

# add_five and add_ten are distinct closures, each remembering its own 'x'.
print(f"Type of add_five: {type(add_five)}")
print(f"Type of add_ten: {type(add_ten)}")
```

### What Qualifies as a Closure? (3-part test)
A function is a closure only if **all three** hold:
1. It's a nested function.
2. It references a variable from its enclosing (non-global) scope.
3. The enclosing function has returned / finished executing.

A nested function that only uses global variables, or that never escapes its enclosing function, is **not** a closure in the technical sense.

## Closure Variable Storage — Cell Objects

Python stores closure variables in a specialized object called a **cell object**, not by copying the value directly into the inner function.

```python
def outer_function(x):
    def inner_function():
        return x * 2
    return inner_function

func1 = outer_function(5)
func2 = outer_function(10)
```

**Why cells are needed:**

- **Lifetime discrepancy:** When `outer_function(5)` finishes, its local scope would normally be destroyed — but `inner_function` still needs `x`. A plain local variable would be gone by then.
- **Multiple instances:** `func1` and `func2` each need their own independent `x`. A shared global slot would let one call clobber the other.

**How it works:**
- When Python compiles a function and detects that an inner function references a variable from the outer scope, it marks that variable as a **free variable** in the inner function and a **cell variable** in the outer function.
- The outer function allocates a **cell object** to hold the variable's value instead of a normal stack slot.
- Both the outer and inner functions hold a reference to the *same* cell object, so they always see the same underlying value.

This gives you:
- **Persistence** — the cell (and its value) survives as long as anything references it, even after the outer function returns.
- **Shared access** — if multiple inner functions in the same outer scope reference the same variable, they all share one cell; mutating it via `nonlocal` in one is visible to the others.
- **Distinct instances per call** — each call to `outer_function` creates a fresh cell, so `func1`'s `x` and `func2`'s `x` never interfere.

```python
def outer_function(x):  # x is initially 5
    # Python creates a cell object, conceptually: Cell_X.cell_contents = 5
    def inner_function():
        return Cell_X.cell_contents * 2  # accesses value through the cell
    return inner_function
```

## Practical Inspection: `__closure__` and `cell_contents`

```python
def outer_function(x):
    def inner_function(y):
        return x + y
    return inner_function

add_five = outer_function(5)
add_ten = outer_function(10)

print(add_five.__closure__)
# (<cell at 0x...: int object at 0x...>,)  — a tuple of cell objects

print(add_five.__closure__[0])
# <cell at 0x...: int object at 0x...>

print(add_five.__closure__[0].cell_contents)
# 5

print(add_ten.__closure__[0].cell_contents)
# 10
```

You can also inspect which names are free variables vs. locally defined, from the code object itself:

```python
print(add_five.__code__.co_freevars)   # ('x',)  — names closed over
print(outer_function.__code__.co_varnames)  # locals of outer_function
```

If a function has no closure, `__closure__` is `None`.

## `nonlocal` vs `global`

- `global` binds a name to the module-level (global) scope.
- `nonlocal` binds a name to the **nearest enclosing function scope** (not global) — this is what lets an inner function *rebind* (not just read) a variable from its closure.

```python
def outer():
    x = 1
    def inner():
        nonlocal x   # without this, 'x += 1' would raise UnboundLocalError
        x += 1
        return x
    return inner
```

Without `nonlocal`, assigning to `x` inside `inner` would make Python treat `x` as a *new local variable* of `inner`, and reading it before that assignment raises `UnboundLocalError` — a very common interview gotcha. Reading a non-local variable (without assigning to it) never requires `nonlocal`; only rebinding does.

## The Classic Gotcha: Late Binding in Loops

Closures capture **variables**, not the **values** those variables held at creation time. This causes a well-known bug when closures are created in a loop:

```python
def make_multipliers():
    multipliers = []
    for i in range(3):
        def multiplier(x):
            return x * i          # 'i' is looked up when multiplier() is *called*
        multipliers.append(multiplier)
    return multipliers

funcs = make_multipliers()
print([f(10) for f in funcs])
# [20, 20, 20]  — NOT [0, 10, 20]!
# All three closures share the same cell for 'i', and by the time
# they're called, the loop has finished and i == 2.
```

**Fix 1 — default argument (evaluated at definition time, not call time):**
```python
def make_multipliers():
    multipliers = []
    for i in range(3):
        def multiplier(x, i=i):   # binds current value of i as a default
            return x * i
        multipliers.append(multiplier)
    return multipliers
```

**Fix 2 — extra enclosing scope (factory function) so each `i` gets its own cell:**
```python
def make_multiplier(i):
    def multiplier(x):
        return x * i
    return multiplier

funcs = [make_multiplier(i) for i in range(3)]
```

This is one of the most frequently asked "explain this output" questions in Python interviews.

---

## Applications of Closures

### 1. Function Factory
```python
def make_multiplier(factor):
    def multiplier(number):
        return number * factor
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)
quadruple = make_multiplier(4)

print(f"2 * 5 = {double(5)}")
print(f"3 * 7 = {triple(7)}")
print(f"4 * 10 = {quadruple(10)}")
```

### 2. Simple Counter
```python
def create_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count
    return increment

counter1 = create_counter()
print(counter1())  # 1
print(counter1())  # 2

counter2 = create_counter()  # independent counter
print(counter2())  # 1
print(counter1())  # 3 — counter1 is unaffected by counter2
```

### 3. Data Hiding / Encapsulation
```python
def bank_account(initial_balance):
    balance = initial_balance  # "private" — no external attribute access

    def get_balance():
        return balance

    def deposit(amount):
        nonlocal balance
        if amount > 0:
            balance += amount
            return True
        return False

    def withdraw(amount):
        nonlocal balance
        if 0 < amount <= balance:
            balance -= amount
            return True
        return False

    return {'get_balance': get_balance, 'deposit': deposit, 'withdraw': withdraw}

account = bank_account(100)
print(account['get_balance']())      # 100
account['deposit'](50)
print(account['get_balance']())      # 150
account['withdraw'](75)
print(account['get_balance']())      # 75
account['withdraw'](100)             # fails — insufficient funds
print(account['get_balance']())      # 75

# print(account.balance)  # AttributeError — no such attribute exists
```

### 4. Decorators (closures in disguise)
```python
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)      # 'func' is closed over
        print(f"{func.__name__} took {time.time() - start:.4f}s")
        return result
    return wrapper

@timer
def slow_add(a, b):
    return a + b
```
Here `wrapper` is a closure over `func` — this is exactly why decorators are described as "closures with syntax sugar."

### 5. Memoization / Caching
```python
def memoize(func):
    cache = {}                 # shared, private cache — closed over
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def slow_square(n):
    return n * n
```

## Closures vs. Classes

Anything a closure does (retain state between calls, hide internal data) can also be done with a class using instance attributes. Rule of thumb often asked in interviews:

- **Use a closure** when you need to encapsulate a small amount of state behind **one or a few functions** (like the `bank_account` dict-of-functions example, or a decorator).
- **Use a class** when you have **multiple pieces of state and several related behaviors**, need inheritance, or want a clearer, more discoverable interface (`obj.method()` vs `funcs['method']()`).

```python
# Closure version
def counter_closure():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

# Equivalent class version
class Counter:
    def __init__(self):
        self.count = 0
    def increment(self):
        self.count += 1
        return self.count
```

## Notes & Gotchas

- **Closures capture variables by reference (via cells), not by value** — this is the root cause of the loop gotcha above.
- **`UnboundLocalError`:** assigning to a name anywhere in a function makes Python treat it as local for the *entire* function body, even before the assignment line — this is why `nonlocal`/`global` are needed for rebinding outer variables.
- **Mutable objects don't need `nonlocal`:** if the outer variable is a mutable object (e.g., a list or dict), you can *mutate* it (`lst.append(x)`) from the inner function without `nonlocal` — you only need `nonlocal` to *rebind* the name itself (`lst = []`).
- **Memory:** as long as a closure exists, the cells (and whatever they reference) stay alive — a subtle source of memory retention if closures capture large objects unnecessarily.
- **`__closure__` is `None`** for functions that don't close over anything, even if they're nested.
- Closures are why **decorators**, **partial application**, and patterns like memoization work the way they do in Python.
