# Chapter 1: OOP — Classes & Objects

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | OOP (Object-Oriented Programming) is a way of writing code where you group **data** (variables) and **behavior** (functions) together into a single unit called a **class**. |
| **Why It Exists** | Without OOP, large programs become a tangled mess of functions and variables. OOP organizes code into clean, reusable, self-contained units. |
| **Where It Is Used** | Literally everywhere — web apps (Django, Flask), ML (PyTorch, TensorFlow), game development, APIs, databases — any non-trivial Python program. |
| **Real-World Analogy** | A **class** is like a blueprint for a house. An **object** is an actual house built from that blueprint. One blueprint → many houses. |

---

## Think of It This Way (Verbal Explanation)

Imagine you're running a school. You have 500 students. Each student has a name, age, grade, and attendance record. You also need to mark attendance, calculate grades, etc.

**Without OOP**, you'd have:
- A list of names: `["Alice", "Bob", "Charlie", ...]`
- A list of ages: `[15, 16, 15, ...]`
- A list of grades: `[85, 92, 78, ...]`
- A function `mark_attendance(student_index)` that takes an index...

What a nightmare! You're keeping track of everything with matching indexes. One mistake and Alice's grade goes to Bob.

**With OOP**, each student is ONE thing:
```python
alice = Student("Alice", 15, 85)
alice.mark_attendance()
print(alice.grade)
```

Everything about Alice — her data AND her behavior — lives together in one place. THAT is the core idea of OOP.

---

## Step 1: Your First Class (The Absolute Basics)

```python
# A class is created with the 'class' keyword
class Dog:
    pass    # 'pass' means "nothing inside for now"

# Creating an object (also called "instance") from the class
buddy = Dog()
max = Dog()

# These are TWO different objects made from the SAME class
print(type(buddy))    # <class '__main__.Dog'>
print(type(max))      # <class '__main__.Dog'>
print(buddy == max)   # False — they're different objects
```

**In plain English:** `class Dog:` says "I'm defining what a Dog is." `buddy = Dog()` says "Make me an actual Dog and call it buddy."

---

## Step 2: `__init__` — Setting Up Your Object

An empty class is useless. You need to give your objects some **data** when they're created. That's what `__init__` does.

```python
class Dog:
    def __init__(self, name, breed, age):
        self.name = name       # Store name ON this specific dog
        self.breed = breed     # Store breed ON this specific dog
        self.age = age         # Store age ON this specific dog

# Now when we create a Dog, we MUST give it name, breed, and age
buddy = Dog("Buddy", "Golden Retriever", 3)
max = Dog("Max", "German Shepherd", 5)

print(buddy.name)     # Buddy
print(max.name)       # Max
print(buddy.age)      # 3
print(max.breed)      # German Shepherd
```

### What is `self`?

This confuses everyone at first. Here's the simple version:

**`self` = "the object we're working with right now"**

When you write `buddy = Dog("Buddy", "Golden Retriever", 3)`, Python does this internally:

```
Step 1: Create an empty Dog object → let's call it "buddy"
Step 2: Call __init__(buddy, "Buddy", "Golden Retriever", 3)
                     ↑ self
```

So `self` IS `buddy`. When you write `self.name = name`, you're saying `buddy.name = "Buddy"`.

```python
# These two lines do the EXACT same thing:
buddy.name                    # Access buddy's name from outside
# Inside __init__, self.name  # Access buddy's name from inside the class
```

**You never pass `self` yourself.** Python does it automatically. You just define it.

---

## Step 3: Methods — Functions That Belong to an Object

```python
class Dog:
    def __init__(self, name, breed, age):
        self.name = name
        self.breed = breed
        self.age = age

    def bark(self):
        return f"{self.name} says: Woof! Woof!"

    def describe(self):
        return f"{self.name} is a {self.age}-year-old {self.breed}"

    def birthday(self):
        self.age += 1
        return f"Happy birthday {self.name}! Now {self.age} years old."

    def is_puppy(self):
        return self.age < 2

# Using the methods
buddy = Dog("Buddy", "Golden Retriever", 1)

print(buddy.bark())       # Buddy says: Woof! Woof!
print(buddy.describe())   # Buddy is a 1-year-old Golden Retriever
print(buddy.is_puppy())   # True
print(buddy.birthday())   # Happy birthday Buddy! Now 2 years old.
print(buddy.is_puppy())   # False (age changed to 2)
```

