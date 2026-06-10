# Chapter 2: Inheritance — "My Child Gets Everything I Have"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Inheritance lets a new class (child/subclass) **automatically get all the attributes and methods** of an existing class (parent/superclass). |
| **Why It Exists** | Code reuse. Instead of copy-pasting the same code into multiple classes, you write it once in a parent and let children inherit it. |
| **Where It Is Used** | Frameworks (Django models, Flask views), game entities (Player → Warrior/Mage), ML (BaseModel → specific models). |
| **Real-World Analogy** | A child inherits traits from their parents. A **Smartphone** inherits all the features of a **Phone** (calling, contacts) and adds new ones (apps, camera). |

---

## Think of It This Way (Verbal Explanation)

You're building a game. You have:
- `Warrior` — has name, health, can walk, can attack with sword
- `Mage` — has name, health, can walk, can cast spells
- `Archer` — has name, health, can walk, can shoot arrows

Notice that **name, health, and walking** are the same for all three. Without inheritance, you'd copy-paste that code three times. With inheritance:

1. Create a `Character` class with name, health, and walk()
2. `Warrior`, `Mage`, and `Archer` all **inherit** from `Character`
3. Each adds their own unique ability

---

## Step 1: Basic Inheritance

```python
class Animal:
    def __init__(self, name, sound):
        self.name = name
        self.sound = sound

    def speak(self):
        return f"{self.name} says {self.sound}!"

    def eat(self):
        return f"{self.name} is eating."

# Dog INHERITS from Animal — gets everything Animal has
class Dog(Animal):
    def fetch(self):
        return f"{self.name} fetches the ball!"

# Cat INHERITS from Animal
class Cat(Animal):
    def purr(self):
        return f"{self.name} is purring..."

# Creating objects
buddy = Dog("Buddy", "Woof")
kitty = Cat("Kitty", "Meow")

# Methods inherited from Animal — we didn't write these in Dog or Cat!
print(buddy.speak())     # Buddy says Woof!
print(buddy.eat())       # Buddy is eating.
print(kitty.speak())     # Kitty says Meow!

# Methods specific to each child
print(buddy.fetch())     # Buddy fetches the ball!
print(kitty.purr())      # Kitty is purring...

# buddy.purr()  ← Error! Dogs can't purr. Only Cat has that method.
```

**What happened?** Dog didn't define `speak()` or `eat()`, but it can use them because it inherited them from Animal. Dog only needed to add its own special method `fetch()`.

---

## Step 2: `super()` — Calling the Parent's `__init__`

When the child has its own `__init__`, you need to call the parent's `__init__` too. Otherwise, the parent's attributes won't be set up.

```python
class Vehicle:
    def __init__(self, brand, fuel_type):
        self.brand = brand
        self.fuel_type = fuel_type

    def start(self):
        return f"{self.brand} engine started."

class Car(Vehicle):
    def __init__(self, brand, fuel_type, num_doors):
        super().__init__(brand, fuel_type)    # Call parent's __init__ FIRST
        self.num_doors = num_doors            # Then add Car-specific stuff

    def honk(self):
        return "Beep beep!"

class Truck(Vehicle):
    def __init__(self, brand, fuel_type, payload_tons):
        super().__init__(brand, fuel_type)
        self.payload_tons = payload_tons

my_car = Car("Toyota", "Petrol", 4)
my_truck = Truck("Ford", "Diesel", 5)

print(my_car.brand)         # Toyota (set by Vehicle's __init__)
print(my_car.num_doors)     # 4 (set by Car's __init__)
print(my_car.start())       # Toyota engine started. (inherited)
print(my_car.honk())        # Beep beep! (Car's own method)

print(my_truck.brand)       # Ford
print(my_truck.payload_tons) # 5
```

### What if you forget `super().__init__()`?

```python
class BadCar(Vehicle):
    def __init__(self, brand, fuel_type, num_doors):
        # Forgot super().__init__()!
        self.num_doors = num_doors

bad = BadCar("Honda", "Petrol", 4)
print(bad.num_doors)    # 4 — this works
# print(bad.brand)      # ❌ AttributeError! brand was never set!
# print(bad.start())    # ❌ AttributeError! self.brand doesn't exist!
```

**Rule:** Always call `super().__init__()` if the parent has one.

---

## Step 3: Method Overriding — "I'll Do It Differently"

A child can **replace** a parent's method with its own version:

