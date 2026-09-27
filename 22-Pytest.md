# Basic pytest Setup

#### 1. Install `pytest`
```bash
pip install pytest
```

#### 2. Project Structure Example
```
my_project/
├── app/
│   └── calculator.py
├── tests/
│   └── test_calculator.py
└── requirements.txt
```

#### 3. Sample Code (`calculator.py`)
```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b
```

#### 4. Sample Test (`test_calculator.py`)
```python
from app.calculator import add, subtract

def test_add():
    assert add(2, 3) == 5

def test_subtract():
    assert subtract(5, 3) == 2
```

#### 5. Run Tests
```bash
pytest
```
```
============================= test session starts =============================
collected 2 items

tests/test_calculator.py ..                                           [100%]

============================== 2 passed in 0.01s ==============================
```

#### Test Discovery Rules

pytest finds tests automatically by convention, with no registration step needed:
- Files matching `test_*.py` or `*_test.py`
- Functions prefixed `test_`
- Classes prefixed `Test` (with no `__init__` method), containing methods prefixed `test_`

Knowing these conventions matters — a test function named `check_add` instead of `test_add` is silently **never run**, a common source of "why isn't my test executing" confusion.

---

# Assertions — Plain `assert`, No Special API

Unlike `unittest` (`self.assertEqual`, `self.assertTrue`, ...), pytest uses Python's built-in `assert` directly, and rewrites it at import time to give rich, introspective failure output:

```python
def test_add():
    assert add(2, 3) == 5    # on failure, pytest shows both sides' actual values automatically
```

If this fails, pytest doesn't just say "assertion failed" — it shows the actual computed values on both sides of the comparison, which is why you rarely need custom assertion messages compared to older frameworks.

## Testing for Exceptions — `pytest.raises`

```python
import pytest

def divide(a, b):
    if b == 0:
        raise ValueError("cannot divide by zero")
    return a / b

def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(10, 0)

def test_divide_by_zero_message():
    with pytest.raises(ValueError, match="cannot divide by zero"):
        divide(10, 0)
```

`match` takes a regex checked against `str(exception)` — useful for asserting not just *that* an exception was raised, but that it carries the right message.

---

# Fixtures in pytest

A **fixture** provides **setup and teardown code** for tests — preparing the environment before a test runs and cleaning up afterward. Fixtures are reusable, modular, and can be scoped to function, class, module, or session.

#### Basic Fixture Example
```python
import pytest

@pytest.fixture
def sample_data():
    return {"a": 1, "b": 2}

def test_sum(sample_data):
    result = sample_data["a"] + sample_data["b"]
    assert result == 3
```
`@pytest.fixture` marks `sample_data` as a fixture; it's automatically injected into any test function that names it as a parameter — pytest matches by **name**, not by import.

#### Setup and Teardown Example
```python
import pytest

@pytest.fixture
def resource():
    print("Setting up resource")
    yield {"status": "ready"}
    print("Tearing down resource")

def test_resource_usage(resource):
    assert resource["status"] == "ready"
```
`yield` separates setup (before) from teardown (after) — code after `yield` runs even if the test fails, similar in spirit to a `finally` block.

#### Fixture Scope
```python
@pytest.fixture(scope="module")
def db_connection():
    print("Connecting to DB")
    yield "db_conn"
    print("Disconnecting DB")
```
- `"function"` (default) — runs before each test.
- `"class"` — once per test class.
- `"module"` — once per module (file).
- `"session"` — once per entire test run.

**Gotcha:** a wider-scoped fixture (e.g., `"module"`) that returns a **mutable object** (a list, dict) is *shared* across every test in that scope — if one test mutates it, later tests in the same scope see the mutation. This is a common source of order-dependent test failures ("my tests pass individually but fail together").

#### Fixture Dependency
```python
@pytest.fixture
def config():
    return {"env": "test"}

@pytest.fixture
def client(config):
    return f"Client for {config['env']}"

def test_client(client):
    assert "Client for test" == client
```
Fixtures can depend on other fixtures, forming a dependency graph — pytest resolves and injects the whole chain automatically.

## `autouse` Fixtures — Run Without Being Requested

Sometimes you want setup/teardown to run for *every* test in scope, without every test function explicitly listing the fixture as a parameter:

```python
@pytest.fixture(autouse=True)
def reset_global_state():
    print("Resetting shared state before test")
    yield
    print("Cleanup after test")
```
Useful for things like resetting a global cache or clearing a database table between tests — but use sparingly, since implicit behavior that runs everywhere can make test failures harder to trace back to a cause.

## Sharing Fixtures Across Files — `conftest.py`

Defining the same fixture in every test file doesn't scale. A file named `conftest.py` in a test directory makes its fixtures **automatically available** to every test file in that directory (and subdirectories), with no import needed:

```
tests/
├── conftest.py        # fixtures defined here are visible to every test file below
├── test_calculator.py
└── test_api.py
```
```python
# tests/conftest.py
import pytest

@pytest.fixture
def sample_data():
    return {"a": 1, "b": 2}
```
This is the standard mechanism for sharing setup logic across a whole test suite, and a near-guaranteed practical-round question ("how do you share a fixture across multiple test files").

