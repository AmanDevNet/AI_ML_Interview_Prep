# Chapter 19: Exception Handling — What To Do When Things Go Wrong

---

## The One-Line Summary

> **Exception Handling = Your code WILL crash. This is how you prepare for it.**

Without it, one bad input kills your entire program.
With it, you catch the problem, handle it, and keep running.

---

## Step 1: What Is an Exception?

An exception is Python YELLING at you that something went wrong.

```python
print(10 / 0)
```
```
ZeroDivisionError: division by zero    ← Python crashed. Program dead.
```

You've already seen these:
```python
int("hello")           # ValueError
[1, 2, 3][10]          # IndexError
my_dict["missing_key"] # KeyError
open("nope.txt")       # FileNotFoundError
```

**Every one of these crashes your program.** Exception handling stops that crash.

---

## Step 2: try / except — Catch the Crash

```python
try:
    result = 10 / 0          # this line will crash
except ZeroDivisionError:
    print("Can't divide by zero!")  # instead of crashing, run this

print("Program still alive!")  # ✅ this line runs now!
```

**How to read this:**
- `try` = "attempt this risky code"
- `except` = "if it explodes, do this instead"

That's it. That's the entire concept.

---

## Step 3: Get the Error Message with `as`

```python
try:
    num = int("hello")
except ValueError as e:
    print(f"What went wrong: {e}")
# Output: What went wrong: invalid literal for int() with base 10: 'hello'
```

`as e` saves the error message into variable `e` so you can print it, log it, etc.

---

## Step 4: Catching Different Errors Separately

What if your code can crash in multiple ways?

```python
def get_item(data, index_str):
    try:
        index = int(index_str)       # could crash: ValueError
        return data[index]           # could crash: IndexError
    except ValueError:
        print("That's not a number!")
    except IndexError:
        print("That index doesn't exist!")

get_item([10, 20, 30], "abc")   # "That's not a number!"
get_item([10, 20, 30], "99")    # "That index doesn't exist!"
get_item([10, 20, 30], "1")     # returns 20 (no crash)
```

**Think of it like multiple safety nets at different heights — whichever error falls first, the right net catches it.**

---

## Step 5: else — "Run This Only If Nothing Crashed"

```python
try:
    num = int("42")
except ValueError:
    print("Bad input!")
else:
    print(f"Success! Got {num}")    # ← only runs if try DIDN'T crash
```

**Why not just put it inside `try`?** Because `else` makes it clear: this code should ONLY run on success. It also won't accidentally catch new errors from the success code.

---

## Step 6: finally — "Run This No Matter What"

```python
try:
    f = open("data.txt")
    data = f.read()
except FileNotFoundError:
    print("File missing!")
finally:
    print("Cleanup done!")    # ← runs whether crash happened or not
```

**Use `finally` for cleanup** — closing files, closing database connections, releasing resources. It runs even if you `return` early.

```python
def test():
    try:
        return "hello"
    finally:
        print("finally runs!")    # ← still runs, even with return!

test()   # prints "finally runs!" then returns "hello"
```

---

## Step 7: The Full Pattern

```python
try:
    # risky code
    result = some_function()
except SpecificError:
    # handle that specific error
    handle_error()
except AnotherError:
    # handle a different error
    handle_other_error()
else:
    # runs only if NO error happened
    save(result)
finally:
    # ALWAYS runs — cleanup
    close_connections()
```

**Memory trick**: T-E-E-F → "**T**ry **E**xcept **E**lse **F**inally"

---

## Step 8: raise — Throw Your Own Errors

Sometimes YOU want to crash on purpose. Like when someone gives invalid input:

```python
def set_age(age):
    if age < 0:
        raise ValueError("Age can't be negative!")
    if age > 150:
        raise ValueError("That's not a real age!")
    return age

set_age(-5)   # ValueError: Age can't be negative!
```

**`raise` = "I'm deliberately crashing here because something is wrong."**

### Re-raise: Catch, Log, Then Crash Anyway

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Logging error: {e}")
    raise       # ← crash anyway after logging
```

Useful when you want to KNOW about the error but not hide it.

---

## Step 9: Custom Exceptions — Make Your Own Error Types

```python
class InsufficientFundsError(Exception):
    """Bank account doesn't have enough money."""
    pass

class BankAccount:
    def __init__(self, balance):
        self.balance = balance

    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError(
                f"Need ₹{amount} but only have ₹{self.balance}"
            )
        self.balance -= amount
        return self.balance

# Usage
account = BankAccount(1000)
try:
    account.withdraw(5000)
except InsufficientFundsError as e:
    print(f"Error: {e}")
