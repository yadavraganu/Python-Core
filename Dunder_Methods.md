# Attribute Access
These methods control how attributes are accessed, set, and deleted on objects.
### `__getattr__(self, name)`:
__Description:__ Called when an attribute name is not found in the usual places (instance dictionary, class, parent classes). Useful for dynamic attribute generation or handling misspelled attributes.

__Raises:__ AttributeError if the attribute cannot be handled.
### `__setattr__(self, name, value)`:
__Description:__ Called whenever an attribute is assigned (e.g., obj.name = value). This method is called unconditionally for all attribute assignments. Careful implementation is crucial to avoid infinite recursion (e.g., directly assigning self.name = value inside `__setattr__`).

__Common Usage:__ Use`object.__setattr__(self, name, value)` to set attributes without triggering `__setattr__` recursively.
### `__delattr__(self, name`):
__Description:__ Called when an attribute is deleted (e.g., del obj.name).

__Common Usage:__ Use `object.__delattr__(self, name)` to delete attributes without triggering `__delattr__` recursively.
### `__getattribute__(self, name)`:
__Description:__ Called unconditionally for all attribute accesses (e.g., obj.name). This is even before looking up the attribute in the instance's dictionary. Overriding this can be tricky and lead to infinite recursion if not handled carefully.

__Common Usage:__ Use `object.__getattribute__(self, name)` to get attributes without triggering `__getattribute__` recursively.

__Note:__ If `__getattribute__` is implemented, `__getattr__` will only be called if `__getattribute__` explicitly raises an AttributeError.
### `__dir__(self)`:
__Description:__ Called by dir() on an object to return a list of valid attributes for that object.