**Key rule:** Every method inside a class takes `self` as its first parameter. When you CALL the method (`buddy.bark()`), you don't pass `self` — Python passes `buddy` as `self` automatically.

---

## Step 4: Instance Attributes vs Class Attributes

There are two types of attributes:

```python
class Car:
    # CLASS attribute — shared by ALL cars
    wheels = 4
    total_cars = 0

    def __init__(self, brand, color):
        # INSTANCE attributes — unique to EACH car
        self.brand = brand
        self.color = color
        Car.total_cars += 1   # Update the class attribute

toyota = Car("Toyota", "Red")
honda = Car("Honda", "Blue")

# Instance attributes — different for each object
print(toyota.brand)      # Toyota
print(honda.brand)       # Honda
print(toyota.color)      # Red
print(honda.color)       # Blue

# Class attribute — same for all objects
print(toyota.wheels)     # 4
print(honda.wheels)      # 4
print(Car.wheels)        # 4
print(Car.total_cars)    # 2
```

### The Tricky Part: What Happens When You Assign?

```python
# Reading a class attribute through an instance — works fine
print(toyota.wheels)     # 4 (finds it on the class)

# But if you ASSIGN through an instance, you create an INSTANCE attribute
toyota.wheels = 6        # Creates a NEW attribute on toyota only!
print(toyota.wheels)     # 6 (now has its OWN wheels attribute)
print(honda.wheels)      # 4 (still using the class attribute)
print(Car.wheels)        # 4 (class attribute unchanged)
```

**Think of it like this:** Reading goes up (instance → class). Writing stays local (creates on instance).

---

## Step 5: A Complete Real-World Example

```python
class BankAccount:
    # Class attribute
    bank_name = "Python National Bank"
    total_accounts = 0

    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
        self.transactions = []
        BankAccount.total_accounts += 1

    def deposit(self, amount):
        if amount <= 0:
            return "Deposit must be positive!"
        self.balance += amount
        self.transactions.append(f"+${amount}")
        return f"Deposited ${amount}. New balance: ${self.balance}"

    def withdraw(self, amount):
        if amount <= 0:
            return "Withdrawal must be positive!"
        if amount > self.balance:
            return f"Insufficient funds! Balance: ${self.balance}"
        self.balance -= amount
        self.transactions.append(f"-${amount}")
        return f"Withdrew ${amount}. New balance: ${self.balance}"

    def get_statement(self):
        print(f"\n--- {self.bank_name} ---")
        print(f"Account Owner: {self.owner}")
        print(f"Balance: ${self.balance}")
        print(f"Transactions: {', '.join(self.transactions) or 'None'}")
        print("-------------------")

# Use it!
alice = BankAccount("Alice", 1000)
bob = BankAccount("Bob", 500)

print(alice.deposit(500))       # Deposited $500. New balance: $1500
print(alice.withdraw(200))      # Withdrew $200. New balance: $1300
print(bob.deposit(100))         # Deposited $100. New balance: $600
print(alice.withdraw(5000))     # Insufficient funds! Balance: $1300

alice.get_statement()
print(f"Total accounts: {BankAccount.total_accounts}")  # 2
```

---

## Step 6: `__str__` and `__repr__` — Making Objects Printable

By default, printing an object gives you garbage:

```python
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade

s = Student("Alice", "A")
print(s)  # <__main__.Student object at 0x7f...>  ← useless!
```

Fix it:

