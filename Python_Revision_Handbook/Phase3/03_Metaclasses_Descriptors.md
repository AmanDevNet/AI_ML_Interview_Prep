# Chapter 3: Metaclasses & Descriptors — "Python's Meta-programming Engine"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **Metaclasses** are the classes that define how classes themselves are built. **Descriptors** are classes that customize how attributes are accessed, set, or deleted. |
| **Why It Exists** | Metaclasses allow validating class structures at definition time before any objects are created. Descriptors allow writing reusable attribute logic (like custom getters/setters) once and sharing it across classes. |
| **Where It Is Used** | ORMs (Django ORM/SQLAlchemy fields mapping), serialization frameworks (Pydantic models), plugin registries, and API validation. |
| **Real-World Analogy** | **Metaclass:** A mold factory. It creates the molds (classes) which are then used to build concrete plastic objects (instances). **Descriptor:** A smart security gate on a drawer. Every time you try to read or put something inside the drawer, the security gate validates your action. |

---

## Think of It This Way

### 1. Metaclasses: The Blueprint of the Blueprint
Normally, a class is a blueprint for creating objects:
`my_dog = Dog()`

But what creates the class `Dog`? In Python, **classes are objects too**. 
The blueprint that creates a class is a **metaclass**. By default, all classes in Python are created by the built-in metaclass `type`.

You can create a class dynamically using `type()` directly:
```python
# type(name, bases, dict)
MyClass = type("MyClass", (object,), {"x": 10})
obj = MyClass()
print(obj.x) # 10
```

By writing a custom metaclass, you can hook into this creation phase to inspect, modify, or validate classes at load time.

### 2. Descriptors: Controlling Attribute Access
If you have multiple attributes in multiple classes that all need validation (e.g. "must be a positive integer"), writing `@property` getters and setters for every single attribute creates tons of duplicate code.
A **descriptor** lets you pack this getter/setter logic into a single reusable class.

---

## Step 1: Writing a Custom Metaclass

A custom metaclass inherits from `type`. It overrides the `__new__` method to intercept class construction:

```python
class ForceUppercaseMeta(type):
    def __new__(cls, name, bases, dct):
        # Intercept attributes and force them to uppercase
        uppercased_dct = {}
        for key, value in dct.items():
            if not key.startswith("__"): # Skip magic attributes
                uppercased_dct[key.upper()] = value
            else:
                uppercased_dct[key] = value
        
        # Call type.__new__ to actually build the class object
        return super().__new__(cls, name, bases, uppercased_dct)

# Assign our custom metaclass
class Test(metaclass=ForceUppercaseMeta):
    x = 10
    def greet(self):
        return "hello"

t = Test()
print(t.X)      # 10 (renamed from x to X!)
print(t.GREET()) # "hello"
```

---

## Step 2: The Descriptor Protocol

A class becomes a descriptor if it implements any of:
* `__get__(self, instance, owner)`
* `__set__(self, instance, value)`
* `__delete__(self, instance)`

```python
class PositiveNumber:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name)

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError(f"{self.name} must be positive!")
        instance.__dict__[self.name] = value

class Product:
    # Set up descriptor instances on the class
    price = PositiveNumber("price")
    stock = PositiveNumber("stock")

p = Product()
p.price = 100 # ✅ Works
# p.stock = -5  # ❌ ValueError: stock must be positive!
```

---

## Code Examples

### Easy Examples
```python
# 1. Simple descriptor (Read-only attribute)
class Constant:
    def __get__(self, instance, owner):
        return 42

class Calculation:
    answer = Constant()

c = Calculation()
print(c.answer) # 42
# c.answer = 10 # AttributeError (cannot set attribute)

# 2. Simple class dynamic creation
Dummy = type("Dummy", (), {"say_hi": lambda self: "hi"})
print(Dummy().say_hi()) # "hi"
```

### Medium Examples
```python
# 3. Dynamic Registry Pattern via Metaclass
# Useful for routing/plugin architectures
PLUGIN_REGISTRY = {}

class PluginMeta(type):
    def __new__(cls, name, bases, dct):
        new_class = super().__new__(cls, name, bases, dct)
        # Register the class if it has a registry_name
        if "registry_name" in dct:
            PLUGIN_REGISTRY[dct["registry_name"]] = new_class
        return new_class

class BasePlugin(metaclass=PluginMeta):
    pass

class AudioPlugin(BasePlugin):
    registry_name = "audio"

class VideoPlugin(BasePlugin):
    registry_name = "video"

print(PLUGIN_REGISTRY) 
# {'audio': <class '__main__.AudioPlugin'>, 'video': <class '__main__.VideoPlugin'>}
```

