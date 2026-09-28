# Python Modules and Packages

A **module** is a single `.py` file; a **package** is a directory of modules. Code modularity lets you break large applications into smaller, manageable, and reusable pieces.

## 1. Definitions and Architecture

### Modules
A module is a single Python file containing executable code, function definitions, classes, or variables.

* **File Name**: Any file ending in `.py` (e.g., `calculator.py`).
* **Purpose**: Groups related code together to avoid naming conflicts and maximize reusability.

### Packages
A package is a collection of modules organized in a directory structure.

* **Folder Name**: Any directory containing Python files.
* **The `__init__.py` file**: Tells Python that the directory should be treated as a package.
  * It can be completely empty.
  * It runs initialization code when the package is first imported.
  * It can use `__all__` to control what gets exported during wildcard (`*`) imports.

### Typical Directory Structure
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

## 2. The Import Mechanism

Python provides several syntax options to bring external code into your active workspace.

### Standard Import
Imports the entire module. You must use dot (`.`) syntax to access anything inside it.

* **Syntax**: `import package.module`
```python
import shop.billing
# Accessing a function requires the full path
shop.billing.create_invoice()
```

### Specific `from ... import`
Imports specific parts (functions, classes, variables) directly into your namespace. No module-name prefix needed.

* **Syntax**: `from package.module import items`
```python
from shop.billing import create_invoice
create_invoice()
```

### Alias Imports (`as`)
Renames a module or function locally — shortens long names or avoids naming conflicts.

* **Syntax**: `import module as alias`
```python
import shop.delivery.tracking as track
from shop.inventory import get_item_count as check_stock

track.locate_package()
items = check_stock()
```

### Wildcard Imports (`from ... import *`)
Imports everything from a module directly into your namespace.

* **Syntax**: `from module import *`
* **Warning**: Avoid in production code — it causes "namespace pollution" because you can't easily tell where a function came from, and it may silently overwrite existing names.

## 3. Controlling Exports with `__all__`

