### What Is a Context Manager?
A **context manager** is a construct that handles the setup and teardown of resources automatically. It’s most commonly used with the `with` statement.
```python
with open('file.txt', 'r') as f:
    data = f.read()
# File is automatically closed here
```
This ensures the file is closed even if an error occurs during reading.
### Why Use Context Managers?

- **Automatic cleanup**: Frees resources like files, sockets, or DB connections.
- **Exception-safe**: Cleanup happens even if an error is raised.
- **Cleaner code**: Replaces verbose `try-finally` blocks.
- **Custom control**: You can manage any resource using `__enter__` and `__exit__` methods.

### Real-World Use Cases

- **File handling**: `open()`
- **Database connections**: `psycopg2.connect()`
- **Thread locks**: `threading.Lock()`
- **Web scraping**: Closing browser sessions
- **Socket programming**: Managing open sockets
- **Subprocesses**: Cleaning up child processes

### Custom Context Manager with Exception Handling
```python
class SafeDivision:
    def __init__(self, a, b):
        self.a = a
        self.b = b

    def __enter__(self):
        print("Entering context: preparing to divide")
        return self

    def divide(self):
        print("➗ Performing division")
        return self.a / self.b

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type:
            print(f"Exception occurred: {exc_value}")
        print("Exiting context: cleaning up")
        # Suppress exception if handled
        return True  # Change to False to propagate exception
```
### Usage with Explanation

#### Case 1: No Exception
```python
with SafeDivision(10, 2) as sd:
    result = sd.divide()
    print(f"Result: {result}")
```
**Execution Flow:**
1. `__init__` is called → sets `a = 10`, `b = 2`
2. `__enter__` is called → prints "Entering context"
3. `divide()` is called → prints "Performing division"
4. `__exit__` is called → prints "Exiting context"
#### Case 2: With Exception (division by zero)
```python
with SafeDivision(10, 0) as sd:
    result = sd.divide()
    print(f"Result: {result}")
```
**Execution Flow:**
1. `__init__` is called → sets `a = 10`, `b = 0`
2. `__enter__` is called → prints "Entering context"
3. `divide()` raises `ZeroDivisionError`
4. `__exit__` is called → catches exception, prints it
5. Exception is **suppressed** because `__exit__` returns `True`

### Summary of Method Calls

| Method         | When It’s Called                          |
|----------------|--------------------------------------------|
| `__init__`     | When the context manager object is created |
| `__enter__`    | At the start of the `with` block           |
| `divide()`     | Inside the `with` block                    |
| `__exit__`     | At the end of the `with` block or on error |

### How `@contextmanager` Works

It splits the context manager logic into two parts:
- **Before `yield`**: Setup code (like opening a file or acquiring a lock)
- **After `yield`**: Teardown code (like closing the file or releasing the lock)

### Example: Logging Context Manager
```python
from contextlib import contextmanager

@contextmanager
def log_context(name):
    print(f"Entering: {name}")
    try:
        yield
    except Exception as e:
        print(f"Exception in {name}: {e}")
        raise  # Optional: re-raise if you want the error to propagate
    finally:
        print(f"Exiting: {name}")
```
Usage:
```python
with log_context("Test Block"):
    print("Doing work")
    # raise ValueError("Oops!")  # Uncomment to test exception handling
```
### Execution Flow

| Line | When It Runs |
|------|--------------|
| `print("Entering")` | Immediately when `with` starts |
| `yield`             | Pauses to run the block inside `with` |
| `except` block      | Runs if an exception occurs inside the block |
| `finally` block     | Always runs when the block ends (even on error) |

### Key Points

- You must `yield` exactly once.
- The value yielded is returned to the `with` block (e.g., a file object).
- If an exception occurs, it’s re-raised inside the generator at the `yield` point.
- You can suppress or log exceptions inside the `except` block.

## File Handling with `@contextmanager`

```python
from contextlib import contextmanager

@contextmanager
def open_file(filename, mode):
    print("Opening file")
    f = open(filename, mode)
    try:
        yield f  # This is where the file is used inside the `with` block
    except Exception as e:
        print(f"Error: {e}")
        raise
    finally:
        print("Closing file")
        f.close()
```
###  Usage
```python
with open_file("example.txt", "w") as file:
    file.write("Hello, Anurag!")
```
### Execution Flow
1. `open_file()` is called → file is opened
2. `yield f` → control passes to the `with` block
3. After block ends or error occurs → `finally` closes the file

## Database Connection with `@contextmanager`
```python
from contextlib import contextmanager
import sqlite3

@contextmanager
def db_connection(db_name):
    print("🔌 Connecting to database")
    conn = sqlite3.connect(db_name)
    try:
        yield conn  # Use the connection inside the block
        conn.commit()
        print("Committed changes")
    except Exception as e:
        conn.rollback()
        print(f"Rolled back due to: {e}")
        raise
    finally:
        print("Closing connection")
        conn.close()
```
### Usage
```python
with db_connection("test.db") as conn:
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)")
    cursor.execute("INSERT INTO users VALUES (?, ?)", (1, "Anurag"))
```
### Execution Flow
| Step | What Happens |
|------|--------------|
| `db_connection()` | Opens DB connection |
| `yield conn` | Executes SQL inside `with` block |
| `commit()` | Saves changes if no error |
| `rollback()` | Reverts changes if error occurs |
| `close()` | Always closes connection |
