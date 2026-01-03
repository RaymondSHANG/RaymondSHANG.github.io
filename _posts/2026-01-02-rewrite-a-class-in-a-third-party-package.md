---
layout: post
title: "Rewrite A Class In A Third party Package"
subtitle: "From Runtime Replacement To Classpath Shadowing"
date: 2026-01-02 23:29:05
header-style: text
catalog: true
author: "Yuan"
tags: [python, ]
---
{% include linksref.html %}

# How to Rewrite a Class in a Third-Party Package

You can rewrite a class in a third-party package. The strategy you choose depends heavily on your programming language (dynamic vs. static) and the package architecture.

The core challenge isn't writing the new class; it is **forcing the internal logic of the package to use your new class** instead of the original one (e.g., when the package calls `new BadClass()` internally).

Here are the four standard strategies.

---

## 1. Monkey Patching (Runtime Replacement)
**Best for:** Dynamic languages (Python, JavaScript/TypeScript, Ruby).

Monkey patching involves swapping out the class definition in memory at runtime *before* the package uses it.

* **How it works:** Import the package, then explicitly overwrite the class symbol with your custom class.
* **Result:** When the package's internal code calls `OriginalClass()`, it unknowingly executes `YourNewClass()`.

### Python Example
```python
import third_party_pkg.core

# 1. Define your replacement
class BetterClass:
    def __init__(self):
        print("This is the new logic!")

# 2. Overwrite the package's reference to the class
third_party_pkg.core.OriginalClass = BetterClass

# 3. Execution
# When logic inside the package runs: obj = OriginalClass()
# It now actually creates an instance of BetterClass.
```

## 2. Dependency Injection / Subclassing
Best for: Well-designed libraries in any language (Java, C#, TS).

If the library follows "Inversion of Control" principles, it won't hard-code new ClassName(). Instead, it accepts a class definition or instance via configuration.

* **How it works**: Create a class that inherits from the original (overriding specific methods) and pass it into the library's initialization function.

* **Result**: The library uses the instance you provided.

### TypeScript/JS Example
```javascript
import { BadClass, Processor } from 'library';

// 1. Extend the original class
class GoodClass extends BadClass {
    save() {
        console.log("Custom save logic executing...");
    }
}

// 2. Inject it via config
const proc = new Processor({
    storageDriver: new GoodClass() 
});
```

## 3. Classpath Shadowing (The "Ghost" Class)
Best for: Compiled/Static languages (Java, Scala) or specific build tools.

This relies on the order in which the compiler or runtime loads files (classpath precedence).

* **How it works:** Create a file in your own source folder with the exact same directory structure and package name as the original class.

* **Result:** Because local source code usually takes precedence over external JARs/dependencies in the load order, the runtime loads your file and ignores the one inside the library.

### Java Example
    If the target class is com.vendor.lib.utils.Helper:

    Create src/main/java/com/vendor/lib/utils/Helper.java in your project.

    Paste the original code into your file.

    Modify the logic you dislike.

    Compile. The JVM will load your class instead of the JAR version.

## 4. Forking (Vendoring)
Best for: Massive changes or Open Source libraries.

If the changes are complex, it is safer to own the code entirely.

* **How it works:** Clone the repository, make changes, and point your dependency manager to your specific Git URL or local folder.

* **Result:** You are now the maintainer of that specific version.

### package.json Example (Node.js)
```json
{
  "dependencies": {
    "my-library": "git+[https://github.com/my-username/my-forked-library.git](https://github.com/my-username/my-forked-library.git)"
  }
}
```

## Recommendation
    Check for Config: Always look for Dependency Injection options first.

    Monkey Patch: Good for quick fixes in dynamic languages.

    Shadow: Good for static languages if you can't fork.

    Fork: The nuclear option; use only when necessary.

### Risk Comparison Matrix

| Method | Risk Level | The Danger |
| :--- | :--- | :--- |
| **Dependency Injection** | Low | Requires the library author to have written flexible code. |
| **Forking** | Medium | You stop getting official updates or security patches unless you manually merge them. |
| **Shadowing** | High | If the library updates and adds a new method to that class, your shadowed version won't have it, causing a crash. |
| **Monkey Patching** | High | Highly fragile. If the library changes internal variable names or import paths, your patch will silently fail. |
---