When someone uses a wildcard import, `__all__` in that module (or the package's `__init__.py`) controls exactly what gets exported.

```python
# Inside shop/billing.py
__all__ = ['create_invoice', 'calculate_tax']  # Only these two will be exported

def create_invoice():
    pass

def calculate_tax():
    pass

def _internal_helper():
    # Hidden by default due to the underscore prefix
    pass

def hidden_bonus_function():
    # Clean code, but excluded because it's not listed in __all__
    pass
```

If another file runs `from shop.billing import *`, it only gets `create_invoice` and `calculate_tax`.

### Defining a Package's Public API in `__init__.py`

`__init__.py` is also where a package decides what its *public surface* looks like. Re-exporting names there lets callers skip the internal file layout:

```python
# shop/__init__.py
from .billing import create_invoice, calculate_tax
from .inventory import get_item_count

__all__ = ["create_invoice", "calculate_tax", "get_item_count"]
__version__ = "1.0.0"
```
```python
# Callers now write this...
from shop import create_invoice
# ...instead of this, and you can reorganize billing.py later without breaking them
from shop.billing import create_invoice
```
Importing any submodule (`import shop.billing`) always runs `shop/__init__.py` **first**, so keep it light: heavy work or many eager imports there slows down every import of the package.

## 4. Checking Import State with `in`

`in` is a plain membership operator — it's never part of import syntax itself, but it's very useful for checking the status and contents of imports.

### Verifying Loaded Modules with `sys.modules`
On import, Python loads the module into memory and caches it in the `sys.modules` dictionary.

```python
import sys

# Check if 'json' has been imported anywhere in the application
if "json" in sys.modules:
    print("The JSON module is cached and ready to use.")
```

### Checking Module Content with `dir()`
`dir()` lists all functions, classes, and variables inside a module — useful for checking a feature exists before using it (e.g., across Python versions).

```python
import math

# Check if the 'factorial' function exists in this version of Python
if "factorial" in dir(math):
    print(math.factorial(5))
```

## 5. Absolute vs. Relative Imports

| Metric | Absolute Imports | Relative Imports |
|---|---|---|
| Path Origin | Always starts from the project root directory | Starts from the current module's position |
| Syntax Style | Explicit full path using words | Uses single (`.`) or double (`..`) dots |
| PEP 8 Status | Preferred for readability and safety | Acceptable for complex, deeply nested packages |
| Execution | Works from anywhere in the application | Only works when executed as part of a package |

### Absolute Imports
Outlines the exact path from the project root to the desired resource.

```python
# Working inside: shop/billing.py
from shop.inventory import get_item_count  # Clear, explicit, and readable
```

### Relative Imports
Detects the target module's location based on where the current file is.

* `.` (one dot) — current directory.
* `..` (two dots) — parent directory (one level up).

```python
# Working inside: shop/billing.py
from .inventory import get_item_count  # Look in the same folder for inventory.py
```
```python
# Working inside: shop/delivery/tracking.py
from ..billing import create_invoice   # Go up to 'shop', then look for billing.py
```

### The Relative Import Trap
Relative imports only work when Python runs your code **as part of a package**. If you `cd` directly into `shop/` and run:

```bash
python billing.py
```

you'll get:
```
ImportError: attempted relative import with no known parent package
```

**The fix:** run from the project root using the module flag (`-m`):
```bash
python -m shop.billing
```

## 6. Module Lifecycle and Resolution

### Module Caching and Re-execution
Python executes a module's code exactly once per process, then caches the resulting module object in `sys.modules`.

* Subsequent imports of the same module don't re-run the file — they read it from the cache.
* To forcefully re-run a module (e.g., live debugging in a shell), use `importlib`:
```python
import importlib
import shop.billing

importlib.reload(shop.billing)  # Force re-executes the file
```

**`reload` gotcha:** it re-executes the module and updates the existing module object in place, but any name you previously bound with `from shop.billing import create_invoice` still points at the **old** function object. Only code that accesses it through the module (`shop.billing.create_invoice()`) sees the new version. This is one reason `import module` is friendlier to live-reloading than `from module import name`.

### How Python Locates Modules (`sys.path`)
On `import xyz`, Python searches an ordered list of directory paths in `sys.path`:

1. **The Current Directory** — the folder containing the script you launched.
2. **PYTHONPATH** — an optional environment variable with extra directory paths.
3. **Standard Library** — built-in locations (where `math`, `os`, `sys` live).
4. **Third-Party Packages (`site-packages`)** — where `pip`-installed packages live.

If nothing matches across all of these, Python raises `ModuleNotFoundError`. You can inspect or extend the search path directly:
```python
import sys
print(sys.path)
sys.path.append("/custom/path")   # ad-hoc addition, resolved at runtime
```

**Precision on the first entry:** `sys.path[0]` is the **directory of the script you ran** (for `python script.py`), the current working directory (for `python -m package.module`), or an empty string in the interactive shell. Editing `sys.path` inside application code is usually a smell; prefer an editable install (`pip install -e .`) so your package is importable the normal way. Python 3.11+ also offers `python -P` / `PYTHONSAFEPATH=1` to stop the script directory (or cwd) from being prepended automatically, which closes off the accidental-shadowing problem described in the troubleshooting section.

### Bytecode Caching — `__pycache__` and `.pyc` files
When a module is imported (not run directly as `__main__`), CPython compiles its source into bytecode and caches it as a `.pyc` file inside a `__pycache__` folder next to the source (e.g., `__pycache__/billing.cpython-311.pyc`).

* On the next import, Python compares the source file's timestamp/hash against the cached bytecode; if unchanged, it skips recompilation and loads the cached bytecode directly — faster startup.
* `.pyc` files are version-specific (tied to the interpreter version in the filename) and are safe to delete; Python regenerates them automatically.
* This caching is purely an import-time optimization — it has no effect on how the *script you actually run* (`python billing.py`) is executed, since the top-level script itself isn't cached this way.

## 7. `__name__ == "__main__"` — Script vs. Import Behavior

Every module has a built-in `__name__` attribute:

* If the file is **run directly** (`python billing.py`), Python sets `__name__ = "__main__"`.
* If the file is **imported** by another module, `__name__` is set to the module's actual name (e.g., `"shop.billing"`).

This is why the following guard is one of the most common patterns in Python:

```python
# billing.py
def create_invoice():
    return "Invoice created"

def main():
    print(create_invoice())

if __name__ == "__main__":
    main()   # only runs when billing.py is executed directly, not when imported
```

**Why it matters (frequent interview question):** without the guard, importing `billing` elsewhere (`import shop.billing`) would also immediately execute `main()` — any test/demo code at module level runs as a side effect of import, which is almost never what you want. The guard lets a file serve double duty as both a reusable module and a standalone script.

Other module-level introspection attributes worth knowing:
```python
print(__name__)      # "__main__" or the module's dotted name
print(__file__)      # the file path this code was loaded from
print(__doc__)        # the module's top-level docstring, if any
print(__package__)    # the package this module belongs to (or None/'' for top-level scripts)
```

## 8. Packaging Your Code — Step-by-Step Guide

Goal: turn the `shop` package into something anyone can `pip install`. Modern packaging is driven by a single `pyproject.toml` file (replacing the older `setup.py`), plus a **build backend** that turns your source into installable files.

**Terminology first (a common source of confusion):**
- **Distribution name**: what you `pip install` (e.g., `my-custom-shop-pkg`), set by `name` in `pyproject.toml`.
- **Import name**: what you `import` (e.g., `shop`), determined by the folder under `src/`.
- They don't have to match, but keeping them close makes life easier. Names must also be **unique on PyPI**, so check availability before committing to one.

### Step 1 — Create the Project Layout

```
my_shared_package/
├── pyproject.toml          # project metadata + build configuration
├── README.md               # shown on the PyPI page
├── LICENSE                 # your license text
├── src/
│   └── shop/               # the importable package
│       ├── __init__.py
│       ├── __main__.py     # optional: enables `python -m shop`
│       └── billing.py
└── tests/
    └── test_billing.py
```

**Why the `src/` layout?** With `src/`, your package is *not* importable from the project root by accident, so tests run against the **installed** package instead of the loose source folder. That catches packaging mistakes (a missing file, a wrong path) before your users do.

### Step 2 — Create and Activate a Virtual Environment

Isolate this project's dependencies so they don't collide with other projects or your system Python:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
python -m pip install --upgrade pip
```

### Step 3 — Write the Package Code

```python
# src/shop/__init__.py
from .billing import create_invoice

__all__ = ["create_invoice"]
__version__ = "1.0.0"
```
```python
# src/shop/billing.py
def create_invoice():
    return "Invoice created"
```

### Step 4 — Write `pyproject.toml`

```toml
[build-system]
requires = ["setuptools>=77.0.0"]        # what pip installs to build your project
build-backend = "setuptools.build_meta"   # which backend does the building

[project]
name = "my-custom-shop-pkg"              # the DISTRIBUTION name (what users pip install)
version = "1.0.0"
description = "An ecommerce utility package"
readme = "README.md"
requires-python = ">=3.10"
license = "MIT"                           # SPDX expression (PEP 639)
authors = [{ name = "Your Name", email = "you@example.com" }]
classifiers = ["Programming Language :: Python :: 3"]
dependencies = ["requests>=2.28.0"]       # runtime dependencies

[project.optional-dependencies]
dev = ["pytest>=8", "build", "twine"]     # installed with: pip install -e ".[dev]"

[project.scripts]
shop-cli = "shop.__main__:main"           # creates a `shop-cli` command on install

[project.urls]
Homepage = "https://github.com/you/my-custom-shop-pkg"

[tool.setuptools.packages.find]
where = ["src"]                           # look for packages inside src/
```

What each part does:
- **`[build-system]`**: names the backend (PEP 517/518). Setuptools is one option; `hatchling`, `flit-core`, and `poetry-core` work through the same table, so swapping backends doesn't change the rest of your file.
- **`[project]`**: standard metadata (PEP 621), understood by every backend and tool.
- **`dependencies`** vs **`optional-dependencies`**: the first is installed for every user; the second is opt-in extras like dev tools.
- **`[project.scripts]`**: turns `shop.__main__:main` into a real command-line program on the user's PATH.
- Version numbers here are illustrative; check current guidance for your backend.

*Optional, single-source the version:* instead of repeating `1.0.0` in two places, declare it dynamic and let setuptools read it from the package:
```toml
[project]
dynamic = ["version"]        # and delete the static `version = ...` line

[tool.setuptools.dynamic]
version = { attr = "shop.__version__" }
```

### Step 5 — Install in Editable Mode and Try It

```bash
pip install -e ".[dev]"
python -c "import shop; print(shop.__version__)"
shop-cli
```
An **editable install** links your source folder into the environment, so code edits take effect immediately with no reinstall. If `import shop` works from *any* directory, your package layout and config are correct.

### Step 6 — Run Your Tests

```bash
pytest
```
Because of the `src/` layout, these tests exercise the installed package, which is exactly what your users will get.

### Step 7 — Build the Distribution Files

```bash
python -m build
```
This creates a `dist/` folder containing two artifacts:

| File | Type | What it is |
|---|---|---|
| `my_custom_shop_pkg-1.0.0.tar.gz` | **sdist** (source distribution) | Your source plus metadata; pip must run the build backend to install it |
| `my_custom_shop_pkg-1.0.0-py3-none-any.whl` | **wheel** (built distribution) | A pre-built zip that pip installs directly, so it's fast and needs no build step |

Reading the wheel filename: `name-version-pythontag-abitag-platformtag`. `py3-none-any` means **pure Python**, working on any OS and any Python 3. Packages with compiled C extensions produce platform-specific wheels instead. Publish **both** files: pip prefers the wheel and falls back to the sdist.

### Step 8 — Inspect and Validate the Artifacts

```bash
python -m zipfile -l dist/*.whl      # list what actually got packaged
tar tf dist/*.tar.gz                  # list the sdist contents
twine check dist/*                    # validates metadata and the README rendering
```
Look for missing files (e.g., data files, which need extra config; see the pitfalls below) *before* uploading anything.

### Step 9 — Test the Built Wheel in a Clean Environment

```bash
python -m venv /tmp/verify-env
source /tmp/verify-env/bin/activate
pip install dist/*.whl
python -c "import shop; print(shop.create_invoice())"
```
This proves the wheel works **without** your source tree or dev environment lying around.

### Step 10 — Publish to TestPyPI First (a Safe Rehearsal)

1. Create accounts on [test.pypi.org](https://test.pypi.org) and [pypi.org](https://pypi.org) (separate accounts) and enable 2FA.
2. Create an **API token** on TestPyPI.
3. Upload:
```bash
twine upload --repository testpypi dist/*
# username: __token__
# password: <your API token, including the pypi- prefix>
```
4. Install it back from TestPyPI to verify:
```bash
pip install --index-url https://test.pypi.org/simple/ \
            --extra-index-url https://pypi.org/simple/ my-custom-shop-pkg
```
The `--extra-index-url` matters: your dependencies (like `requests`) live on the real PyPI, not TestPyPI.

### Step 11 — Publish to the Real PyPI

```bash
twine upload dist/*
```
Then `pip install my-custom-shop-pkg` works for anyone in the world.

**Important:** PyPI **never lets you re-upload a version**, even after deleting it. A mistake means bumping the version number, so the TestPyPI rehearsal is worth the extra minute.

**Better than long-lived tokens:** use **Trusted Publishing**. You register your GitHub repository and workflow with PyPI, and the release workflow gets a short-lived credential automatically through OIDC (typically via the `pypa/gh-action-pypi-publish` action), so there's no secret token to store or leak.

### Step 12 — Releasing Updates

1. Bump the version (follow **semantic versioning**: `MAJOR.MINOR.PATCH`, where a breaking change bumps MAJOR, a new feature bumps MINOR, and a fix bumps PATCH). Pre-releases look like `1.1.0rc1` (PEP 440).
2. Commit and tag the release in git (`git tag v1.1.0`).
3. **Delete the old `dist/` folder** (`rm -rf dist/`), or you'll re-upload stale files.
4. Rebuild (`python -m build`), check (`twine check dist/*`), and upload.

### Tooling Alternatives

The steps above use the standard, backend-agnostic tools (`build` + `twine`), so they work everywhere. All-in-one tools wrap the same workflow: **`uv`** (`uv build`, `uv publish`), **Hatch** (`hatch build`, `hatch publish`), and **Poetry** (`poetry build`, `poetry publish`). Knowing the underlying steps makes any of them easy to pick up.

### Common Packaging Pitfalls

| Symptom | Likely cause and fix |
|---|---|
| `ModuleNotFoundError` after installing your own package | The `[tool.setuptools.packages.find] where = ["src"]` setting is missing or the folder layout doesn't match |
| Works in your repo, breaks for users | You tested against loose source instead of the installed package (the reason for `src/` layout and Step 9) |
| Non-`.py` files (templates, JSON, data) missing from the wheel | They aren't included by default; declare them, e.g. `[tool.setuptools.package-data] shop = ["data/*.json"]` |
| `400 File already exists` on upload | That version was already published; bump the version and rebuild |
| Old code uploaded after a rebuild | Stale files in `dist/`; clear it before building |
| `pip install` name differs from `import` name | Expected: distribution name and import name are separate (`pip install pillow` → `import PIL`) |
| Secrets committed to git | Never commit API tokens; use Trusted Publishing or CI secrets, and keep `dist/` and `.venv/` in `.gitignore` |

## 9. Common Troubleshooting Guide

### Circular Imports

* **Cause**: Module A imports Module B at the top of its file, and Module B imports Module A at the top of *its* file. Nothing deadlocks or freezes. Python simply finds the other module **partially initialized** in `sys.modules`: it exists, but the names defined *after* the import line haven't been created yet.
```python
# --- user.py ---
from .order import Order      # starts executing order.py right here...
class User: pass              # ...so User doesn't exist yet when order.py needs it
```
```python
# --- order.py ---
from .user import User        # user.py is only partially initialized at this point
class Order: pass
```
* **Symptom**: `ImportError: cannot import name 'User' from partially initialized module 'shop.user' (most likely due to a circular import)`. With the plain form (`import shop.user`, then `shop.user.User`) you'd instead get an `AttributeError` at the moment the missing name is accessed.

* **Solution A — Deferred (inline) imports:** move the import inside the function/method that actually needs it.
```python
# --- order.py ---
class Order:
    def assign_to_user(self):
        from .user import User  # inline import avoids the deadlock
        pass
```
* **Solution B — Structural redesign:** move shared classes/functions into an independent third module (e.g., `models.py`) that both sides can safely import.
* **Solution C — `TYPE_CHECKING` guard (when the import is only for type hints):** most circular imports in modern code exist purely for annotations. Import the name only for the type checker, never at runtime:
```python
# --- order.py ---
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:            # False at runtime, True for mypy/Pyright
    from .user import User

class Order:
    def assign_to_user(self, user: User) -> None:   # annotation is never evaluated at runtime
        ...
```

### Shadowing / Name Conflicts

* **Cause**: A local file shares a name with a built-in module or third-party library (e.g., naming a test file `math.py` or `requests.py`). Since the current directory is checked first in `sys.path`, Python imports the local file instead of the real library.
* **Symptom**: `AttributeError: module 'math' has no attribute 'sqrt'`
* **Solution**: Rename the local file to something unique (e.g., `math_test.py`).

## 10. Advanced Tip: Namespace Packages

Since Python 3.3 (PEP 420), you can create a package **without** an `__init__.py` file — a **namespace package**.

* **Why use it?** It lets large companies or open-source projects split a single package across completely different directories or separate git repositories.
* **Example**: Company A distributes `company.core`; Company B distributes `company.auth`. When a user installs both, Python seamlessly merges them under the single `company` namespace, even though they originated from separate codebases.

## 11. Import Style and Ordering (PEP 8)

- Put imports at the **top of the file**, one per line for plain `import` statements.
- Group them in this order, with a blank line between groups: **standard library**, then **third-party**, then **local application** imports.
```python
import os
import sys

import requests

from shop.billing import create_invoice
```
- Prefer **absolute imports**; use explicit relative imports only within a package. (Python 3 removed *implicit* relative imports, where a bare `import sibling` used to find a neighboring file.)
- Avoid `from module import *` outside interactive sessions and package `__init__` re-exports.
- Tools like `isort` and `ruff` enforce this ordering automatically.

## 12. Dynamic and Lazy Imports

### Importing by String Name
When the module to load isn't known until runtime (plugin systems, config-driven code), use `importlib.import_module` instead of the low-level `__import__`:

```python
import importlib

module_name = "shop.billing"
billing = importlib.import_module(module_name)
billing.create_invoice()
```

### Lazy Loading
Imports cost startup time. Two techniques to defer them:
- **Inline import** inside the function that needs it (also useful for breaking cycles, but repeated calls pay a small dictionary lookup, not a full re-import, since the module is cached).
- **Module-level `__getattr__` (PEP 562)** lets a package load a heavy submodule only on first access:
```python
# shop/__init__.py
def __getattr__(name):
    if name == "analytics":
        import importlib
        return importlib.import_module(".analytics", __name__)
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")
```

### Measuring Import Cost
```bash
python -X importtime -c "import shop"
```
Prints how long each import took (self and cumulative), which is the quickest way to find what's making a CLI tool slow to start.

## 13. Making a Package Runnable with `__main__.py`

Adding a `__main__.py` file lets you run the **package itself** with `python -m`:

```
shop/
├── __init__.py
├── __main__.py     # executed by: python -m shop
└── billing.py
```
```python
# shop/__main__.py
from .billing import create_invoice

def main():
    print(create_invoice())

if __name__ == "__main__":
    main()
```
```bash
python -m shop
```
This is the standard way to ship a command-line entry point inside a package, and it runs with the package context set up, so relative imports like `from .billing import ...` work (unlike running a file directly).

## Notes & Gotchas

- **`if __name__ == "__main__":`** is the single most-asked module concept — know both *what* it does and *why* (prevents import-time side effects).
- **Modules are only executed once per process** — top-level code (including print statements, class/function definitions) runs at import time, not each time you reference the module afterward.
- **Circular imports** are usually a design smell — needing an inline import to "fix" one is often a sign two modules are too tightly coupled and share responsibilities that belong in a third module.
- **`__pycache__`/.pyc files** are safe to delete; Python regenerates them and uses source timestamps/hashes to know when to recompile.
- **Wildcard imports (`from x import *`) and `__all__`** are frequently paired questions — know that `__all__` only affects `import *`, not `from module import specific_name`, which always works regardless of `__all__`.
- **A package needs `__init__.py`** for a "regular" package; omitting it deliberately creates a **namespace package** instead — a common point of confusion.
- **`sys.path` order matters** — the current directory is checked before site-packages, which is exactly why local files can accidentally shadow standard-library or third-party modules of the same name.
- **A circular import isn't a deadlock**; it's a *partially initialized module*. `from X import Y` fails with `ImportError` (the name doesn't exist *yet*), while `import X; X.Y` fails with `AttributeError`. Being able to explain that distinction is a strong answer.
- **`TYPE_CHECKING` is the cleanest fix when a cycle exists only for type hints**, since the import never runs at runtime.
- **`import module` vs `from module import name`:** the second binds the object itself, so it goes stale after `importlib.reload()`, and it's also where partial-initialization errors surface.
- **Importing a submodule runs every parent package's `__init__.py` first**, so `import shop.delivery.tracking` executes `shop/__init__.py`, then `shop/delivery/__init__.py`, then `tracking.py`.
- **`python -m package` needs a `__main__.py`**, and running with `-m` (rather than by file path) is what makes relative imports work.
- **Prefer an editable install over editing `sys.path`** to make your own package importable during development.
- **Distribution name vs import name:** `pip install my-custom-shop-pkg` and `import shop` are independent (think `pip install pillow` / `import PIL`).
- **sdist vs wheel:** an sdist is source that needs a build step at install time; a wheel is pre-built and installs directly. Publish both, and pip prefers the wheel.
- **PyPI versions are immutable**: you can never re-upload a version, so rehearse on TestPyPI and bump the version to fix a bad release.
- **`pyproject.toml` separates three concerns:** `[build-system]` (how to build), `[project]` (metadata and dependencies), and `[tool.*]` (backend/tool settings).