```python
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade

    def __str__(self):
        """Called by print() and str(). For humans."""
        return f"{self.name} (Grade: {self.grade})"

    def __repr__(self):
        """Called in REPL and when inside lists. For developers."""
        return f"Student('{self.name}', '{self.grade}')"

s = Student("Alice", "A")
print(s)         # Alice (Grade: A)          ← __str__
print(repr(s))   # Student('Alice', 'A')     ← __repr__
print([s])       # [Student('Alice', 'A')]   ← lists use __repr__
```

**Simple rule:**
- `__str__` = how you'd describe it to a normal person
- `__repr__` = how you'd describe it to a programmer (ideally could recreate the object)
- If you only write one, write `__repr__`. Python falls back to it if `__str__` is missing.

---

## Step 7: Class Methods and Static Methods

### Regular Method (most common)

```python
class Pizza:
    def __init__(self, size, toppings):
        self.size = size
        self.toppings = toppings

    def describe(self):     # Regular method — needs self (the object)
        return f"{self.size}-inch with {', '.join(self.toppings)}"
```

### Class Method (`@classmethod`)

Use when you need an **alternative way to create objects** (a "factory"):

```python
class Pizza:
    def __init__(self, size, toppings):
        self.size = size
        self.toppings = toppings

    @classmethod
    def margherita(cls, size):
        """Alternative constructor — creates a specific type of pizza."""
        return cls(size, ["mozzarella", "tomato", "basil"])

    @classmethod
    def pepperoni(cls, size):
        return cls(size, ["mozzarella", "pepperoni"])

# Instead of remembering toppings:
p1 = Pizza(12, ["mozzarella", "tomato", "basil"])   # manual
p2 = Pizza.margherita(12)                            # factory ← same result, cleaner!
p3 = Pizza.pepperoni(10)
```

**`cls` = the class itself** (like `self` = the object). You can call `cls(...)` to create an instance.

### Static Method (`@staticmethod`)

Use for utility functions that logically belong to the class but don't need the object or class:

```python
class Pizza:
    def __init__(self, size, toppings):
        self.size = size
        self.toppings = toppings

    @staticmethod
    def is_valid_size(size):
        """Doesn't need self or cls — just a helper function."""
        return size in [8, 10, 12, 14, 16]

print(Pizza.is_valid_size(12))   # True
print(Pizza.is_valid_size(15))   # False
```

### When to Use What?

| Type | First Param | When to Use | Example |
|------|-------------|-------------|---------|
| Regular method | `self` | Need to access/modify object data | `account.deposit(100)` |
| `@classmethod` | `cls` | Alternative constructors, class-level operations | `Pizza.margherita(12)` |
| `@staticmethod` | Nothing | Utility functions grouped in the class | `Pizza.is_valid_size(12)` |

---

## Step 8: The `@property` Decorator — Controlled Access

Sometimes you want to **validate** data when it's set, or **compute** values on the fly:

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius    # This triggers the setter below!

    @property
    def radius(self):
        """This runs when you READ circle.radius"""
        return self._radius

    @radius.setter
    def radius(self, value):
        """This runs when you WRITE circle.radius = something"""
        if value < 0:
            raise ValueError("Radius can't be negative!")
        self._radius = value

    @property
    def area(self):
        """This is a computed property — read-only, no setter."""
        import math
        return math.pi * self._radius ** 2

    @property
    def diameter(self):
        return self._radius * 2

c = Circle(5)
print(c.radius)     # 5         ← calls the getter
print(c.area)       # 78.54     ← computed on the fly
print(c.diameter)   # 10

c.radius = 10       # ← calls the setter (validates!)
print(c.area)       # 314.16