# Error: Need ₹5000 but only have ₹1000
```

**Why custom exceptions?** So your code speaks YOUR language. `InsufficientFundsError` is way clearer than a generic `Exception`.

---

## Common Exception Types — Cheat Sheet

| Exception | Trigger | Example |
|---|---|---|
| `ValueError` | Right type, wrong value | `int("abc")` |
| `TypeError` | Wrong type | `"a" + 1` |
| `IndexError` | List index out of range | `[1,2,3][10]` |
| `KeyError` | Dict key not found | `{}["x"]` |
| `ZeroDivisionError` | Dividing by zero | `1 / 0` |
| `FileNotFoundError` | File doesn't exist | `open("nope.txt")` |
| `AttributeError` | Object has no attribute | `"str".foo()` |
| `NameError` | Variable not defined | `print(xyz)` |
| `ImportError` | Module not found | `import blahblah` |

---

## Common Mistakes

```python
# ❌ MISTAKE 1: Bare except — catches EVERYTHING (even Ctrl+C!)
try:
    do_something()
except:                     # NEVER do this
    pass

# ✅ FIX: Be specific
try:
    do_something()
except Exception as e:      # catches errors, but lets Ctrl+C work
    print(f"Error: {e}")
```

```python
# ❌ MISTAKE 2: Silently swallowing errors
try:
    data = process(input)
except Exception:
    pass                    # error? what error? 🤷 — bugs hide here forever

# ✅ FIX: At minimum, log it
try:
    data = process(input)
except Exception as e:
    print(f"Error: {e}")    # at least you'll KNOW something went wrong
```

```python
# ❌ MISTAKE 3: Catching too broadly
try:
    user = find_user(name)
    send_email(user.email)      # if this crashes, you'll never know
except Exception:
    print("User not found")     # wrong! the email send could've failed

# ✅ FIX: Catch only what you expect
try:
    user = find_user(name)
except KeyError:
    print("User not found")
```

---

## 🔥 Real-World Example: ML Model Training Pipeline

This is how exception handling looks in a REAL ML project:

```python
import os

# --------- Custom Exceptions ---------
class DataLoadError(Exception):
    """Data file couldn't be loaded."""
    pass

class ModelTrainingError(Exception):
    """Model training failed."""
    pass

# --------- Functions ---------
def load_data(filepath):
    """Load CSV data for training."""
    if not os.path.exists(filepath):
        raise DataLoadError(f"Dataset not found: {filepath}")

    data = []
    with open(filepath, "r") as f:
        header = f.readline().strip().split(",")
        for line_num, line in enumerate(f, start=2):
            values = line.strip().split(",")
            try:
                row = [float(v) for v in values]
            except ValueError as e:
                print(f"⚠️ Skipping row {line_num}: can't convert '{e}'")
                continue  # skip bad rows, don't crash
            data.append(row)

    if len(data) == 0:
        raise DataLoadError("Dataset is empty after cleaning!")

    print(f"✅ Loaded {len(data)} rows, {len(header)} columns")
    return header, data


def split_features_target(data):
    """Last column = target (y), rest = features (X)."""
    X = [row[:-1] for row in data]
    y = [row[-1] for row in data]
    return X, y


def train_model(X, y):
    """Simple average-based 'model' (for demonstration)."""
    if len(X) == 0:
        raise ModelTrainingError("No training data!")
    if len(X) != len(y):
        raise ModelTrainingError(
            f"X has {len(X)} samples but y has {len(y)}"
        )

    # "Train" — just compute mean of each feature
    n_features = len(X[0])
    weights = []
    for col in range(n_features):
        col_values = [row[col] for row in X]
        weights.append(sum(col_values) / len(col_values))

    print(f"✅ Model trained with {len(weights)} weights")
    return weights


def predict(weights, sample):
    """Simple dot-product prediction."""
    if len(weights) != len(sample):
        raise ValueError(
            f"Model expects {len(weights)} features, got {len(sample)}"
        )
    return sum(w * x for w, x in zip(weights, sample))


# --------- The Pipeline ---------
def run_pipeline(filepath):
    """Full ML pipeline with proper error handling."""
    model = None

    try:
        # Step 1: Load data
        print(f"📂 Loading data from {filepath}...")
        header, data = load_data(filepath)

        # Step 2: Split features and target
        X, y = split_features_target(data)

        # Step 3: Train model
        print("🧠 Training model...")
        model = train_model(X, y)

    except DataLoadError as e:
        print(f"❌ DATA ERROR: {e}")
        print("   → Check your file path and data format")
        return None

    except ModelTrainingError as e:
        print(f"❌ TRAINING ERROR: {e}")
        print("   → Check your dataset isn't empty or corrupted")
        return None

    except Exception as e:
        print(f"❌ UNEXPECTED ERROR: {type(e).__name__}: {e}")
        raise  # re-raise unexpected errors (don't hide bugs)

    else:
        # Only runs if training succeeded
        print("✅ Pipeline completed successfully!")

        # Test prediction
        try:
            sample = X[0]
            pred = predict(model, sample)
            print(f"   Test prediction on first sample: {pred:.2f}")
        except ValueError as e:
            print(f"   ⚠️ Prediction failed: {e}")

    finally:
        # Always runs — cleanup
        print("🧹 Cleanup: releasing resources")

    return model


