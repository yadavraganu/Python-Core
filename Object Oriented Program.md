# Python’s attribute access hierarchy

1. **Instance Attributes**  
   Looks in the object’s own `__dict__`.

2. **Class Attributes**  
   If not found, checks the class’s `__dict__`.

3. **Parent Classes (MRO)**  
   If still not found, follows the **Method Resolution Order** (MRO) through base classes.

4. **Data Descriptors**  
   If the attribute is a descriptor with `__get__`, `__set__`, or `__delete__`, it takes priority.

5. **`__getattr__`**  
   Called **only if** the attribute wasn’t found above.

6. **`__getattribute__`**  
   Called **first** for every attribute access (even before checking instance/class), unless overridden carefully.

### Tip:
- Use `__getattr__` for fallback behavior.
- Use `__getattribute__` only if you need to intercept **all** attribute access (advanced use).

# **Types of Attributes**

Attributes are variables associated with a class or instance.

### 1. **Instance Attributes**
Defined inside the `__init__` method and tied to a specific object.
```python
class Car:
    def __init__(self, brand):
        self.brand = brand  # instance attribute
```

### 2. **Class Attributes**
Shared across all instances of the class.
```python
class Car:
    wheels = 4  # class attribute
```

### 3. **Private Attributes**
Prefixed with double underscore `__`, not truly private but name-mangled.
```python
class Car:
    def __init__(self):
        self.__engine = "V8"
```

### 4. **Protected Attributes**
Prefixed with a single underscore `_`, meant for internal use.
```python
class Car:
    def __init__(self):
        self._mileage = 10000
```

# **Types of Methods**

Methods are functions defined inside a class.

### 1. **Instance Methods**
Operate on instance data. First parameter is always `self`.
```python
class Car:
    def drive(self):
        print("Driving")
```

### 2. **Class Methods**
Operate on class-level data. First parameter is `cls`. Use `@classmethod` decorator.
```python
class Car:
    count = 0

    @classmethod
    def increment_count(cls):
        cls.count += 1
```

### 3. **Static Methods**
Don’t access instance or class data. Use `@staticmethod` decorator.
```python
class Car:
    @staticmethod
    def honk():
        print("Beep!")
```
# Python Properties
Python's `@property` decorator is used to define methods in a class that can be accessed like regular attributes. This provides a clean interface while maintaining control over data access, validation, and deletion.

### Core Concepts
- **Getter (`@property`)**: Defines a method that behaves like a read-only attribute. It runs when you read the property.
- **Setter (`@<property_name>.setter`)**: Defines a method that runs when you assign a value to the property. It is best used for data validation.
- **Deleter (`@<property_name>.deleter`)**: Defines a method that runs when the attribute is deleted using the `del` keyword. Great for cleanup or resetting data safely.

### Code Example
```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self._salary = salary  # The underscore indicates this is a "protected" variable

    # 1. GETTER
    @property
    def salary(self):
        """Triggers when you do `print(emp.salary)`"""
        return f"${self._salary:,.2f}"

    # 2. SETTER
    @salary.setter
    def salary(self, value):
        """Triggers when you do `emp.salary = value`"""
        if value < 0:
            raise ValueError("Salary cannot be negative!")
        self._salary = value

    # 3. DELETER
    @salary.deleter
    def salary(self):
        """Triggers when you do `del emp.salary`"""
        print(f"Resetting salary record for {self.name}...")
        self._salary = 0


# --- Usage ---
emp = Employee("Bob", 50000)

# Reading the property (Triggers the Getter)
print(emp.salary)  
# Output: $50,000.00

# Modifying the property (Triggers the Setter)
emp.salary = 65000
print(emp.salary)  
# Output: $65,000.00

# Deleting the property (Triggers the Deleter)
del emp.salary     
# Output: Resetting salary record for Bob...

# Verifying the deleted/reset state
print(emp.salary)  
# Output: $0.00
```
