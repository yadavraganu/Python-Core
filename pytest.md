# Basic pytest Setup
####  **1. Install `pytest`**
```bash
pip install pytest
```
#### **2. Project Structure Example**
```
my_project/
├── app/
│   └── calculator.py
├── tests/
│   └── test_calculator.py
└── requirements.txt
```
#### **3. Sample Code (`calculator.py`)**
```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b
```
#### **4. Sample Test (`test_calculator.py`)**
```python
from app.calculator import add, subtract

def test_add():
    assert add(2, 3) == 5

def test_subtract():
    assert subtract(5, 3) == 2
```
#### **5. Run Tests**
From the root directory:
```bash
pytest
```
You’ll see output like:
```
============================= test session starts =============================
collected 2 items

tests/test_calculator.py ..                                           [100%]

============================== 2 passed in 0.01s ==============================
```
# Fixture in pytest
In **pytest**, a **fixture** is a way to provide **setup and teardown code** for your tests. Fixtures help you prepare the environment before a test runs and clean it up afterward. They’re reusable, modular, and can be scoped to function, class, module, or session.

#### **Basic Fixture Example**
```python
import pytest

@pytest.fixture
def sample_data():
    return {"a": 1, "b": 2}

def test_sum(sample_data):
    result = sample_data["a"] + sample_data["b"]
    assert result == 3
```

- `@pytest.fixture` marks `sample_data` as a fixture.
- It gets automatically injected into the test function `test_sum`.

#### **Setup and Teardown Example**
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

- `yield` separates setup and teardown.
- Code before `yield` runs before the test.
- Code after `yield` runs after the test.

#### **Fixture Scope**

You can control how often a fixture is created using `scope`:

```python
@pytest.fixture(scope="module")
def db_connection():
    print("Connecting to DB")
    yield "db_conn"
    print("Disconnecting DB")
```

Scopes:
- `"function"` (default): runs before each test
- `"class"`: runs once per test class
- `"module"`: runs once per module
- `"session"`: runs once per test session

#### **Fixture Dependency**

Fixtures can depend on other fixtures:

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
