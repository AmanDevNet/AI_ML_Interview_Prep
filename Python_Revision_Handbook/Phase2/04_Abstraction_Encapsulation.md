# Chapter 4: Abstraction & Encapsulation

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Abstraction** | Hiding the **complexity** and showing only the **interface**. You use `car.start()` without knowing how the engine works inside. |
| **Encapsulation** | Controlling **who can access what** inside your class. Bundling data + methods together and protecting the data from direct access. |
| **Where Used** | APIs, frameworks, libraries — anywhere you want a clean interface that hides messy implementation details. |
| **Real-World Analogy** | **Abstraction**: A TV remote — you press buttons, you don't know the circuits inside. **Encapsulation**: A capsule pill — medicine is wrapped inside, you can't touch the powder directly. |

---

## Part 1: Abstraction — "Hide the How, Show the What"

### Verbal Explanation

When you use `len([1, 2, 3])`, do you know HOW Python counts the elements? No. You just call `len()` and get `3`. The implementation is **hidden** from you. That's abstraction.

When you write `file.read()`, you don't care whether the file is on an SSD, a network drive, or in the cloud. You just call `.read()`. The complexity is abstracted away.

### Abstract Base Classes (ABCs)

An abstract class is like a **contract**: "If you want to be a Shape, you MUST have an `area()` method."

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        """Every shape MUST implement this. No exceptions."""
        pass

    @abstractmethod
    def perimeter(self):
        """Every shape MUST implement this too."""
        pass

    def describe(self):
        """This is a regular method — children get it for free."""
        return f"I'm a {type(self).__name__} with area {self.area():.2f}"

# Can you create a Shape directly? NO!
# shape = Shape()  # ❌ TypeError: Can't instantiate abstract class Shape

# You MUST create a child that implements ALL abstract methods
class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):           # ✅ implemented!
        import math
        return math.pi * self.radius ** 2

    def perimeter(self):      # ✅ implemented!
        import math
        return 2 * math.pi * self.radius

c = Circle(5)
print(c.area())          # 78.54
print(c.describe())      # I'm a Circle with area 78.54

# What if you forget to implement a method?
class BadShape(Shape):
    def area(self):
        return 0
    # Forgot perimeter()!

# bad = BadShape()  # ❌ TypeError: Can't instantiate abstract class BadShape
#                   #    with abstract method perimeter
```

**Why not just use `raise NotImplementedError`?**

```python
# With NotImplementedError — error happens when you CALL the method
class Shape:
    def area(self):
        raise NotImplementedError

s = Shape()        # ✅ works (bad! shouldn't be allowed)
# s.area()         # ❌ crashes HERE — maybe in production!

# With ABC — error happens when you CREATE the object
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

# Shape()           # ❌ crashes HERE — immediately at creation time. Much safer!
```

ABCs catch the problem **earlier** — at object creation, not at runtime.

---

### Practical Example: Payment System

```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def charge(self, amount):
        pass

    @abstractmethod
    def refund(self, amount):
        pass

    def validate_amount(self, amount):
        """Common logic — all processors share this."""
        if amount <= 0:
            raise ValueError("Amount must be positive")

class StripeProcessor(PaymentProcessor):
    def charge(self, amount):
        self.validate_amount(amount)
        return f"Charged ${amount:.2f} via Stripe"

    def refund(self, amount):
        self.validate_amount(amount)
        return f"Refunded ${amount:.2f} via Stripe"

class PayPalProcessor(PaymentProcessor):
    def charge(self, amount):
        self.validate_amount(amount)
        return f"Charged ${amount:.2f} via PayPal"

    def refund(self, amount):
        self.validate_amount(amount)
        return f"Refunded ${amount:.2f} via PayPal"

# The checkout function works with ANY payment processor
def checkout(processor: PaymentProcessor, amount):
    print(processor.charge(amount))

