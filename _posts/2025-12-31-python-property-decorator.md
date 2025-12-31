---
layout: post
title: "Python Property Decorator"
subtitle: "explained with examples"
date: 2025-12-31 12:31:21
header-style: text
catalog: true
author: "Yuan"
tags: [decorator, python, @,]
---
{% include linksref.html %}
In Python Class, @ decorator with **@property** is used to implement encapsulation, meaning that methods are disguised as attributes, so that when the user reads or modifies the data, additional checks or cleanup logic are automatically executed.

There are 3 decorators, usually defined together: **Getter (@property), Setter (@z.setter), and Deleter (@z.deleter)**. **@property** needs to be defined first

```python
class Test:
    def __init__(self):
        # Initialize internal variable _z
        self._z = 0

    # 1. Getter: Defines logic when reading z
    @property
    def z(self):
        print(" -> [Getter] Reading value of z...")
        # Error handling in case _z was already deleted
        if not hasattr(self, '_z'):
            return "z does not exist (has been deleted)"
        return self._z

    # 2. Setter: Defines logic when modifying z
    # Note: Must use @z.setter, and function name must be 'z'
    @z.setter
    def z(self, value):
        print(f" -> [Setter] Setting z to: {value}")
        if value < 0:
            print("    Warning: Negative values not allowed. Resetting to 0.")
            self._z = 0
        else:
            self._z = value

    # 3. Deleter: Defines logic when deleting z
    # Note: Must use @z.deleter, and function name must be 'z'
    @z.deleter
    def z(self):
        print(" -> [Deleter] Deleting internal variable _z ...")
        # Perform the actual deletion or cleanup logic
        del self._z

# ==========================================
# Test Execution
# ==========================================

print("--- 1. Initialize Object ---")
t = Test()

print("\n--- 2. Test Read (Getter) ---")
print(f"Current value of z: {t.z}") 

print("\n--- 3. Test Write (Setter) ---")
t.z = 100       # Normal assignment
t.z = -50       # Testing the logic inside the Setter

print("\n--- 4. Test Delete (Deleter) ---")
del t.z         # This triggers the @z.deleter function

print("\n--- 5. Read After Delete ---")
print(f"Current value of z: {t.z}")
```

Output will be like:
```text
--- 1. Initialize Object ---

--- 2. Test Read (Getter) ---
 -> [Getter] Reading value of z...
Current value of z: 0

--- 3. Test Write (Setter) ---
 -> [Setter] Setting z to: 100
 -> [Setter] Setting z to: -50
    Warning: Negative values not allowed. Resetting to 0.

--- 4. Test Delete (Deleter) ---
 -> [Deleter] Deleting internal variable _z ...

--- 5. Read After Delete ---
 -> [Getter] Reading value of z...
Current value of z: z does not exist (has been deleted)
```


### Key Takeaways
- ***Naming Consistency***: All three methods (getter, setter, deleter) must have the exact same name (in this case, z).
- ***Order Matters***: You must define the ```@property``` (getter) first. The ```@z.setter``` and ```@z.deleter``` rely on the property already existing.
- ***Triggers***:

    ```t.z``` (accessing) $\rightarrow$ triggers @property

    ```t.z = 100``` (assigning) $\rightarrow$ triggers ```@z.setter```

    ```del t.z``` (deleting) $\rightarrow$ triggers ```@z.deleter```

- ***Major Benefits of Using ```@property```***:
    **Encapsulation**: It hides internal implementation details (like _z) while providing a clean, simple public interface (z).

    **Backward Compatibility (API Stability)**: You can start with a simple public variable (e.g., self.z). Later, if you need to add logic (like validation), you can switch to @property without breaking existing code. Users still type t.z, but now it runs your function behind the scenes.

    **Data Validation**: The setter allows you to reject bad data (e.g., preventing negative numbers or incorrect types) before it ever touches your internal variables.

    **Computed Attributes**: You can expose a value that doesn't actually exist in memory but is calculated on the fly (e.g., area calculated from width and height), while to the user, it looks like a static variable.

# Other Python Decorators

Decorators are one of Python's most powerful features, allowing you to modify the behavior of functions or classes without changing their source code. Here is a quick reference guide to the most commonly used decorators.

| Decorator | Module | Primary Benefit | Use Case |
| :--- | :--- | :--- | :--- |
| **`@dataclass`** | `dataclasses` | **Productivity** | Auto-generates boilerplate code like `__init__`, `__repr__`, and `__eq__`. Great for data storage classes. |
| **`@lru_cache`** | `functools` | **Performance** | Caches the results of function calls. Ideal for recursive functions or expensive computations. |
| **`@classmethod`** | *(Built-in)* | **Factory Patterns** | Defines a method that is bound to the *class* rather than the instance. Receives `cls` as the first argument. |
| **`@staticmethod`** | *(Built-in)* | **Code Organization** | Defines a utility method that doesn't need access to `self` or `cls`. logically groups functions within a class. |
| **`@property`** | *(Built-in)* | **Encapsulation** | Allows you to access a method like an attribute. Enables getter/setter logic and computed attributes. |
| **`@abstractmethod`** | `abc` | **Enforcement** | Forces subclasses to implement a specific method. Essential for building strict interfaces in OOP. |
| **`@wraps`** | `functools` | **Best Practice** | Used when writing *custom* decorators. It ensures the decorated function keeps its original name and docstring. |


