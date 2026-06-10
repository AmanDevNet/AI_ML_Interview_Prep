# Chapter 4: Typing, Protocols & Pydantic — "Type Safety & Validation"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **Typing** provides hints for static analysis. **Protocols** implement static duck typing. **Pydantic** enforces data validation and type conversion at runtime. |
| **Why It Exists** | Python's dynamic nature makes it easy to pass wrong objects into methods, causing runtime failures. Type checking catches errors before execution. Pydantic guarantees clean input boundaries. |
| **Where It Is Used** | API endpoints (FastAPI), data ingestion pipelines, production codebases using static type check analysis (Mypy), and system configuration. |
| **Real-World Analogy** | **Typing/Protocols:** Writing "Requires 12V DC Adapter" next to a power plug. You check it before plugging things in. **Pydantic:** An airport security gate that physically inspects luggage dimensions and stops you if it's too large. |

---

## Think of It This Way

### 1. Nominal vs Structural Subtyping
* **Nominal (Standard Inheritance):** If Class `Car` inherits from `Vehicle`, it is a subclass because we explicitly declared it: `class Car(Vehicle)`. This is "nominal" (by name).
* **Structural (Protocols / Duck Typing):** We define a `Protocol` called `Flyer` that has a `fly()` method. Any class that has a `fly()` method is compatible with `Flyer`, even if it has no inheritance relationship whatsoever. It's static duck typing!

### 2. Typing is Static; Pydantic is Runtime
* Standard type hints are ignored by Python during execution. If you write `def greet(name: str):`, Python won't stop you from passing `123`.
* Pydantic enforces types at runtime. If you pass `"123"` to a Pydantic `str` field, it coerces/converts it to a string. If you pass a list where an integer is expected, it throws a validation error.

---

## Step 1: Typing & Generics

You can define generic algorithms that work with multiple types using `TypeVar`:

```python
from typing import TypeVar, List

# Create a Generic type variable bound to numbers
T = TypeVar('T', int, float)

def get_first(lst: List[T]) -> T:
    return lst[0]

print(get_first([1, 2, 3]))      # ✅ Valid type int
print(get_first([1.1, 2.2]))    # ✅ Valid type float
# get_first(["a", "b"])         # ❌ Mypy static type check failure
```

---

## Step 2: Protocols (Structural Subtyping)

Use `Protocol` to define interface behavior contracts:

```python
from typing import Protocol

class Renderable(Protocol):
    def render(self) -> str:
        ...  # Empty method body

# This function accepts any object that satisfies Renderable Protocol
def print_page(item: Renderable):
    print(item.render())

class Button:
    def render(self) -> str:
        return "<button>Click</button>"

class Link:
    def render(self) -> str:
        return "<a>Link</a>"

# Both work even though they do not inherit from a shared base class!
print_page(Button())
print_page(Link())
```

---

## Step 3: Pydantic v2 validation

Pydantic v2 has a validation engine written in Rust, making it up to 10x faster than v1.

```python
from pydantic import BaseModel, Field, field_validator, ValidationError

class User(BaseModel):
    id: int
    name: str = Field(min_length=2)
    email: str
    
    # Custom Field Validator
    @field_validator('email')
    @classmethod
    def validate_email(cls, value: str) -> str:
        if "@" not in value:
            raise ValueError("Invalid email format")
        return value

try:
    # Coerces string "1" to integer 1 automatically!
    u = User(id="1", name="Alice", email="alice@company.com")
    print(u) # id=1 name='Alice' email='alice@company.com'
    
    # Trigger Validation Error
    User(id=2, name="A", email="bademail")
except ValidationError as e:
    print(e.errors()) # Clean JSON-like error logs explaining the failure
```

---

## Code Examples

### Easy Examples
```python
from typing import Union, Optional

# 1. Union and Optional Types
# Optional[x] is shorthand for Union[x, None]
def process_data(val: Union[str, int], prefix: Optional[str] = None):
    pass

# 2. Simple Pydantic model serialization
class Item(BaseModel):
    name: str
    price: float

item = Item(name="Book", price=12.5)
print(item.model_dump())      # Converts to dictionary: {'name': 'Book', 'price': 12.5}
print(item.model_dump_json()) # Converts to JSON string
```