checkout(StripeProcessor(), 49.99)   # Charged $49.99 via Stripe
checkout(PayPalProcessor(), 29.99)   # Charged $29.99 via PayPal
```

---

## Part 2: Encapsulation — "Control Access to Your Data"

### The Three Access Levels in Python

```python
class Employee:
    def __init__(self, name, salary, ssn):
        self.name = name          # PUBLIC — anyone can access
        self._salary = salary     # PROTECTED — "please don't use from outside"
        self.__ssn = ssn          # PRIVATE — name mangled (harder to access)

emp = Employee("Alice", 90000, "123-45-6789")

# Public — access freely
print(emp.name)           # Alice ✅

# Protected — works, but it's a "don't touch" sign
print(emp._salary)        # 90000 ✅ (but you shouldn't)

# Private — name mangled!
# print(emp.__ssn)         # ❌ AttributeError!
print(emp._Employee__ssn) # 123-45-6789 ✅ (but DON'T do this)
```

### What's REALLY happening?

| Prefix | Convention | What Python Does |
|--------|-----------|------------------|
| `name` | Public | Nothing. Full access for everyone. |
| `_name` | Protected | Nothing. Just a convention: "Hey, this is internal." |
| `__name` | Private | **Name mangling**: renames to `_ClassName__name`. Prevents accidental access in child classes. |
| `__name__` | Dunder | Reserved for Python's special methods. Don't invent your own. |

### Why Name Mangling Exists

It's NOT about security. It prevents **name collisions in inheritance**:

```python
class Parent:
    def __init__(self):
        self.__value = "parent"   # becomes _Parent__value

class Child(Parent):
    def __init__(self):
        super().__init__()
        self.__value = "child"    # becomes _Child__value (DIFFERENT!)

c = Child()
print(c._Parent__value)   # "parent"
print(c._Child__value)    # "child"
# They don't collide! That's the whole point of name mangling.
```

---

### The Pythonic Way: Use `@property` for Encapsulation

Instead of making things private and writing getters/setters like Java, Python uses `@property`:

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner
        self._balance = balance

    @property
    def balance(self):
        """Reading balance — just return it."""
        return self._balance

    @balance.setter
    def balance(self, amount):
        """Setting balance — validate first!"""
        if amount < 0:
            raise ValueError("Balance can't be negative!")
        self._balance = amount

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self._balance += amount

    def withdraw(self, amount):
        if amount > self._balance:
            raise ValueError("Insufficient funds!")
        self._balance -= amount

acc = BankAccount("Alice", 1000)
print(acc.balance)        # 1000 (calls getter)
acc.balance = 2000        # calls setter — validates!
acc.deposit(500)
print(acc.balance)        # 2500
# acc.balance = -100      # ❌ ValueError: Balance can't be negative!
```

**The beauty:** From the outside, `acc.balance` looks like a simple attribute. But behind the scenes, Python runs your getter/setter code with validation. You can add validation later WITHOUT changing how users access the attribute.

---

## Common Mistakes

| Mistake | Why It's Wrong |
|---------|----------------|
| Thinking `_var` is truly private | It's just a convention. Python doesn't enforce it. |
| Thinking `__var` is truly private | It's accessible via `_ClassName__var`. It's name mangling, not security. |
| Making everything private | Python philosophy: "We're all consenting adults." Use `_` for internal stuff, that's enough. |
| Inventing `__name__` dunders | Double underscores on BOTH sides are reserved for Python. Don't create your own. |

---

## Code Examples

