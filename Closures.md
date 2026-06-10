# Python Closures
A closure in Python is a technique by which a function retains the memory of the environment in which it was created, even after the outer (enclosing) function has finished executing. More specifically, a closure is a nested function that "remembers" and can access the non-local variables from its enclosing scope.

This concept is fundamental to understanding decorators, as decorators are essentially closures in disguise.

## Core Concepts
__Nested Functions:__ A function defined inside another function.

__Non-local Variables:__ Variables that are not in the local scope of the nested function, nor in the global scope, but rather in the scope of an enclosing (outer) function.

__Returning a Nested Function:__ The outer function must return the nested function.

__Retention of Environment:__ The crucial part is that the returned nested function "closes over" the variables from its creation environment.
```python
def outer_function(x):
    # 'x' is a non-local variable for inner_function
    def inner_function(y):
        return x + y
    return inner_function

# Step 1: Call outer_function, which returns inner_function
# 'add_five' now holds a reference to 'inner_function'
# The environment where 'inner_function' was defined (where x=5) is "closed over"
add_five = outer_function(5)

# Step 2: Call the returned inner_function
# Even though outer_function has finished executing, add_five (which is inner_function)
# still remembers and can access 'x' (which is 5).
result1 = add_five(3) # 5 + 3 = 8
print(f"Result 1: {result1}")

add_ten = outer_function(10)
result2 = add_ten(7) # 10 + 7 = 17
print(f"Result 2: {result2}")

# You can see that add_five and add_ten are distinct instances
# of the inner_function, each with its own 'x' value remembered.
print(f"Type of add_five: {type(add_five)}")
print(f"Type of add_ten: {type(add_ten)}")
```
## Closure Variables Storage
Python stores closure variables in a specialized object called a cell object.Let's break down why and how this happens:
```python
def outer_function(x):
    def inner_function():
        return x * 2
    return inner_function

func1 = outer_function(5)
func2 = outer_function(10)
```
__Lifetime Discrepancy:__ When outer_function(5) finishes executing, its local scope (where x=5 resided) would normally be destroyed. However, inner_function (which func1 refers to) still needs access to that x. If x were just a regular local variable, it would be gone.

__Multiple Instances:__ Both func1 and func2 need their own distinct version of x. If x was stored in some shared global space, func1 might incorrectly get x=10 when it should have x=5.

To resolve these issues, Python introduces cell objects which works as below
- When a closure is formed (i.e., an inner function references a non-local variable from an outer scope), Python doesn't directly store the value of the non-local variable within the inner function's own scope. Instead:
- The outer function creates a cell object for each non-local variable that is referenced by any inner (nested) function.
- The value of the non-local variable is stored inside this cell object.
- Both the outer function and the inner function (or functions) get a reference to this same cell object.

This mechanism ensures:
__Persistence:__ Even after the outer function finishes executing, the cell object (and thus the value it holds) persists as long as there is at least one reference to it (which the inner function holds).

__Shared Access:__ If multiple inner functions in the same outer scope refer to the same non-local variable, they all share the same cell object. This means if one inner function modifies the value in the cell (using nonlocal), the change is visible to all other inner functions and the outer function itself (if it still had access to the variable).

__Distinct Instances:__ When outer_function is called multiple times (e.g., outer_function(5) and outer_function(10)), each call creates a new set of cell objects for its respective non-local variables. This ensures that func1's x is separate from func2's x.

```python
def outer_function(x): # x is initially 5
    # Python sees that inner_function will use 'x'.
    # It creates a cell object. Let's imagine it as Cell_X.
    # Cell_X.cell_contents = 5

    def inner_function():
        # inner_function now has a reference to Cell_X
        return Cell_X.cell_contents * 2 # Accesses the value through the cell
    return inner_function # Returns inner_function, which carries the reference to Cell_X
```
Now, when you call func1 = outer_function(5):

- func1 is the inner_function returned.
- func1 has an attribute `__closure__` which is a tuple.
- This tuple contains one cell object.
- That cell object's cell_contents holds the integer 5.

When you call func2 = outer_function(10):

- func2 is a different inner_function instance.
- It has its own `__closure__` attribute.
- This `__closure__` tuple contains a new cell object.
- That new cell object's cell_contents holds the integer 10.
## Practical Inspection (`__closure__` and cell_contents)
```python
def outer_function(x):
    def inner_function(y):
        return x + y
    return inner_function

add_five = outer_function(5)
add_ten = outer_function(10)

print(f"add_five.__closure__: {add_five.__closure__}")
# Output: (<cell at 0x...: int object at 0x...>,)
# This is a tuple containing one cell object.

print(f"add_five.__closure__[0]: {add_five.__closure__[0]}")
# Output: <cell at 0x...: int object at 0x...>
# This is the cell object itself.

print(f"add_five.__closure__[0].cell_contents: {add_five.__closure__[0].cell_contents}")
# Output: 5
# This is the actual value stored inside the cell.

print(f"add_ten.__closure__[0].cell_contents: {add_ten.__closure__[0].cell_contents}")
# Output: 10
```
## Application of Closures
### 1. Function Factory
```python
def make_multiplier(factor):
    def multiplier(number):
        return number * factor
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)
quadruple = make_multiplier(4)

print(f"2 * 5 = {double(5)}")    # factor = 2
print(f"3 * 7 = {triple(7)}")    # factor = 3
print(f"4 * 10 = {quadruple(10)}") # factor = 4
```
### 2. Simple Counter
```python
def create_counter():
    count = 0 # Non-local variable

    def increment():
        nonlocal count # Declare intent to modify non-local variable
        count += 1
        return count
    return increment

counter1 = create_counter()
print(f"Counter 1: {counter1()}") # 1
print(f"Counter 1: {counter1()}") # 2

counter2 = create_counter() # New independent counter
print(f"Counter 2: {counter2()}") # 1
print(f"Counter 1: {counter1()}") # 3 (counter1 is still independent)
```
### 3. Data Hiding / Encapsulation
```python
def bank_account(initial_balance):
    balance = initial_balance # This is "private"

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
print(f"Initial balance: {account['get_balance']()}") # 100

account['deposit'](50)
print(f"Balance after deposit: {account['get_balance']()}") # 150

account['withdraw'](75)
print(f"Balance after withdrawal: {account['get_balance']()}") # 75

account['withdraw'](100) # Fails
print(f"Balance after failed withdrawal: {account['get_balance']()}") # 75

# You cannot directly access account.balance
# print(account.balance) # AttributeError: 'dict' object has no attribute 'balance'
```
