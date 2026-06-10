# Chapter 3: Polymorphism — "Same Action, Different Behavior"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Polymorphism means the **same method name** does **different things** depending on which object calls it. |
| **Why It Exists** | Write code that works with ANY type, as long as it has the right methods. Your code becomes flexible and extensible. |
| **Where It Is Used** | Every `for` loop (iteration protocol), `print()` (calls `__str__`), `len()` (calls `__len__`), `sorted()` (calls `__lt__`), ML frameworks (model.fit()). |
| **Real-World Analogy** | "Press the power button" works on a TV, a laptop, and a phone — same action, different result for each device. |

---

## Think of It This Way

Imagine a function called `make_sound()`. 

If you call it on a Dog → "Woof!"  
If you call it on a Cat → "Meow!"  
If you call it on a Duck → "Quack!"  

You didn't write three different function names (`make_dog_sound`, `make_cat_sound`...). You used ONE name (`make_sound`), and each object did its own thing. **That's polymorphism.**

---

## Step 1: Polymorphism Through Method Overriding

The most common type. Child classes override a parent's method:

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

class Cow(Animal):
    def speak(self):
        return "Moo!"

# Polymorphism in action — same method, different behavior
animals = [Dog("Buddy"), Cat("Kitty"), Cow("Bessie"), Dog("Max")]

for animal in animals:
    print(f"{animal.name}: {animal.speak()}")
# Buddy: Woof!
# Kitty: Meow!
# Bessie: Moo!
# Max: Woof!
```

The `for` loop doesn't care if it's a Dog, Cat, or Cow. It just calls `.speak()`. Each object knows how to handle it.

---

## Step 2: Duck Typing — Python's Secret Weapon

Python doesn't care about the **type** of an object. It only cares about **what methods** the object has.

> "If it walks like a duck and quacks like a duck, it IS a duck."

```python
# These classes have NO common parent — they're totally unrelated!
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

class Robot:
    def speak(self):
        return "Beep boop!"

class Rock:
    pass   # No speak() method

# This function works with ANY object that has a speak() method
def make_it_talk(thing):
    return thing.speak()

print(make_it_talk(Dog()))    # Woof!
print(make_it_talk(Cat()))    # Meow!
print(make_it_talk(Robot()))  # Beep boop!
# make_it_talk(Rock())        # ❌ AttributeError — Rock doesn't have speak()
```

**No shared parent needed.** No interface declaration. No type checking. If the method exists, it works. That's duck typing.

### Where You See Duck Typing Every Day

```python
# len() works on ANYTHING that has __len__
print(len([1, 2, 3]))        # 3 (list)
print(len("hello"))           # 5 (string)
print(len({"a": 1, "b": 2})) # 2 (dict)

# for loop works on ANYTHING that has __iter__
for x in [1, 2, 3]: pass      # list
for x in "hello": pass         # string
for x in range(5): pass        # range
for x in {"a": 1}: pass        # dict

# + works on ANYTHING that has __add__
print(1 + 2)             # 3 (int addition)
print("hello" + " world") # hello world (string concatenation)
print([1, 2] + [3, 4])   # [1, 2, 3, 4] (list concatenation)
```

---

## Step 3: Polymorphism with Built-in Functions

```python
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add(self, item):
        self.items.append(item)

    def __len__(self):
        """Now len() works on our cart!"""
        return len(self.items)

    def __contains__(self, item):
        """Now 'in' operator works on our cart!"""
        return item in self.items

    def __iter__(self):
        """Now for loops work on our cart!"""
        return iter(self.items)

cart = ShoppingCart()
cart.add("apple")
cart.add("banana")
cart.add("cherry")

print(len(cart))            # 3        ← __len__
print("banana" in cart)     # True     ← __contains__
for item in cart:           #          ← __iter__
    print(f"  - {item}")
```

By adding these magic methods, our custom class works with Python's built-in syntax just like lists and dicts do. That's polymorphism.

---

## Step 4: Operator Overloading

You can make `+`, `-`, `==`, `<` work with your own classes:

```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount = amount
        self.currency = currency

    def __add__(self, other):
        """What happens when you do money1 + money2"""
        if self.currency != other.currency:
            raise ValueError(f"Can't add {self.currency} and {other.currency}")
        return Money(self.amount + other.amount, self.currency)

    def __sub__(self, other):
        """What happens when you do money1 - money2"""
        if self.currency != other.currency:
            raise ValueError(f"Can't subtract different currencies")
        return Money(self.amount - other.amount, self.currency)

    def __eq__(self, other):
        """What happens when you do money1 == money2"""
        return self.amount == other.amount and self.currency == other.currency

    def __lt__(self, other):
        """What happens when you do money1 < money2"""
        return self.amount < other.amount

    def __repr__(self):
        return f"${self.amount} {self.currency}"

a = Money(100)
b = Money(50)
c = Money(100)

print(a + b)      # $150 USD
print(a - b)      # $50 USD
print(a == c)     # True (same amount and currency)
print(b < a)      # True (50 < 100)
print(sorted([a, b, c]))  # [$50 USD, $100 USD, $100 USD]
```

### Operator → Magic Method Reference

| What You Write | What Python Calls |
|---------------|-------------------|
| `a + b` | `a.__add__(b)` |
| `a - b` | `a.__sub__(b)` |
| `a * b` | `a.__mul__(b)` |
| `a / b` | `a.__truediv__(b)` |
| `a == b` | `a.__eq__(b)` |
| `a != b` | `a.__ne__(b)` |
| `a < b` | `a.__lt__(b)` |
| `a <= b` | `a.__le__(b)` |
| `a > b` | `a.__gt__(b)` |
| `len(a)` | `a.__len__()` |
| `str(a)` | `a.__str__()` |
| `a[key]` | `a.__getitem__(key)` |
| `a(x)` | `a.__call__(x)` |
| `x in a` | `a.__contains__(x)` |

---

## Step 5: `__call__` — Making Objects Act Like Functions

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, value):
        """Now the object can be called like a function!"""
        return value * self.factor

double = Multiplier(2)
triple = Multiplier(3)

# Using the object like a function
print(double(5))     # 10
print(double(100))   # 200
print(triple(7))     # 21

# Check if something is callable
print(callable(double))  # True
print(callable(42))      # False
```