### Easy
```python
# Example 1: Simple abstraction — user doesn't know the internals
class EmailSender:
    def send(self, to, subject, body):
        """User just calls this. Don't care about SMTP details."""
        self._connect()
        self._authenticate()
        self._send_message(to, subject, body)
        self._disconnect()

    def _connect(self):
        print("  Connecting to server...")

    def _authenticate(self):
        print("  Authenticating...")

    def _send_message(self, to, subject, body):
        print(f"  Sending '{subject}' to {to}")

    def _disconnect(self):
        print("  Disconnecting...")

sender = EmailSender()
sender.send("bob@email.com", "Hello!", "Hi Bob!")
# User calls ONE method. Four internal steps happen automatically.


# Example 2: Encapsulation with property
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius   # triggers setter

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero!")
        self._celsius = value

    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

t = Temperature(100)
print(t.celsius)      # 100
print(t.fahrenheit)   # 212.0
t.celsius = 0
print(t.fahrenheit)   # 32.0
# t.celsius = -300    # ❌ ValueError!
```

### Medium
```python
# Example 3: Abstract class for a plugin system
from abc import ABC, abstractmethod

class Plugin(ABC):
    @abstractmethod
    def execute(self, data):
        pass

    @property
    @abstractmethod
    def name(self):
        pass

class UpperCasePlugin(Plugin):
    @property
    def name(self):
        return "UPPERCASE"

    def execute(self, data):
        return data.upper()

class ReversePlugin(Plugin):
    @property
    def name(self):
        return "REVERSE"

    def execute(self, data):
        return data[::-1]

# Process data through any set of plugins
def process(data, plugins):
    for plugin in plugins:
        data = plugin.execute(data)
        print(f"  After {plugin.name}: {data}")
    return data

result = process("hello world", [UpperCasePlugin(), ReversePlugin()])
# After UPPERCASE: HELLO WORLD
# After REVERSE: DLROW OLLEH
```

### Advanced
```python
# Example 4: Fully encapsulated user class
class User:
    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self._password_hash = self._hash(password)

    @staticmethod
    def _hash(password):
        """Simple hash (use bcrypt in real code!)"""
        import hashlib
        return hashlib.sha256(password.encode()).hexdigest()

    def check_password(self, password):
        """The ONLY way to verify password. Can't read it directly."""
        return self._password_hash == self._hash(password)

    def change_password(self, old_password, new_password):
        if not self.check_password(old_password):
            raise ValueError("Incorrect old password!")
        if len(new_password) < 8:
            raise ValueError("New password must be 8+ characters!")
        self._password_hash = self._hash(new_password)
        return "Password changed successfully!"

    def __repr__(self):
        return f"User('{self.username}', '{self.email}')"

user = User("alice", "alice@email.com", "MySecurePass123")
print(user.check_password("wrong"))         # False
print(user.check_password("MySecurePass123"))  # True
print(user.change_password("MySecurePass123", "NewPass456!"))
# The password hash is never exposed. You can only CHECK or CHANGE it.
```

---

## Interview Questions (Top 5)

**Q1: What is the difference between abstraction and encapsulation?**
> **Abstraction** hides complexity — shows WHAT something does, not HOW. **Encapsulation** controls access — bundles data and methods together and restricts direct access to internal data. Abstraction is the concept; encapsulation is the mechanism.

**Q2: What is an abstract class? Can you create an instance of it?**
> A class with at least one `@abstractmethod`. No, you can't instantiate it. You must subclass it and implement ALL abstract methods. It enforces a contract that children must follow.

**Q3: What is the difference between `_var` and `__var` in Python?**
> `_var` is a convention meaning "internal, don't use from outside." Python does nothing special. `__var` triggers name mangling — Python renames it to `_ClassName__var` to prevent name collisions in inheritance. Neither is truly private.

**Q4: Why doesn't Python have strict access control like Java?**
> Python follows "we're all consenting adults." Strict access control adds complexity without much benefit in a dynamic language. Conventions (`_private`) and name mangling (`__private`) are sufficient.

**Q5: Why use ABCs instead of just raising `NotImplementedError`?**
> ABCs catch the error at **creation time** (immediately when you try to instantiate an incomplete class). `NotImplementedError` only catches it when you actually **call** the method — which could be in production.

---

*← [Previous: Chapter 3 (Polymorphism)](03_Polymorphism.md) | [Next: Chapter 5 (Decorators) →](05_Decorators.md)*