```python
class Shape:
    def area(self):
        return 0   # default — doesn't make sense for a generic shape

    def describe(self):
        return f"I'm a shape with area {self.area()}"

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):       # OVERRIDES Shape's area()
        import math
        return math.pi * self.radius ** 2

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):       # OVERRIDES Shape's area()
        return self.width * self.height

class Triangle(Shape):
    def __init__(self, base, height):
        self.base = base
        self.height = height

    def area(self):
        return 0.5 * self.base * self.height

# Each shape calculates area its own way
shapes = [Circle(5), Rectangle(4, 6), Triangle(3, 8)]
for shape in shapes:
    print(shape.describe())
# I'm a shape with area 78.53981633974483
# I'm a shape with area 24
# I'm a shape with area 12.0
```

**Notice:** `describe()` calls `self.area()`. Even though `describe()` is defined in `Shape`, when a `Circle` calls it, `self.area()` calls `Circle`'s version. Python always checks the actual object's type first.

---

## Step 4: Types of Inheritance

### Single Inheritance (most common)
```python
class Parent:
    pass

class Child(Parent):
    pass
```

### Multi-Level Inheritance (chain)
```python
class Grandparent:
    def speak(self):
        return "I'm the grandparent"

class Parent(Grandparent):
    pass

class Child(Parent):
    pass

c = Child()
print(c.speak())  # "I'm the grandparent" — inherited through the chain
```

### Multiple Inheritance (two parents — Python allows this!)
```python
class Flyable:
    def fly(self):
        return f"{self.name} is flying!"

class Swimmable:
    def swim(self):
        return f"{self.name} is swimming!"

class Duck:
    def __init__(self, name):
        self.name = name

class SuperDuck(Duck, Flyable, Swimmable):
    def quack(self):
        return f"{self.name} says Quack!"

donald = SuperDuck("Donald")
print(donald.fly())    # Donald is flying!
print(donald.swim())   # Donald is swimming!
print(donald.quack())  # Donald says Quack!
```

---

## Step 5: The MRO — Method Resolution Order

When a class has multiple parents, Python needs to decide **which method to call first**. This order is called the MRO.

```python
class A:
    def greet(self):
        return "Hello from A"

class B(A):
    def greet(self):
        return "Hello from B"

class C(A):
    def greet(self):
        return "Hello from C"

class D(B, C):
    pass    # D inherits from both B and C

d = D()
print(d.greet())   # "Hello from B" — why B and not C?

# Check the MRO:
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

Python searches: D → B → C → A → object. First match wins. Since B has `greet()`, it stops there.

**Simple rule:** Left to right in the parent list. `D(B, C)` → B is checked before C.

---

## Step 6: The Diamond Problem

```
       A
      / \
     B   C
      \ /
       D
```

Both B and C inherit from A. D inherits from both. If A has `__init__`, does it run twice?

```python
class A:
    def __init__(self):
        print("A init")
        super().__init__()

class B(A):
    def __init__(self):
        print("B init")
        super().__init__()

class C(A):
    def __init__(self):
        print("C init")
        super().__init__()

class D(B, C):
    def __init__(self):
        print("D init")
        super().__init__()

d = D()
# D init
# B init
# C init
# A init    ← called ONLY ONCE! Thanks to MRO + super()
```

**Without `super()` and MRO**, A's `__init__` would run twice. Python's MRO prevents that.

---

## Step 7: `isinstance()` and `issubclass()`

```python
class Animal: pass
class Dog(Animal): pass
class Cat(Animal): pass

buddy = Dog()

# isinstance — checks if an object is an instance of a class
print(isinstance(buddy, Dog))     # True  (buddy IS a Dog)
print(isinstance(buddy, Animal))  # True  (buddy IS ALSO an Animal)
print(isinstance(buddy, Cat))     # False (buddy is NOT a Cat)

# issubclass — checks if a CLASS is a subclass of another CLASS
print(issubclass(Dog, Animal))    # True
print(issubclass(Animal, Dog))    # False (Animal is NOT a subclass of Dog)
print(issubclass(Dog, Dog))       # True  (a class is a subclass of itself)
```

---

## Step 8: Inheritance vs Composition

This is a super important design decision:

**Inheritance = "IS-A" relationship**
```python
class Dog(Animal):     # A Dog IS an Animal ✅
    pass
```

**Composition = "HAS-A" relationship**
```python
class Car:
    def __init__(self):
        self.engine = Engine()     # A Car HAS an Engine ✅
        self.wheels = [Wheel() for _ in range(4)]
```

```python
# Example:
class Engine:
    def start(self):
        return "Engine started"

    def stop(self):
        return "Engine stopped"

class Car:
    def __init__(self, brand):
        self.brand = brand
        self.engine = Engine()   # Car HAS-A Engine (composition)

    def start(self):
        return f"{self.brand}: {self.engine.start()}"