**Where `__call__` is used in real life:**
- PyTorch neural network layers use `__call__` for the forward pass
- Decorators can be implemented as classes with `__call__`
- Function-like objects that need to remember state between calls

---

## Step 6: Full Example — Shape Calculator

```python
import math

class Shape:
    def area(self):
        raise NotImplementedError("Each shape must implement area()")

    def perimeter(self):
        raise NotImplementedError("Each shape must implement perimeter()")

    def __str__(self):
        return f"{type(self).__name__}: area={self.area():.2f}, perimeter={self.perimeter():.2f}"

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return math.pi * self.radius ** 2

    def perimeter(self):
        return 2 * math.pi * self.radius

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

class Square(Rectangle):
    def __init__(self, side):
        super().__init__(side, side)

# Polymorphism — same interface, different shapes
shapes = [Circle(5), Rectangle(4, 6), Square(3), Circle(10)]

total_area = 0
for shape in shapes:
    print(shape)                 # calls __str__ which calls area() and perimeter()
    total_area += shape.area()   # each shape calculates its own way

print(f"\nTotal area: {total_area:.2f}")
```

---

## Common Mistakes

| Mistake | Why It's Wrong |
|---------|----------------|
| Type checking instead of duck typing | `if isinstance(x, Dog): x.speak()` — just call `x.speak()` directly |
| Forgetting `__repr__`/`__str__` | Printing objects shows `<object at 0x...>` — useless |
| Defining `__eq__` without `__hash__` | Makes objects unhashable — can't use in sets or as dict keys |
| Not using `NotImplementedError` in base class | Children might forget to implement required methods — silent bugs |

---

## Code Examples

### Easy
```python
# Example 1: Polymorphism with built-in len()
class Team:
    def __init__(self, members):
        self.members = members
    def __len__(self):
        return len(self.members)

t = Team(["Alice", "Bob", "Charlie"])
print(len(t))  # 3 — works just like len([1,2,3])!


# Example 2: Same method, different classes
class Email:
    def send(self, message):
        return f"Email sent: {message}"

class SMS:
    def send(self, message):
        return f"SMS sent: {message}"

class PushNotification:
    def send(self, message):
        return f"Push notification: {message}"

# Works with any of them!
def notify(channel, message):
    return channel.send(message)

print(notify(Email(), "Hello!"))
print(notify(SMS(), "Hello!"))
print(notify(PushNotification(), "Hello!"))
```

### Medium
```python
# Example 3: Custom container with polymorphism
class Playlist:
    def __init__(self, name):
        self.name = name
        self._songs = []

    def add(self, song):
        self._songs.append(song)

    def __len__(self):
        return len(self._songs)

    def __getitem__(self, index):
        return self._songs[index]

    def __contains__(self, song):
        return song in self._songs

    def __iter__(self):
        return iter(self._songs)

    def __repr__(self):
        return f"Playlist('{self.name}', {len(self)} songs)"

pl = Playlist("Road Trip")
pl.add("Bohemian Rhapsody")
pl.add("Hotel California")
pl.add("Stairway to Heaven")

print(len(pl))                          # 3
print(pl[0])                            # Bohemian Rhapsody
print("Hotel California" in pl)         # True
print(pl)                               # Playlist('Road Trip', 3 songs)
for song in pl:
    print(f"  🎵 {song}")
```

### Advanced
```python
# Example 4: Vector class with full operator overloading
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __abs__(self):
        return (self.x**2 + self.y**2) ** 0.5

    def __bool__(self):
        return self.x != 0 or self.y != 0

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(3, 4)
v2 = Vector(1, 2)

print(v1 + v2)      # Vector(4, 6)
print(v1 - v2)      # Vector(2, 2)
print(v1 * 3)       # Vector(9, 12)
print(abs(v1))       # 5.0 (magnitude)
print(bool(Vector(0, 0)))  # False (zero vector)
print(bool(v1))      # True (non-zero)
```

---

## Interview Questions (Top 5)

**Q1: What is polymorphism?**
> Polymorphism means "many forms." The same method name behaves differently depending on the object that calls it. In Python, this is achieved through method overriding and duck typing — you don't need a common parent class.

**Q2: What is duck typing?**
> Python doesn't check what TYPE an object is — it checks whether the object has the METHODS it needs. "If it walks like a duck and quacks like a duck, it's a duck." This is why `len()` works on lists, strings, dicts — they all have `__len__`.

**Q3: What is operator overloading?**
> Defining magic methods like `__add__`, `__eq__`, `__lt__` so that operators like `+`, `==`, `<` work with your custom classes. When you write `a + b`, Python calls `a.__add__(b)`.

**Q4: Does Python support method overloading?**
> Not in the traditional sense (same name, different number of parameters like Java). Instead, Python uses default arguments, `*args`, `**kwargs`, or `functools.singledispatch` to handle different input types.

**Q5: What is `__call__` and when would you use it?**
> `__call__` makes an object callable like a function: `obj()`. Use it for function-like objects that need to maintain state between calls, or in ML frameworks where layers are objects that you "call" with data.

---

*← [Previous: Chapter 2 (Inheritance)](02_Inheritance.md) | [Next: Chapter 4 (Abstraction & Encapsulation) →](04_Abstraction_Encapsulation.md)*