__Returns:__ A list of strings.
```python
class SimpleAttributeDemo:
    def __init__(self, value):
        object.__setattr__(self, 'stored_value', value)
        object.__setattr__(self, 'my_id', id(self))

    def __getattribute__(self, name):
        print(f"--> __getattribute__ called for: '{name}'")
        
        if name in ['stored_value', 'my_id']:
            return object.__getattribute__(self, name)
        
        try:
            result = object.__getattribute__(self, name)
            return result
        except AttributeError:
            raise

    def __getattr__(self, name):
        print(f"--> __getattr__ called for: '{name}' (fallback)")
        if name == "dynamic_attr":
            return "Hello from dynamic_attr!"
        else:
            raise AttributeError(f"'{type(self).__name__}' object has no attribute '{name}'")

    def __setattr__(self, name, value):
        print(f"--> __setattr__ called for: '{name}' = '{value}'")
        object.__setattr__(self, name, value)

    def __delattr__(self, name):
        print(f"--> __delattr__ called for: '{name}'")
        if name == 'stored_value':
            raise AttributeError(f"Deletion of '{name}' is not allowed.")
        object.__delattr__(self, name)

# --- DEMONSTRATION ---

print("\n### Part 1: Object Creation and Initial State ###")
# Output:
# ### Part 1: Object Creation and Initial State ###

my_obj = SimpleAttributeDemo(123)
# Execution enters SimpleAttributeDemo.__init__(my_obj, 123)
# Calls object.__setattr__(my_obj, 'stored_value', 123)
# Calls object.__setattr__(my_obj, 'my_id', id(my_obj)) (e.g., 140722330752560)

print(f"Initial stored_value: {my_obj.stored_value}")
# Execution:
# my_obj.stored_value triggers __getattribute__('stored_value')
# --> __getattribute__ called for: 'stored_value'
# Inside __getattribute__, 'stored_value' is in ['stored_value', 'my_id'], so it calls object.__getattribute__(my_obj, 'stored_value')
# Returns 123
# Output:
# Initial stored_value: 123

print(f"Object ID: {my_obj.my_id}")
# Execution:
# my_obj.my_id triggers __getattribute__('my_id')
# --> __getattribute__ called for: 'my_id'
# Inside __getattribute__, 'my_id' is in ['stored_value', 'my_id'], so it calls object.__getattribute__(my_obj, 'my_id')
# Returns the ID (e.g., 140722330752560)
# Output:
# Object ID: 140722330752560

print("-" * 40)
# Output:
# ----------------------------------------

print("\n### Part 2: Reading Attributes ###")
# Output:
#
# ### Part 2: Reading Attributes ###

print("\n--- Reading an existing attribute 'my_data' ---")
# Output:
#
# --- Reading an existing attribute 'my_data' ---

my_obj.my_data = "Hello World"
# Execution:
# my_obj.my_data = "Hello World" triggers __setattr__('my_data', "Hello World")
# --> __setattr__ called for: 'my_data' = 'Hello World'
# Calls object.__setattr__(my_obj, 'my_data', "Hello World")
# Output:
# --> __setattr__ called for: 'my_data' = 'Hello World'

print(f"Reading my_obj.my_data: {my_obj.my_data}")
# Execution:
# my_obj.my_data triggers __getattribute__('my_data')
# --> __getattribute__ called for: 'my_data'
# Inside __getattribute__, 'my_data' is NOT in ['stored_value', 'my_id'].
# Tries object.__getattribute__(my_obj, 'my_data')
# This succeeds, returns "Hello World"
# Output:
# --> __getattribute__ called for: 'my_data'
# Reading my_obj.my_data: Hello World

print("\n--- Reading a dynamic attribute 'dynamic_attr' ---")
# Output:
#
# --- Reading a dynamic attribute 'dynamic_attr' ---

print(f"Reading my_obj.dynamic_attr: {my_obj.dynamic_attr}")
# Execution:
# my_obj.dynamic_attr triggers __getattribute__('dynamic_attr')
# --> __getattribute__ called for: 'dynamic_attr'
# Inside __getattribute__, 'dynamic_attr' is NOT in ['stored_value', 'my_id'].
# Tries object.__getattribute__(my_obj, 'dynamic_attr')
# This FAILS, raises AttributeError.
# Python then calls __getattr__('dynamic_attr')
# --> __getattr__ called for: 'dynamic_attr' (fallback)
# Inside __getattr__, 'dynamic_attr' is handled, returns "Hello from dynamic_attr!"
# Output:
# --> __getattribute__ called for: 'dynamic_attr'
# --> __getattr__ called for: 'dynamic_attr' (fallback)
# Reading my_obj.dynamic_attr: Hello from dynamic_attr!

print("\n--- Reading a truly non-existent attribute 'non_existent_key' ---")
# Output:
#
# --- Reading a truly non-existent attribute 'non_existent_key' ---

try:
    print(my_obj.non_existent_key)
    # Execution:
    # my_obj.non_existent_key triggers __getattribute__('non_existent_key')
    # --> __getattribute__ called for: 'non_existent_key'
    # Inside __getattribute__, 'non_existent_key' is NOT in ['stored_value', 'my_id'].
    # Tries object.__getattribute__(my_obj, 'non_existent_key')
    # This FAILS, raises AttributeError.
    # Python then calls __getattr__('non_existent_key')
    # --> __getattr__ called for: 'non_existent_key' (fallback)
    # Inside __getattr__, 'non_existent_key' is not 'dynamic_attr', so it raises AttributeError.
    # The 'try...except' block catches this.
except AttributeError as e:
    print(f"Caught expected error: {e}")
# Output:
# --> __getattribute__ called for: 'non_existent_key'
# --> __getattr__ called for: 'non_existent_key' (fallback)
# Caught expected error: 'SimpleAttributeDemo' object has no attribute 'non_existent_key'

print("-" * 40)
# Output:
# ----------------------------------------

print("\n### Part 3: Writing Attributes ###")
# Output:
#
# ### Part 3: Writing Attributes ###

print("\n--- Setting a new attribute 'new_value' ---")
# Output:
#
# --- Setting a new attribute 'new_value' ---

my_obj.new_value = 456
# Execution:
# my_obj.new_value = 456 triggers __setattr__('new_value', 456)
# --> __setattr__ called for: 'new_value' = '456'
# Calls object.__setattr__(my_obj, 'new_value', 456)
# Output:
# --> __setattr__ called for: 'new_value' = '456'

print(f"Verify new_value: {my_obj.new_value}")
# Execution:
# my_obj.new_value triggers __getattribute__('new_value')
# --> __getattribute__ called for: 'new_value'
# Inside __getattribute__, 'new_value' is NOT in ['stored_value', 'my_id'].
# Tries object.__getattribute__(my_obj, 'new_value')
# This succeeds, returns 456
# Output:
# --> __getattribute__ called for: 'new_value'
# Verify new_value: 456

print("-" * 40)
# Output:
# ----------------------------------------

print("\n### Part 4: Deleting Attributes ###")
# Output:
#
# ### Part 4: Deleting Attributes ###

print(f"\n--- Deleting 'my_data' ---")
# Output:
#
# --- Deleting 'my_data' ---

print(f"Before deletion: {my_obj.my_data}")
# Execution:
# my_obj.my_data triggers __getattribute__('my_data')
# --> __getattribute__ called for: 'my_data'
# Tries object.__getattribute__(my_obj, 'my_data'), succeeds, returns "Hello World"
# Output:
# --> __getattribute__ called for: 'my_data'
# Before deletion: Hello World

del my_obj.my_data
# Execution:
# del my_obj.my_data triggers __delattr__('my_data')
# --> __delattr__ called for: 'my_data'
# 'my_data' is not 'stored_value'.
# Calls object.__delattr__(my_obj, 'my_data')
# Output:
# --> __delattr__ called for: 'my_data'

try:
    print(my_obj.my_data) # Access after deletion
    # Execution:
    # my_obj.my_data triggers __getattribute__('my_data')
    # --> __getattribute__ called for: 'my_data'
    # Tries object.__getattribute__(my_obj, 'my_data'), fails (AttributeError)
    # Python then calls __getattr__('my_data')
    # --> __getattr__ called for: 'my_data' (fallback)
    # Inside __getattr__, 'my_data' is not 'dynamic_attr', so it raises AttributeError.
    # The 'try...except' block catches this.
except AttributeError as e:
    print(f"Caught error: {e}")
# Output:
# --> __getattribute__ called for: 'my_data'
# --> __getattr__ called for: 'my_data' (fallback)
# Caught error: 'SimpleAttributeDemo' object has no attribute 'my_data'

print("\n--- Attempting to delete 'stored_value' (should be prevented) ---")
# Output:
#
# --- Attempting to delete 'stored_value' (should be prevented) ---

try:
    del my_obj.stored_value
    # Execution:
    # del my_obj.stored_value triggers __delattr__('stored_value')
    # --> __delattr__ called for: 'stored_value'
    # 'stored_value' IS 'stored_value'. Raises AttributeError.
    # The 'try...except' block catches this.
except AttributeError as e:
    print(f"Caught error: {e}")
# Output:
# --> __delattr__ called for: 'stored_value'
# WARNING: Cannot delete 'stored_value'!
# Caught error: Deletion of 'stored_value' is not allowed.

print("-" * 40)
# Output:
# ----------------------------------------
```
