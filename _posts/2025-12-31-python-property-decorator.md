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
---