my_car = Car("Toyota")
print(my_car.start())  # Toyota: Engine started
```

**Rule of thumb:** Prefer composition. Use inheritance only for clear IS-A relationships. "Car IS-A Vehicle" = inheritance. "Car HAS-A Engine" = composition.

---

## Common Mistakes

| Mistake | What Happens | Fix |
|---------|-------------|-----|
| Forgetting `super().__init__()` | Parent's attributes don't exist | Always call `super().__init__()` |
| Wrong MRO assumption | Unexpected method called | Check `Class.__mro__` |
| Too deep inheritance (5+ levels) | Code becomes impossible to follow | Keep it to 2-3 levels max |
| Using inheritance for HAS-A | Tight coupling, inflexible code | Use composition instead |

---

## Code Examples

### Easy
```python
# Example 1: Simple parent-child
class Phone:
    def call(self, number):
        return f"Calling {number}..."

class Smartphone(Phone):
    def take_photo(self):
        return "📸 Photo taken!"

s = Smartphone()
print(s.call("123-456"))   # Calling 123-456... (inherited)
print(s.take_photo())       # 📸 Photo taken! (own method)


# Example 2: Child with super()
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Student(Person):
    def __init__(self, name, age, grade):
        super().__init__(name, age)
        self.grade = grade

s = Student("Alice", 20, "A")
print(f"{s.name}, age {s.age}, grade {s.grade}")
```

### Medium
```python
# Example 3: Method overriding with super()
class Logger:
    def log(self, message):
        print(f"[LOG] {message}")

class TimestampLogger(Logger):
    def log(self, message):
        from datetime import datetime
        timestamp = datetime.now().strftime("%H:%M:%S")
        super().log(f"[{timestamp}] {message}")  # Call parent's log, then add timestamp

logger = TimestampLogger()
logger.log("Server started")  # [LOG] [14:30:45] Server started


# Example 4: Multiple inheritance in practice
class JSONMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__, indent=2)

class CSVMixin:
    def to_csv(self):
        return ",".join(str(v) for v in self.__dict__.values())

class User(JSONMixin, CSVMixin):
    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

u = User("Alice", 25, "alice@example.com")
print(u.to_json())
print(u.to_csv())   # Alice,25,alice@example.com
```

### Advanced
```python
# Example 5: Full game character hierarchy
class Character:
    def __init__(self, name, health=100):
        self.name = name
        self.health = health
        self.alive = True

    def take_damage(self, amount):
        self.health -= amount
        if self.health <= 0:
            self.health = 0
            self.alive = False
            return f"{self.name} has been defeated!"
        return f"{self.name} takes {amount} damage. HP: {self.health}"

    def __repr__(self):
        status = "alive" if self.alive else "dead"
        return f"{type(self).__name__}('{self.name}', HP={self.health}, {status})"

class Warrior(Character):
    def __init__(self, name):
        super().__init__(name, health=150)
        self.weapon = "Sword"

    def slash(self, target):
        damage = 25
        return target.take_damage(damage)

class Mage(Character):
    def __init__(self, name):
        super().__init__(name, health=80)
        self.mana = 100

    def fireball(self, target):
        if self.mana < 20:
            return "Not enough mana!"
        self.mana -= 20
        return target.take_damage(40)

w = Warrior("Conan")
m = Mage("Gandalf")
print(w)                    # Warrior('Conan', HP=150, alive)
print(m)                    # Mage('Gandalf', HP=80, alive)
print(w.slash(m))           # Gandalf takes 25 damage. HP: 55
print(m.fireball(w))        # Conan takes 40 damage. HP: 110
```

---

## Interview Questions (Top 5)

**Q1: What is inheritance in Python?**
> Inheritance lets a child class reuse all the code from a parent class — methods, attributes, everything. The child can add new features or override existing ones. It promotes code reuse.

**Q2: What is `super()` and when do you need it?**
> `super()` gives you access to the parent class. You use it most often in `__init__` to call the parent's initializer so parent attributes get set up. Also important in multiple inheritance to follow the MRO correctly.

**Q3: What is the MRO?**
> Method Resolution Order — the order Python searches for methods in an inheritance chain. Uses C3 linearization. Check with `ClassName.__mro__`. For `D(B, C)`, the order is D → B → C → common parent → object.

**Q4: What is the difference between method overriding and method overloading?**
> **Overriding**: child replaces parent's method (same name, different implementation). Python supports this. **Overloading**: same name, different number of parameters. Python doesn't support traditional overloading — use default arguments or `*args` instead.

**Q5: When should you use inheritance vs composition?**
> Inheritance for IS-A relationships: "Dog IS-A Animal." Composition for HAS-A relationships: "Car HAS-A Engine." When in doubt, use composition — it's more flexible and less tightly coupled.

---

*← [Previous: Chapter 1 (OOP Basics)](01_OOP_Basics.md) | [Next: Chapter 3 (Polymorphism) →](03_Polymorphism.md)*
