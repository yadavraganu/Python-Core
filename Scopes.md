## Scopes in Python
In Python, **scope** refers to the region of a program where a variable is **defined** and **accessible**. It determines the **visibility** and **lifetime** of variables. Python uses a well-defined rule called the **LEGB Rule** to resolve variable names.

### The LEGB Rule

LEGB stands for:

| Scope Level     | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| **L**ocal        | Names defined inside a function (including parameters)                      |
| **E**nclosing    | Names in enclosing functions (for nested functions)                         |
| **G**lobal       | Names defined at the top-level of a module or file                          |
| **B**uilt-in     | Names preloaded by Python (e.g. `len`, `print`, `sum`)                      |

Python searches for a variable in this order: **Local → Enclosing → Global → Built-in**

### Examples of Each Scope

#### 1. **Local Scope**

```python
def greet():
    name = "Anurag"  # Local to greet()
    print(name)
```

#### 2. **Enclosing Scope**

```python
def outer():
    msg = "Hello"  # Enclosing scope for inner()
    def inner():
        print(msg)
    inner()
```

#### 3. **Global Scope**

```python
greeting = "Hi"  # Global variable

def say():
    print(greeting)
```

#### 4. **Built-in Scope**

```python
print(len("Python"))  # 'len' is a built-in function
```
### Accessing and Modifying Scope

#### Modifying Global Variables

Use the `global` keyword to modify a global variable inside a function:

```python
counter = 0

def increment():
    global counter
    counter += 1
```

#### Modifying Enclosing Variables

Use `nonlocal` to modify a variable in an enclosing (non-global) scope:

```python
def outer():
    count = 0
    def inner():
        nonlocal count
        count += 1
        print(count)
    inner()
```
### Inspecting Scope Programmatically

#### 1. **Using `globals()` and `locals()`**

```python
x = 10

def test():
    y = 20
    print("Local:", locals())
    print("Global:", globals())

test()
```

- `globals()` returns a dictionary of global names  
- `locals()` returns a dictionary of local names (inside a function)

#### 2. **Using `dir()`**

```python
print(dir())  # Lists names in the current scope
```
### Summary

| Scope Type     | Keyword to Modify | Access Method         |
|----------------|-------------------|------------------------|
| Local          | —                 | `locals()`             |
| Enclosing      | `nonlocal`        | Closure or nested func |
| Global         | `global`          | `globals()`            |
| Built-in       | —                 | `__builtins__`         |