### Advanced Examples
```python
# 4. Data vs Non-Data Descriptors
# Non-data descriptors only implement __get__. They can be overridden by instance dictionary assignments.
# Data descriptors implement both __get__ and __set__. They ALWAYS take precedence over instance dictionaries.
class NonDataDescriptor:
    def __get__(self, instance, owner):
        return "non-data"

class DataDescriptor:
    def __get__(self, instance, owner):
        return "data"
    def __set__(self, instance, value):
        pass

class TestClass:
    nd = NonDataDescriptor()
    dd = DataDescriptor()

obj = TestClass()
obj.__dict__['nd'] = "override_nd"
obj.__dict__['dd'] = "override_dd"

print(obj.nd) # "override_nd" (instance variable overrode descriptor!)
print(obj.dd) # "data" (data descriptor took precedence over instance variable!)

# 5. Type Validator Descriptor using __set_name__ (Python 3.6+)
# __set_name__ automatically receives the name of the attribute assigned in the owner class.
class TypeChecked:
    def __init__(self, expected_type):
        self.expected_type = expected_type

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name)

    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type.__name__} for {self.name}")
        instance.__dict__[self.name] = value

class Profile:
    age = TypeChecked(int)
    name = TypeChecked(str)

p = Profile()
p.age = 25     # ✅ Works
# p.name = 100 # ❌ TypeError: Expected str for name
```

---

## Common Mistakes & Edge Cases

### 1. Storing instance values inside the descriptor object itself
Never do this:
```python
# ❌ WRONG
class BadDescriptor:
    def __get__(self, instance, owner):
        return self.value # shared across ALL instances!
```
Because descriptors are class attributes, there is only **one** instance of the descriptor shared across all objects. If you write `self.value = value`, every object created will share and overwrite the same value. Always store values inside the target instance's dictionary `instance.__dict__` or use a weakref mapping.

---

## Interview Questions (Top 5)

**Q1: What is a metaclass, and how does Python's `type` act as the default metaclass?**
> A metaclass is a class whose instances are classes. In Python, class definitions are executed to create class objects. By default, `type` is the metaclass that builds class objects. It takes the class name, parent bases, and the class namespace dictionary, returning a new class object. Custom metaclasses inherit from `type` to intercept this phase.

**Q2: What is the difference between `__new__` and `__init__` in custom metaclasses?**
> * `__new__` is called *before* the class object is created. It receives the raw class name, parent bases, and namespace dict. It is used to inspect or modify the class structure (e.g., modifying class fields or checking constraints) before the class exists.
> * `__init__` is called *after* the class object has been created. It is used to initialize the class metadata once the class structure is locked in memory.

**Q3: What is the difference between a Data Descriptor and a Non-Data Descriptor?**
> * **Data Descriptor:** Implements both `__get__` and `__set__` (or `__delete__`). It always takes precedence over the instance dictionary. If you assign `obj.field = value`, it triggers `__set__` rather than writing to `obj.__dict__`.
> * **Non-Data Descriptor:** Implements only `__get__`. It can be overridden by assignments. If you set `obj.field = value`, it creates an instance variable in `obj.__dict__`, masking the descriptor.

**Q4: How does the `__set_name__` method in descriptors prevent attribute name duplication?**
> Prior to Python 3.6, you had to pass the name of the class attribute to the descriptor's constructor explicitly (e.g. `price = PositiveNumber("price")`). From 3.6 onwards, descriptors can implement `__set_name__(self, owner, name)`. When the owner class is compiled, Python automatically calls this method, passing the owner class and the variable name (`"price"`), allowing the descriptor to configure itself dynamically without manual parameters.

**Q5: When should you use a Class Decorator instead of a Metaclass?**
> Class decorators (`@decorator` on top of a class) are simpler because they take a constructed class object, modify it, and return it. They are ideal for simple post-creation additions (like registering classes or adding simple mixin attributes). Metaclasses should be reserved when you need to enforce class interfaces, dynamically alter parent bases, or hook into the creation phase before the class object is built.

---

*← [Previous: Chapter 2 (Async Programming)](02_Async_Programming.md) | [Next: Chapter 4 (Typing, Protocols & Pydantic) →](04_Typing_Protocols_Pydantic.md)*