# --------- Run It ---------
if __name__ == "__main__":
    # This will show DataLoadError since file doesn't exist
    print("--- Run 1: Bad file path ---")
    run_pipeline("nonexistent_data.csv")

    print()

    # Create a small test CSV to show success path
    test_file = "test_data.csv"
    with open(test_file, "w") as f:
        f.write("feature1,feature2,target\n")
        f.write("1.0,2.0,3.0\n")
        f.write("4.0,5.0,6.0\n")
        f.write("7.0,8.0,9.0\n")
        f.write("bad,data,here\n")   # ← intentionally bad row

    print("--- Run 2: Real data (with one bad row) ---")
    model = run_pipeline(test_file)

    # Cleanup test file
    os.remove(test_file)
```

**Output:**
```
--- Run 1: Bad file path ---
📂 Loading data from nonexistent_data.csv...
❌ DATA ERROR: Dataset not found: nonexistent_data.csv
   → Check your file path and data format
🧹 Cleanup: releasing resources

--- Run 2: Real data (with one bad row) ---
📂 Loading data from test_data.csv...
⚠️ Skipping row 5: can't convert "could not convert string to float: 'bad'"
✅ Loaded 3 rows, 3 columns
🧠 Training model...
✅ Model trained with 2 weights
✅ Pipeline completed successfully!
   Test prediction on first sample: 14.00
🧹 Cleanup: releasing resources
```

### What This Example Teaches:
1. **Custom exceptions** (`DataLoadError`, `ModelTrainingError`) — name your errors clearly
2. **Skipping bad data** with `continue` instead of crashing
3. **Specific catches** — data errors vs training errors vs unexpected errors
4. **`else`** — test prediction only if training succeeded
5. **`finally`** — cleanup always runs
6. **`raise`** — re-raise unexpected errors so you don't hide bugs

---

## Interview Questions — "What Does This Print?"

### Q1: Basic order
```python
try:
    print("A")
    x = 1 / 1
    print("B")
except ZeroDivisionError:
    print("C")
else:
    print("D")
finally:
    print("E")
```
> **Answer:** `A B D E` — No crash, so `except` is skipped. `else` runs because no error. `finally` always runs.

### Q2: Error mid-try
```python
try:
    print("A")
    x = 1 / 0         # 💥 crash here
    print("B")         # skipped!
except ZeroDivisionError:
    print("C")
else:
    print("D")
finally:
    print("E")
```
> **Answer:** `A C E` — Crash at line 3, jumps to `except`, skips `else`, `finally` always runs.

### Q3: Wrong except type
```python
try:
    int("abc")
except IndexError:
    print("caught!")
```
> **Answer:** `ValueError` crashes the program. We were catching `IndexError` but a `ValueError` happened — wrong net!

### Q4: finally with return
```python
def f():
    try:
        return 1
    except:
        return 2
    finally:
        return 3

print(f())
```
> **Answer:** `3` — `finally` OVERRIDES the return from `try`. This is a trap. Never put `return` in `finally`.

### Q5: Nested try-except
```python
try:
    try:
        1 / 0
    except ValueError:      # wrong type!
        print("inner caught")
except ZeroDivisionError:
    print("outer caught")
```
> **Answer:** `"outer caught"` — Inner `except` doesn't match `ZeroDivisionError`, so the error bubbles up to the outer `try`.

### Q6: raise inside except
```python
def f():
    try:
        return "try"
    except:
        return "except"
    finally:
        pass    # no return here

print(f())
```
> **Answer:** `"try"` — No error occurs, so `try`'s return is used. `finally` runs but doesn't override (no return in it).

### Q7: Verbal Question — "When would you use `else` vs just putting code in `try`?"
> **Answer:** Code in `else` only runs if `try` succeeded, AND any errors in `else` won't be caught by the `except` above it. If you put everything in `try`, a bug in your success code might get accidentally caught by your `except` — hiding a real bug.

---

## Summary — The Entire Topic in 6 Lines

1. **`try`** — wrap risky code that might crash.
2. **`except ErrorType`** — catch a specific crash and handle it.
3. **`else`** — runs only if `try` didn't crash.
4. **`finally`** — ALWAYS runs (cleanup).
5. **`raise`** — deliberately throw an error.
6. **Custom exceptions** — inherit from `Exception` to name your errors clearly.

---

*← [Previous: Packages](18_Packages.md) | [Next: File Handling →](20_File_Handling.md)*