## Quick Examples
### @classmethod vs @staticmethod
```python
class DateUtil:
    @classmethod
    def from_string(cls, date_str):
        # Factory: Creates a new instance from a string
        day, month, year = map(int, date_str.split('-'))
        return cls(day, month, year)

    @staticmethod
    def is_valid_year(year):
        # Utility: Just checks logic, doesn't need class info
        return 1900 < year < 3000
```


## 1. @dataclass
**Definition:** Automatically generates special methods (`__init__`, `__repr__`, `__eq__`, etc.) for classes that exist primarily to store data.

```python
from dataclasses import dataclass

# --- Define ---
@dataclass
class InventoryItem:
    name: str
    price: float
    quantity: int = 0

    def total_cost(self):
        return self.price * self.quantity

# --- Call ---
item1 = InventoryItem("Widget", 10.0, 5)
item2 = InventoryItem("Widget", 10.0, 5)

# --- Result ---
print(item1)          # Output: InventoryItem(name='Widget', price=10.0, quantity=5)
print(item1 == item2) # Output: True (Standard classes would return False here)
```

## 2. @lru_cache
**Definition:** "Least Recently Used Cache". It stores the result of function calls. If the function is called again with the same arguments, it returns the stored result instantly instead of recalculating.
```python
from functools import lru_cache
import time

# --- Define ---
@lru_cache(maxsize=None)
def slow_square(n):
    time.sleep(1) # Simulating a heavy task (1 second delay)
    return n * n

# --- Call ---
print("First call (calculating)...")
start = time.time()
print(slow_square(5)) 

print("Second call (cached)...")
start2 = time.time()
print(slow_square(5)) 

# --- Result ---
# Output 1: 25 (Takes ~1.0 seconds)
# Output 2: 25 (Takes ~0.0 seconds - Instant!)
```

## 3. @classmethod
**Definition**: Defines a method that operates on the class itself rather than an instance. The first argument is always ```cls```. Often used for "Factory methods" to create objects in different ways.
```python
class Pizza:
    def __init__(self, ingredients):
        self.ingredients = ingredients

    # --- Define ---
    @classmethod
    def margherita(cls):
        return cls(['mozzarella', 'tomatoes'])

    @classmethod
    def pepperoni(cls):
        return cls(['mozzarella', 'tomatoes', 'pepperoni'])

# --- Call ---
my_pizza = Pizza.margherita()

# --- Result ---
print(my_pizza.ingredients) 
# Output: ['mozzarella', 'tomatoes']
```
## 4. @staticmethod
**Definition:** Defines a method that belongs to a class logically but does not access any class (```cls```) or instance (self) data. It behaves like a regular function sitting inside a class namespace.
```python
class MathUtils:
    # --- Define ---
    @staticmethod
    def is_even(n):
        return n % 2 == 0

# --- Call ---
# You don't need to instantiate the class to use it
check = MathUtils.is_even(10)

# --- Result ---
print(check) 
# Output: True
```

## 5. @abstractmethod
**Definition:** Enforces a rule that any subclass must implement this method. If they don't, Python will raise an error when you try to create an object.
```python
from abc import ABC, abstractmethod

# --- Define ---
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Square(Shape):
    def __init__(self, side):
        self.side = side
    
    # We MUST implement area(), otherwise Square cannot be used
    def area(self):
        return self.side * self.side

# --- Call ---
s = Square(4)

# --- Result ---
print(s.area()) 
# Output: 16
# Note: Trying to do `x = Shape()` would raise TypeError.
```

## 6. @wraps
**Definition**: A helper used only when you are writing your own custom decorators. It copies the metadata (name, docstring) of the original function to the new wrapper function, so debugging remains easy.

If you ever write your own custom decorators, this is mandatory. Without it, your decorated function loses its identity (its name and docstring are replaced by the wrapper's name).

```python
from functools import wraps

# --- Define ---
def uppercase_decorator(func):
    @wraps(func) # <--- Preserves the identity of the function below
    def wrapper():
        """Wrapper function docstring"""
        result = func()
        return result.upper()
    return wrapper

@uppercase_decorator
def greet():
    """Returns a greeting."""
    return "hello world"

# --- Call ---
print(greet())
print(f"Function name: {greet.__name__}")
print(f"Docstring: {greet.__doc__}")

# --- Result ---
# Output: HELLO WORLD
# Output: Function name: greet  (Without @wraps, this would say 'wrapper')
# Output: Docstring: Returns a greeting.
```

---