# c.radius = -1     # ValueError: Radius can't be negative!
# c.area = 100      # AttributeError: can't set (no setter for area)
```

**Why not just use `self.radius` directly?**

You CAN start simple! The beauty of `@property` is that you can add validation later WITHOUT changing how people use your class. `c.radius` still looks like a normal attribute access.

---

## Common Mistakes

| Mistake | What Happens | Fix |
|---------|-------------|-----|
| Forgetting `self` in method | `def bark():` → TypeError when called | `def bark(self):` |
| Forgetting `self.` when accessing data | `print(name)` → NameError | `print(self.name)` |
| Mutable class attribute | `class C: items = []` → ALL instances share the same list | Put `self.items = []` in `__init__` |
| Calling method without `()` | `buddy.bark` → returns method object, doesn't call it | `buddy.bark()` |
| `__init__` returning a value | `return something` in __init__ → TypeError | `__init__` should only set up attributes |

### Mutable Class Attribute Trap (Important!)

```python
# ❌ WRONG — all students share the SAME list
class Student:
    grades = []     # class attribute — shared!

    def add_grade(self, grade):
        self.grades.append(grade)

alice = Student()
bob = Student()
alice.add_grade(90)
print(bob.grades)   # [90] — Bob sees Alice's grade!

# ✅ CORRECT — each student gets their OWN list
class Student:
    def __init__(self):
        self.grades = []   # instance attribute — unique!

    def add_grade(self, grade):
        self.grades.append(grade)

alice = Student()
bob = Student()
alice.add_grade(90)
print(bob.grades)   # [] — Bob's list is separate
```

---

## Code Examples

### Easy Examples

```python
# Example 1: Simplest possible class
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hi, I'm {self.name} and I'm {self.age} years old."

p = Person("Alice", 25)
print(p.greet())   # Hi, I'm Alice and I'm 25 years old.


# Example 2: Class with default value
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1

    def get_count(self):
        return self.count

c = Counter()
c.increment()
c.increment()
c.increment()
print(c.get_count())  # 3


# Example 3: Two objects are independent
class Light:
    def __init__(self):
        self.is_on = False

    def toggle(self):
        self.is_on = not self.is_on

bedroom = Light()
kitchen = Light()
bedroom.toggle()       # Turn ON bedroom
print(bedroom.is_on)   # True
print(kitchen.is_on)   # False — kitchen is independent!


# Example 4: Object with a method that takes parameters
class Calculator:
    def add(self, a, b):
        return a + b

    def multiply(self, a, b):
        return a * b

calc = Calculator()
print(calc.add(3, 5))       # 8
print(calc.multiply(4, 7))  # 28


# Example 5: Using isinstance to check type
class Animal:
    pass

class Cat(Animal):
    pass

kitty = Cat()
print(isinstance(kitty, Cat))     # True
print(isinstance(kitty, Animal))  # True (Cat IS an Animal)
print(isinstance(kitty, int))     # False
```

### Medium Examples

```python
# Example 6: Shopping cart
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add_item(self, name, price, quantity=1):
        self.items.append({"name": name, "price": price, "qty": quantity})

    def total(self):
        return sum(item["price"] * item["qty"] for item in self.items)

    def show(self):
        for item in self.items:
            subtotal = item["price"] * item["qty"]
            print(f"  {item['name']}: ${item['price']} x {item['qty']} = ${subtotal}")
        print(f"  TOTAL: ${self.total()}")

cart = ShoppingCart()
cart.add_item("Laptop", 999.99)
cart.add_item("Mouse", 29.99, 2)
cart.add_item("USB Cable", 9.99, 3)
cart.show()


# Example 7: Class method as factory
class User:
    def __init__(self, name, email, role):
        self.name = name
        self.email = email
        self.role = role

    @classmethod
    def admin(cls, name, email):
        return cls(name, email, "admin")

    @classmethod
    def guest(cls):
        return cls("Guest", "guest@example.com", "guest")

    def __repr__(self):
        return f"User({self.name}, {self.role})"

admin = User.admin("Alice", "alice@company.com")
guest = User.guest()
print(admin)   # User(Alice, admin)
print(guest)   # User(Guest, guest)


# Example 8: Comparing objects
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def fahrenheit(self):
        return self.celsius * 9/5 + 32

    def __eq__(self, other):
        return self.celsius == other.celsius

    def __lt__(self, other):
        return self.celsius < other.celsius

    def __repr__(self):
        return f"{self.celsius}°C ({self.fahrenheit}°F)"

