## Python Modules and Packages

A module is a single .py file, whereas a package is a directory of modules. Code modularity allows you to break large applications into smaller, manageable, and reusable pieces.

## 1. Definitions and Architecture
### Modules
A module is a single Python file containing executable code, function definitions, classes, or variables.

* **File Name**: Any file ending in .py (e.g., calculator.py).
* **Purpose**: Groups related code together to avoid naming conflicts and maximize reusability.

### Packages
A package is a collection of modules organized in a directory structure.

* **Folder Name**: Any directory name that contains Python files.
* **The `__init__.py` file**: This file tells Python that the directory should be treated as a package.
* It can be completely empty.
   * It runs initialization code when the package is first imported.
   * It can use `__all__` to control what gets exported during wildcard (*) imports.

## Typical Directory Structure
```
my_project/                 # Project Root Folder
│
├── main.py                 # Main entry script
│
├── utilities.py            # A Module
│
└── shop/                   # A Package
    ├── __init__.py         # Makes 'shop' a package
    ├── billing.py          # Module inside 'shop'
    ├── inventory.py        # Module inside 'shop'
    │
    └── delivery/           # A Sub-package (nested package)
        ├── __init__.py     # Makes 'delivery' a sub-package
        └── tracking.py     # Module inside 'delivery'
```
<br></br>
## 2. The Import Mechanism
Python provides several syntax options to bring external code into your active workspace.
### Standard import
Imports the entire module. You must use the dot (.) syntax to access anything inside it.

* **Syntax**: `import package.module`
```python
import shop.billing
# Accessing a function requires the full path
shop.billing.create_invoice()
```
### Specific from ... import
Imports specific parts (functions, classes, variables) directly into your namespace. You do not use the module name prefix.

* **Syntax**: `from package.module import items`
```python
from shop.billing import create_invoice
# Accessing the function directly
create_invoice()
```
### Alias Imports (as)
Renames a module or a function locally. Use this to shorten long names or avoid naming conflicts.

* **Syntax**: `import module as alias`
```python
import shop.delivery.tracking as track
from shop.inventory import get_item_count as check_stock
track.locate_package()
items = check_stock()
```
### Wildcard Imports (from ... import *)
Imports everything from a module directly into your namespace.

* **Syntax**: `from module import *`
* **Warning**: Avoid using this in production code. It causes "namespace pollution" because you cannot easily tell where a function came from, and it may accidentally overwrite existing variables.
<br></br>
## 3. Controlling Exports with __all__
When someone uses a wildcard import (`from module import *`), you can control exactly what gets imported by defining an `__all__` list in that module or in the package's `__init__.py`.
```python
# Inside shop/billing.py
__all__ = ['create_invoice', 'calculate_tax']  # Only these two will be exported
def create_invoice():
    pass
def calculate_tax():
    pass
def _internal_helper():
    # This is hidden by default due to the underscore prefix
    pass
def hidden_bonus_function():
    # This is clean code, but excluded because it's not listed in __all__
    pass
```
If another file executes from shop.billing import *, they will only get access to create_invoice and calculate_tax.
<br></br>
## 4. The in Keyword with Imports
The in keyword is a membership operator in Python. While it is never written directly inside an import statement, it is highly useful for checking the status and contents of imports.
## Verifying Loaded Modules with sys.modules
When you import a module, Python loads it into memory and caches it in a dictionary called sys.modules. You can use in to verify if a module is currently active:
```python
import sys
# Check if 'json' has been imported anywhere in the applicationif "json" in sys.modules:
    print("The JSON module is cached and ready to use.")
```
## Checking Module Content with dir()
The dir() function returns a list of all functions, classes, and variables inside a module. You can use in to safely check if a feature exists before using it:
```python
import math
# Check if the 'factorial' function exists in this version of Pythonif "factorial" in dir(math):
    print(math.factorial(5))
```
<br></br>
## 5. Absolute vs. Relative Imports
When modules inside the same package need to share code, you must decide how to write their import paths.

| Metric | Absolute Imports | Relative Imports |
|---|---|---|
| Path Origin | Always starts from the project root directory. | Starts from the current module's position. |
| Syntax Style | Explicit full path using words. | Uses single (.) or double (..) dots. |
| PEP 8 Status | Highly Preferred for readability and safety. | Acceptable for complex, deeply nested packages. |
| Execution | Works from anywhere in the application. | Only works when executed as part of a package. |

## Absolute Imports
An absolute import outlines the exact path from the project root down to the desired resource.