## Useful Built-In Fixtures

pytest ships several fixtures you can request without defining them yourself:

- **`tmp_path`** — a unique, temporary `pathlib.Path` per test, auto-cleaned up:
  ```python
  def test_write_file(tmp_path):
      file = tmp_path / "output.txt"
      file.write_text("hello")
      assert file.read_text() == "hello"
  ```
- **`capsys`** — captures stdout/stderr printed during a test:
  ```python
  def test_print_output(capsys):
      print("hello")
      captured = capsys.readouterr()
      assert captured.out == "hello\n"
  ```
- **`monkeypatch`** — safely patches attributes, environment variables, or dict entries for the duration of a test, automatically undoing the change afterward:
  ```python
  def test_env_var(monkeypatch):
      monkeypatch.setenv("API_KEY", "test-key")
      import os
      assert os.environ["API_KEY"] == "test-key"

  def test_patch_function(monkeypatch):
      import app.calculator as calc
      monkeypatch.setattr(calc, "add", lambda a, b: 999)
      assert calc.add(2, 3) == 999
  ```

---

# Parametrizing Tests — `@pytest.mark.parametrize`

Instead of writing near-identical test functions for different inputs, `parametrize` runs the same test body once per input set:

```python
import pytest

@pytest.mark.parametrize("a, b, expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected
```
Output shows each case as a separate test result (`test_add_parametrized[2-3-5]`, etc.), so a failure immediately identifies *which* input combination failed — much clearer than one test looping internally over cases with a single pass/fail result.

---

# Marks — `skip`, `skipif`, `xfail`

```python
import pytest
import sys

@pytest.mark.skip(reason="not implemented yet")
def test_future_feature():
    ...

@pytest.mark.skipif(sys.version_info < (3, 10), reason="requires Python 3.10+")
def test_new_syntax():
    ...

@pytest.mark.xfail(reason="known bug, tracked in issue #42")
def test_known_bug():
    assert add(2, 2) == 5   # expected to fail; reported as 'xfail', not a suite failure
```
- **`skip`** — never runs the test.
- **`skipif`** — conditionally skips, based on an expression.
- **`xfail`** — runs the test but doesn't count a failure against the suite (marked `XFAIL`); if it unexpectedly *passes*, it's reported as `XPASS`, which can optionally be configured to fail the suite (catching a bug that's since been fixed but never had its `xfail` marker removed).

Custom marks (e.g., `@pytest.mark.slow`) can be registered in `pytest.ini`/`pyproject.toml` and used to selectively run subsets of tests (`pytest -m slow`).

---

# Useful CLI Flags

```bash
pytest -v                  # verbose — show each test name and result
pytest -x                  # stop after the first failure
pytest -k "add"             # run only tests whose name matches "add"
pytest -m slow              # run only tests marked @pytest.mark.slow
pytest --lf                 # rerun only tests that failed last time
pytest --maxfail=2          # stop after 2 failures
pytest --cov=app             # coverage report (requires pytest-cov)
```

---

# pytest vs. `unittest` — Quick Comparison

| | `unittest` (standard library) | `pytest` |
|---|---|---|
| Assertions | `self.assertEqual(a, b)`, `self.assertTrue(x)` | Plain `assert a == b` |
| Test structure | Must subclass `unittest.TestCase` | Plain functions — no class required |
| Setup/teardown | `setUp()`/`tearDown()` methods | Fixtures (`@pytest.fixture`), more composable |
| Parametrization | Manual looping or `subTest()` | Built-in `@pytest.mark.parametrize` |
| Fixture sharing | Inheritance between `TestCase` classes | `conftest.py`, dependency injection by name |
| Ecosystem | Built into the standard library | Large plugin ecosystem (`pytest-cov`, `pytest-mock`, `pytest-django`, ...) |

pytest can still run `unittest`-style tests unmodified — it's a superset in practice, which is part of why many codebases migrate to it incrementally rather than rewriting everything at once.

---

## Notes & Gotchas Worth Knowing for Interviews

- **Test discovery is convention-based** — a test not named `test_*`/`*_test` and not inside a `Test*` class is silently skipped, not an error.
- **`pytest.raises` as a context manager is the idiomatic way to test exceptions** — know the `match=` parameter for asserting on the message, not just the exception type.
- **Wider-scoped fixtures sharing mutable state is a classic source of flaky, order-dependent tests** — a strong answer explains *why* (shared object reference across tests in the same scope), not just "use scope carefully."
- **`conftest.py` requires no import** — fixtures defined there are auto-discovered by pytest for every test file in that directory tree, which surprises people used to explicit imports.
- **`monkeypatch` auto-reverts after each test** — this is exactly why it's preferred over manually saving/restoring an attribute or env var yourself.
- **`xfail` vs. `skip`**: `skip` never runs the test at all; `xfail` runs it but doesn't fail the suite on failure — an important distinction when explaining how to track known bugs without blocking CI.
- **pytest rewrites `assert` statements at import time** (via its import hook) to produce detailed failure diffs — this is *why* plain `assert` gives rich output in pytest but not in a plain Python script.