t1 = Temperature(100)
t2 = Temperature(0)
t3 = Temperature(100)
print(t1)           # 100°C (212.0°F)
print(t1 == t3)     # True
print(t2 < t1)      # True
print(sorted([t1, t2, t3]))  # [0°C, 100°C, 100°C]


# Example 9: __str__ vs __repr__
class Book:
    def __init__(self, title, author, year):
        self.title = title
        self.author = author
        self.year = year

    def __str__(self):
        return f'"{self.title}" by {self.author}'

    def __repr__(self):
        return f"Book('{self.title}', '{self.author}', {self.year})"

b = Book("1984", "George Orwell", 1949)
print(b)        # "1984" by George Orwell           ← __str__
print(repr(b))  # Book('1984', 'George Orwell', 1949) ← __repr__
print([b])      # [Book('1984', 'George Orwell', 1949)] ← list uses __repr__


# Example 10: Static method for validation
class Email:
    def __init__(self, address):
        if not Email.is_valid(address):
            raise ValueError(f"Invalid email: {address}")
        self.address = address

    @staticmethod
    def is_valid(address):
        return "@" in address and "." in address.split("@")[-1]

    def __repr__(self):
        return f"Email('{self.address}')"

e = Email("alice@gmail.com")   # ✅ works
print(e)
# Email("bad-email")           # ❌ ValueError: Invalid email
```

### Advanced Examples

```python
# Example 11: Property with validation
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary    # triggers setter!

    @property
    def salary(self):
        return self._salary

    @salary.setter
    def salary(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("Salary must be a number")
        if value < 0:
            raise ValueError("Salary can't be negative")
        self._salary = value

    @property
    def monthly_salary(self):
        return round(self._salary / 12, 2)

emp = Employee("Alice", 120000)
print(emp.salary)          # 120000
print(emp.monthly_salary)  # 10000.0
emp.salary = 150000        # setter validates
# emp.salary = -1000       # ValueError!
# emp.salary = "lots"      # TypeError!


# Example 12: Class tracking all its instances
class Player:
    all_players = []

    def __init__(self, name, score=0):
        self.name = name
        self.score = score
        Player.all_players.append(self)

    @classmethod
    def get_leaderboard(cls):
        sorted_players = sorted(cls.all_players, key=lambda p: p.score, reverse=True)
        for i, p in enumerate(sorted_players, 1):
            print(f"  {i}. {p.name}: {p.score}")

    def __repr__(self):
        return f"Player('{self.name}', {self.score})"

Player("Alice", 850)
Player("Bob", 920)
Player("Charlie", 780)
Player.get_leaderboard()


# Example 13: Building a linked list node
class Node:
    def __init__(self, value, next_node=None):
        self.value = value
        self.next = next_node

    def __repr__(self):
        values = []
        current = self
        while current:
            values.append(str(current.value))
            current = current.next
        return " → ".join(values)

# Build: 1 → 2 → 3
head = Node(1, Node(2, Node(3)))
print(head)  # 1 → 2 → 3


# Example 14: Context-aware object with __bool__
class Stack:
    def __init__(self):
        self._items = []

    def push(self, item):
        self._items.append(item)

    def pop(self):
        return self._items.pop()

    def __len__(self):
        return len(self._items)

    def __bool__(self):
        return len(self._items) > 0

    def __repr__(self):
        return f"Stack({self._items})"

s = Stack()
print(bool(s))  # False (empty)
s.push("a")
s.push("b")
print(bool(s))  # True (has items)
print(len(s))   # 2

# Works in if statements now!
if s:
    print("Stack has items!")


# Example 15: Full class with everything combined
class Product:
    _id_counter = 0

    def __init__(self, name, price, stock=0):
        Product._id_counter += 1
        self.id = Product._id_counter
        self.name = name
        self.price = price
        self.stock = stock

    @classmethod
    def from_dict(cls, data):
        return cls(data["name"], data["price"], data.get("stock", 0))

    @staticmethod
    def format_price(amount):
        return f"${amount:,.2f}"

    @property
    def is_available(self):
        return self.stock > 0

    def sell(self, qty=1):
        if qty > self.stock:
            return f"Only {self.stock} left!"
        self.stock -= qty
        return f"Sold {qty} {self.name}(s). {self.stock} remaining."

    def __str__(self):
        status = "✅ In Stock" if self.is_available else "❌ Out of Stock"
        return f"[{self.id}] {self.name} - {Product.format_price(self.price)} ({status})"

    def __repr__(self):
        return f"Product('{self.name}', {self.price}, stock={self.stock})"

p1 = Product("Laptop", 999.99, 5)
p2 = Product.from_dict({"name": "Mouse", "price": 29.99, "stock": 50})
print(p1)              # [1] Laptop - $999.99 (✅ In Stock)
print(p2)              # [2] Mouse - $29.99 (✅ In Stock)
print(p1.sell(2))      # Sold 2 Laptop(s). 3 remaining.
print(p1.sell(10))     # Only 3 left!
```

---

## Edge Cases

1. **Class without `__init__`** — works fine, objects just have no initial attributes.
2. **`self` is not a keyword** — you could name it `this` or `me`, but NEVER do that. Convention is `self`.
3. **Objects can get new attributes anytime:** `buddy.nickname = "Bud"` works even if `nickname` isn't in `__init__`.
4. **`__init__` should NOT return anything** — it returns `None` implicitly.
5. **Class attributes with mutable defaults (lists, dicts)** — ALL instances share the same object! Use `__init__` instead.

---

## Interview Questions (Top 10)

**Q1: What is OOP? Why use it?**
> OOP is a programming paradigm that organizes code into classes (blueprints) and objects (instances). Benefits: code reuse, modularity, encapsulation, easier maintenance, models real-world entities naturally.

**Q2: What is the difference between a class and an object?**
> A class is a blueprint/template — it defines what data and methods something has. An object (instance) is an actual thing created from that blueprint. `Dog` is a class. `buddy = Dog("Buddy")` — `buddy` is an object.

**Q3: What is `self` in Python?**
> `self` refers to the current instance. When you call `buddy.bark()`, Python passes `buddy` as `self` to the `bark` method. It's how methods access the object's own data.

**Q4: What is `__init__`? Is it a constructor?**
> `__init__` is the initializer — it sets up the object's initial state (attributes). Technically, `__new__` is the real constructor (creates the object), and `__init__` runs after to initialize it. In practice, most people call `__init__` the constructor.

**Q5: What is the difference between instance attributes and class attributes?**
> Instance attributes belong to a specific object (defined with `self.x` in `__init__`). Class attributes are shared by ALL objects (defined directly in the class body). Python looks at the instance first, then the class.

**Q6: What is the difference between `__str__` and `__repr__`?**
> `__str__` is human-readable (used by `print()`). `__repr__` is developer-readable (used in REPL, lists, debugging — should ideally be valid Python to recreate the object). If only one is defined, write `__repr__`.

**Q7: What is the difference between `@classmethod`, `@staticmethod`, and a regular method?**
> Regular method takes `self` (the instance), for operations on object data. `@classmethod` takes `cls` (the class), for factory methods and class-level operations. `@staticmethod` takes nothing, for utility functions that belong to the class logically.

**Q8: What is `@property` and why use it?**
> `@property` lets you define a method that's accessed like an attribute. Use it for validation (setter), computed values (getter), and read-only attributes (getter without setter). The interface stays clean: `circle.area` instead of `circle.get_area()`.

**Q9: What happens when you write `obj.attribute`?**
> Python first looks in `obj.__dict__` (instance attributes). If not found, it looks at the class's `__dict__`. If still not found, it looks at parent classes (via MRO). If nowhere, raises `AttributeError`.

**Q10: Can you have a class without `__init__`?**
> Yes. Python provides a default `__init__` that does nothing. The class works fine, but objects won't have any initial attributes unless you add them manually after creation.

---

*[Next: Chapter 2 (Inheritance) →](02_Inheritance.md)*
