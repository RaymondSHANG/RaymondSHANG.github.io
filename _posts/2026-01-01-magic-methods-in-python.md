---
layout: post
title: "Magic Methods in Python"
subtitle: "with examples"
date: 2026-01-01 11:11:04
header-style: text
catalog: true
author: "Yuan"
tags: [magic methods, dunder methods, double underscore, python]
---
{% include linksref.html %}

# Python Magic Methods (Dunder Methods)
"Magic methods" (also known as **dunder** methods, short for "double underscore") allow you to define how your custom objects behave with built-in Python operations like operators (`+`, `-`, `<`), standard functions (`len()`, `str()`), and iteration loops.

## 1. Magic Methods Cheat Sheet

### Initialization & Construction
These methods control how an object is created and set up.

| Method | Description | Example Trigger |
| :--- | :--- | :--- |
| `__init__(self, ...)` | The constructor. Initializes a new instance. | `obj = MyClass()` |
| `__new__(cls, ...)` | Creates the instance *before* `__init__`. Rare, used for Singletons/Metaclasses. | `obj = MyClass()` |
| `__del__(self)` | The destructor. Called when an object is garbage collected. | `del obj` |

**Example:**
```python
class Book:
    def __init__(self, title, pages):
        self.title = title
        self.pages = pages

b = Book("Python 101", 200)
```

### String Representation
These control how your object looks when printed or inspected.

| Method | Description | Example Trigger |
| :--- | :--- | :--- |
| `__str__(self)` | User-friendly string representation. | `print(obj)` or `str(obj)` |
| `__repr__(self)` | Developer-friendly string (unambiguous, helpful for debugging). | `obj` in console or `repr(obj)` |

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    
    def __str__(self):
        return f"Point at ({self.x}, {self.y})"
    
    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"

p = Point(1, 2)
print(p)      # Output: Point at (1, 2)
print([p])    # Output: [Point(x=1, y=2)]  (Lists use __repr__)
```

### Comparison Operators

| Method | Operator | Description |
| :--- | :--- | :--- |
| `__eq__(self, other)` | `==` | Equality |
| `__lt__(self, other)` | `<` | Less than (Enables sorting!) |
| `__le__(self, other)` | `<=` | Less than or equal |
| `__gt__(self, other)` | `>` | Greater than |
| `__ge__(self, other)` | `>=` | Greater than or equal |
| `__ne__(self, other)` | `!=` | Not equal |

```python
class Score:
    def __init__(self, value):
        self.value = value
    
    def __lt__(self, other):
        # Enables sorting logic
        return self.value < other.value

s1 = Score(10)
s2 = Score(20)
print(s1 < s2)  # True
```

### Arithmetic Operators

| Method | Operator | Description |
| :--- | :--- | :--- |
| `__add__(self, other)` | `+` | Addition |
| `__sub__(self, other)` | `-` | Subtraction |
| `__mul__(self, other)` | `*` | Multiplication |
| `__truediv__(self, other)` | `/` | Division |
| `__floordiv__(self, other)` | `//` | Floor Division |
| `__mod__(self, other)` | `%` | Modulo |
| `__pow__(self, other)` | `**` | Power |

```python
class Wallet:
    def __init__(self, money):
        self.money = money
    
    def __add__(self, other):
        return Wallet(self.money + other.money)

w1 = Wallet(50)
w2 = Wallet(30)
w3 = w1 + w2  # w3.money is 80
```

### Container & Sequence Methods

| Method | Description | Example Trigger |
| :--- | :--- | :--- |
| `__len__(self)` | Returns the length. | `len(obj)` |
| `__getitem__(self, key)` | Access an item by index/key. | `obj[key]` |
| `__setitem__(self, key, value)` | Set an item by index/key. | `obj[key] = value` |
| `__delitem__(self, key)` | Delete an item by index/key. | `del obj[key]` |
| `__contains__(self, item)` | Check membership. | `item in obj` |
| `__iter__(self)` | Returns an iterator object. | `for x in obj:` |

## 2. Understanding __post_init__
`__post_init__` is not a standard built-in Python magic method for regular classes. It is specific to Dataclasses (@dataclass decorator, introduced in Python 3.7).

When you use the @dataclass decorator, Python automatically generates the `__init__` method for you. `__post_init__`is a "hook" that runs immediately after that auto-generated code finishes.

```python
from dataclasses import dataclass

@dataclass
class Rectangle:
    width: float
    height: float
    area: float = 0.0  # We will calculate this automatically

    def __post_init__(self):
        # This runs immediately AFTER the auto-generated __init__
        if self.width < 0 or self.height < 0:
            raise ValueError("Dimensions cannot be negative")
        
        # Calculate derived attribute
        self.area = self.width * self.height

# Usage
r = Rectangle(5, 10)
print(r.area)  # Output: 50.0
```

---