### Medium Examples
```python
# 3. Pydantic Model Validator (Whole-model checks)
from pydantic import model_validator

class DateRange(BaseModel):
    start_date: int
    end_date: int

    @model_validator(mode='after')
    def check_dates(self) -> 'DateRange':
        if self.start_date > self.end_date:
            raise ValueError("Start date must be before end date")
        return self
```

### Advanced Examples
```python
# 4. Building Generic API Response schemas with Pydantic
from typing import Generic

DataT = TypeVar('DataT')

class APIResponse(BaseModel, Generic[DataT]):
    status: str
    data: DataT

class UserProfile(BaseModel):
    username: str

# Schema resolves the type of data field dynamically
response = APIResponse[UserProfile](
    status="success",
    data=UserProfile(username="alice99")
)
print(response.model_dump())

# 5. Runtime enforcement using TypeGuard
from typing import TypeGuard, Dict, Any

def is_string_dict(val: Dict[Any, Any]) -> TypeGuard[Dict[str, Any]]:
    # Returns true if all keys are strings
    return all(isinstance(k, str) for k in val.keys())

# Allows type checkers to infer type safety within code blocks
```

---

## Common Mistakes & Edge Cases

### 1. Pydantic v1 vs v2 Method Differences
If you are working on an updated project, remember that Pydantic v2 renamed methods:
* **V1:** `model.dict()`, `model.json()`, `class.validate()`
* **V2:** `model.model_dump()`, `model.model_dump_json()`, `class.model_validate()`
Using old methods triggers deprecation warnings or AttributeError.

### 2. Forgetting that type hints do NOT enforce constraints at runtime
A common beginner mistake is writing:
```python
def check_age(age: int):
    # expect Python to fail if string passed
    pass
```
Always use Pydantic, manual assertions, or static check tools like **Mypy**; standard Python ignores annotations during runtime completely.

---

## Interview Questions (Top 5)

**Q1: What is the difference between Nominal Subtyping and Structural Subtyping?**
> * **Nominal Subtyping:** Compatibility is decided based on explicit inheritance declarations (e.g. `class Cat(Animal)`). The subclass relationship is based on names.
> * **Structural Subtyping (Protocols):** Compatibility is decided solely based on matching structure or methods (duck typing). If an object implements all methods defined in a `Protocol`, it is compatible, regardless of its class inheritance tree.

**Q2: What is the purpose of `typing.TypeVar` and how do you write a Generic class in Python?**
> `TypeVar` is a placeholder used to represent generic types in type hints. It allows you to write functions or classes that work with multiple types while maintaining type consistency between inputs and outputs. You write a generic class by inheriting from `typing.Generic[T]`, passing the `TypeVar` as a parameter.

**Q3: How does Pydantic's data parsing differ from strict type enforcement?**
> Pydantic is a data parser, not a strict validator. Instead of immediately failing if a type doesn't match, Pydantic attempts to safely coerce the input data into the expected type (e.g., if you pass string `"42"` to an `int` field, it parses it into the integer `42`). It only throws a validation error if coercion fails (e.g., parsing `"abc"` into an `int`).

**Q4: How do you implement custom validation rules for individual fields versus multiple fields in Pydantic v2?**
> * **Individual Field:** Use the `@field_validator('field_name')` decorator to validate or alter a specific field's value.
> * **Multiple Fields / Model Level:** Use the `@model_validator(mode='after')` decorator. This runs validation after all individual fields have been parsed, allowing you to check relationship constraints between different attributes (like ensuring `start_date < end_date`).

**Q5: What was the main architectural upgrade in Pydantic v2 compared to v1?**
> Pydantic v2 separated the schema definition from the validation engine. The core validation engine was completely rewritten in Rust (`pydantic-core`), moving validation loops out of Python. This makes data validation and serialization up to 10 times faster, especially when parsing nested models or lists of objects.

---

*← [Previous: Chapter 3 (Metaclasses & Descriptors)](03_Metaclasses_Descriptors.md) | [Next: Chapter 5 (FastAPI & Serialization Internals) →](05_FastAPI_Serialization.md)*