# Working inside: shop/billing.pyfrom shop.inventory import get_item_count  # Clear, explicit, and readable

## Relative Imports
A relative import detects the location of the target module based on where the current file is located.

* . (one dot) indicates the current directory.
* .. (two dots) indicates the parent directory (one level up).

# Working inside: shop/billing.pyfrom .inventory import get_item_count  # Look in the same folder for inventory.py

# Working inside: shop/delivery/tracking.pyfrom ..billing import create_invoice   # Go up to 'shop', then look for billing.py

## The Relative Import Trap
Relative imports only work when Python runs your code as a package.
If you open your terminal, navigate directly into the shop/ folder, and run a script using relative imports directly:

python billing.py

You will get this error:
ImportError: attempted relative import with no known parent package
The Fix: Always run your scripts from the project root directory using the module flag (-m):

python -m shop.billing
<br></br>
## 6. Module Lifecycles and Resolution
### Module Caching and Re-execution
When Python imports a module, it executes the code inside that file exactly once. It then saves the module object to sys.modules.

* Subsequent imports of the same module do not re-run the file; they simply read it from the cache.
* If you need to forcefully re-run a module (e.g., during live debugging in a shell), you must use the importlib library:
```python
import importlibimport shop.billing

importlib.reload(shop.billing)  # Force re-executes the file
```

### How Python Locates Modules (sys.path)
When you run import xyz, Python loops through an ordered list of directory paths stored in sys.path:

   1. **The Current Directory**: The folder containing the script you just launched.
   2. **PYTHONPATH**: An optional environment variable set by the user containing extra directory paths.
   3. **Standard Library**: Built-in Python locations (like where math, os, and sys live).
   4. **Third-Party Packages (site-packages)**: Where packages downloaded via pip are installed.

If Python loops through all these directories and finds nothing, it raises a ModuleNotFoundError.
<br></br>
## 7. Modern Packaging with pyproject.toml
To share your package with other developers or publish it to PyPI (Python Package Index), modern Python standards use a pyproject.toml file at the root of your project instead of the old setup.py.
### Project Layout
```
my_shared_package/
├── pyproject.toml
├── README.md
└── src/
    └── shop/
        ├── __init__.py
        └── billing.py
```
### Example pyproject.toml configuration
```python
[build-system]
requires = ["setuptools>=61.0.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my_custom_shop_pkg"
version = "1.0.0"
authors = [
    { name="Your Name", email="you@example.com" }
]
description = "An ecommerce utility package"
readme = "README.md"
requires-python = ">=3.8"
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
]
dependencies = [
    "requests>=2.28.0"  # Third party requirements go here
]

[tool.setuptools.packages.find]
where = ["src"]  # Tells build tools to look inside the src directory
```
To build this package into distributable files (.tar.gz and .whl), run:
```python
pip install build
python -m build
```
<br></br>
## 8. Common Troubleshooting Guide
### 1. Circular Imports

* **Cause**: Module A imports Module B at the top of the file, and Module B imports Module A at the top of its file. This creates an infinite loop where neither module finishes initialization.
```python
# --- user.py ---
from .order import Order  
# Python freezes here waiting for order.py
class User: pass
# --- order.py ---
from .user import User   
# user.py is frozen, causing an AttributeError
class Order: pass
```
* **Solution A (Deferred Imports)**: Move the import statement inside the function or method that actually requires it, instead of putting it at the top of the file.
```python
# --- order.py ---
class Order:
    def assign_to_user(self):
        from .user import User  # Inline import solves the deadlock
        pass
```
* Solution B (Structural Redesign): Move the shared classes/functions into a completely independent third module (e.g., models.py or types.py) that both modules can import safely.

### 2. Shadowing / Name Conflicts

* Cause: You named your local file the exact same name as a built-in module or third-party library (e.g., naming your test file math.py or requests.py). Because the current directory is evaluated first in sys.path, Python imports your local file instead of the real library.
* Symptoms: AttributeError: module 'math' has no attribute 'sqrt'
* Solution: Rename your local script file to something unique (e.g., math_test.py).

## 9. Advanced Tip: Namespace Packages
Starting in Python 3.3, you can create a package without an`__init__.py` file. This is known as a Namespace Package.

* Why use it? It allows large companies or open-source projects to split a single package across completely different directories or distinct git repositories.
* Example: Company A can distribute company.core, and Company B can distribute company.auth. When a user installs both, Python seamlessly merges them under the single company name, even though they originated from separate codebases.
